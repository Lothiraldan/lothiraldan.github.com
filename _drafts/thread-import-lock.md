---
layout: post
title: Froze your Python with str.encode and threads
bigimg: /img/frozen.jpg
tag:
    - Python
---

While playing with threads, I encountered a weird behavior that gave me some hard time to debug and understand.

# The nasty bug

I've add a very nice piece of code that launch a thread, retrieve some json and store it for later use. The code was something like that:

```python
from threading import Thread, Event

class MyThread(Thread):

    def __init__(self):
        self.event = Event()

    def run(self):
        data = self.retrieve_json()
        self.data = data
        self.event.set()
        print("Run done!")

thread = MyThread()
thread.start()
timeout = thread.event.wait(5)

if timeout is not None:
    print("Retrieve json was fast enough :(")
```

These code worked fine, the error message was displayed only when the network latency was too high. It worked correctly until I tried to process the received data this way:

```python
class MyThread(Thread):

    def run(self):
        data = self.retrieve_json()
        # Convert unicode strings into latin1 string because WSGI
        self.data = {key.encode('latin1'): value.encode('latin1') for (key, value) in data.keys()}
        self.event.set()
        print("Run done!")
```

After adding this code, the error message was always displayed, no matter what timeout value I set, 5 seconds or 1 hour. Moreover, the `Run done!` message was always printed just after the error message. I tried with python 3 and saw the correct behavior. There was something nasty behind this, so I decided to dig into the rabbit hole.

# The isolation

I tried to isolate the problem and came to this minimal reproducing script:

```python
import time
import threading


def to_latin_1(event, value):
    print(time.time(), "Before encode")
    value.encode('latin-1')
    print(time.time(), "After encode")
    event.set()
    print(time.time(), "Event set")

event = threading.Event()
thread = threading.Thread(target=to_latin_1, args=(event, u'42'))
thread.start()

event.wait(10)
```

If you try to launch this script with python 3, you should see:

```bash
$> python3 str_encode_thread.py
1467046041.643346 Before encode
1467046041.643446 After encode
1467046041.643483 Event set
```

And with python 2, you should see:

```bash
$> python2 str_encode_thread.py
(1467046099.495261, 'Before encode')
(1467046099.495937, 'After encode')
(1467046099.495961, 'Event set')
```

So far, everything works fine. The fun comes when we try to import this script, for example if the script file is named `str_encode_thread`:

```python
import str_encode_thread
```

Now if you launch this other script with python 3, everything should still be normal:

```bash
$> time python3 test.py
1467046239.921475 Before encode
1467046239.921578 After encode
1467046239.921602 Event set
python3 test.py  0,05s user 0,01s system 84% cpu 0,081 total
```

But with python 2, here comes our nasty friend:

```bash
$> time python2 test.py
(1467046276.826755, 'Before encode')
(1467046286.830485, 'After encode')
(1467046286.830519, 'Event set')
python2 test.py  0,03s user 0,03s system 0% cpu 10,109 total
```

## Disclaimer

There is perfectly valid reasons to launch a thread during an import, not relatively *répandues* but still valid, like launching a background thread in an import hook to process some files or running some code in [`sitecustomize` module](https://docs.python.org/3/library/site.html?highlight=sitecustomize).

# Down in the rabbit hole

Why a simple line like `value.encode('latin1', 'encode')` would block the thread? The GIL has nothing to do here, but could have been. My hunch was that `str.encode` does more that it appears on my back. I already had some fun with some imports that may triggers the same problem, for example importing `_strptime` before trying to call `strptime` from a thread, see [this boto issue](https://github.com/boto/boto/issues/1898) and [the python bug page](http://bugs.python.org/issue7980).

Maybe `str.encode` was importing a module behind my back. Let's try to find out which one:

```python
In [1]: import sys

In [2]: before_modules = sys.modules.keys()

In [3]: '42'.encode('latin-1')
Out[3]: '42'

In [4]: after_modules = sys.modules.keys()

In [5]: print("Imported modules", set(after_modules) - set(before_modules))
('Imported modules', set(['encodings.latin_1']))
```

Gotcha! `str.encode(X)` try to import the module `encodings.X`, it's not very documented, I couldn't find a clear explanation anywhere.

So the fix should be easy, import the module before in the main thread, and it should works, easy, right?

```python
import time
import threading
import encodings.latin_1


def to_latin_1(event, value):
    print(time.time(), "Before encode")
    value.encode('latin-1')
    print(time.time(), "After encode")
    event.set()
    print(time.time(), "Event set")

event = threading.Event()
thread = threading.Thread(target=to_latin_1, args=(event, u'42'))
thread.start()

event.wait(10)
```

Let's try again in python2:

```bash
$> time python2 test.py
(1467049098.995541, 'Before encode')
(1467049109.000293, 'After encode')
(1467049109.000324, 'Event set')
python2 test.py  0,03s user 0,02s system 0% cpu 10,054 total
```

We still have the problem! What is wrong?

# We have to go deeper

We know that `str.encode` try to import a module and that's likely what freeze our thread.

When you do `import module` in your code, what does it do? If we trust `[the python documentation](https://docs.python.org/2/library/imp.html#examples), it does basically that:

```python
import imp
import sys

def __import__(name, globals=None, locals=None, fromlist=None):
    # Fast path: see if the module has already been imported.
    try:
        return sys.modules[name]
    except KeyError:
        pass

    # If any of the following calls raises an exception,
    # there's a problem we can't handle -- let the caller handle it.

    fp, pathname, description = imp.find_module(name)

    try:
        return imp.load_module(name, fp, pathname, description)
    finally:
        # Since we may exit via an exception, close fp explicitly.
        if fp:
            fp.close()
```

Nothing very fancy here, return the module if it have already been imported and put into `sys.modules` or try to locate the source file for the module, load it and return it.

Nothing should blocks here either, but if we continue to read the `imp` module documentation, we can find a [very interesting function named `lock_help`](https://docs.python.org/2/library/imp.html#imp.lock_held) that says:

    On platforms with threads, a thread executing an import holds an internal lock until the import is complete. This lock blocks other threads from doing an import until the original import completes, which in turn prevents other threads from seeing incomplete module objects constructed by the original thread while in the process of completing its import (and the imports, if any, triggered by that).

We are reaching the explanation!

# Locks and thread, a hate-love relation

When googling `python import lock`, we quickly can find this section about [Importing in threaded code](https://docs.python.org/2/library/threading.html#importing-in-threaded-code) that says that it's a very bad idea and you can encounter deadlocks.

Let's sum up what we found.

```python
import imp
import time
import threading


def to_latin_1(event, value):
    print(time.time(), "Before encode", threading.current_thread().name, imp.lock_held())
    value.encode('latin-1')
    print(time.time(), "After encode")
    event.set()
    print(time.time(), "Event set")

event = threading.Event()
thread = threading.Thread(target=to_latin_1, args=(event, u'42'))
thread.start()

event.wait(10)
```

```bash
$> python2 test.py
(1467050582.565935, 'Before encode', 'Thread-1', True)
(1467050592.567373, 'After encode')
(1467050592.567418, 'Event set')
```

When we imported our module `str_encode_thread`, the import lock was acquired by the MainThread. When our custom thread tries to execute the `value.encode('latin-1')`, the stdlib tries to import the module `encoding.latin_1` which block as the import lock is already held by the MainThread. We then block the MainThread by doing `event.wait(10)` and both threads are blocked for 10 seconds, too bad... When the wait timeout, the `import str_encode_thread` ends, release the import lock which awake our custom thread that can continue.

If we didn't set a maximum timeout for the event, we would have deadlock our python interpreter forever!

# Python 3 is the future

But why did it works in Python 3. In addition of breaking all your imports name and forcing you to put `.encode` and `.decode` everywhere in your code, the import lock was refactored in python 3 and [the global import lock is now per module](https://mail.python.org/pipermail/python-dev/2013-August/127902.html).

# We should all use python 3

It was a very nasty bug but it was also very fun to dig into the less known mechanisms of Python.

**TLDR**: In python 2, there is a single reentrant lock that will block your threads if they try to do an import while the lock is held by another thread. In python 3, the import lock is now per module.
