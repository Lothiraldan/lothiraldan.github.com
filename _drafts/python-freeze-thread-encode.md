---
layout: post
title: Freeze your Python with str.encode and threads
bigimg: /img/frozen.jpg
tag:
    - Python
---

While working on a the [sqreen.io Python agent](https://www.sqreen.io/), I struggled against a very nasty but fun bug that lead me to understand some internals of Python better.

# The nasty bug

I had a very nice piece of code that launched a thread, retrieved some JSON and stored it for later use. The code was something like that (simplified for the example):

```python
from threading import Thread, Event

class MyThread(Thread):

    def __init__(self):
        self.event = Event()

    def run(self):
        data = self.retrieve_json()
        self.data = data
        self.event.set()
        print("Data retrieval done!")
        while True:
            pass

thread = MyThread()
thread.start()
timeout = thread.event.wait(5)

if timeout is not None:
    print("JSON retrieval was too slow")
```

It was working, the error message was displayed when network latency was too high:

```bash
$> python2 app.py
Data retrieval done!
```

Then I tried to convert the dict keys and values from unicode to latin-1 strings:

```python
class MyThread(Thread):

    def run(self):
        data = self.retrieve_json()
        # Convert unicode strings into latin1 strings because of WSGI
        self.data = {key.encode('latin1'): value.encode('latin1') for (key, value) in data.keys()}
        self.event.set()
        print("Data retrieval done!")
        while True:
            pass
```

I'm pretty sure you've once made this type of transformation when receiving JSON, haven't you?

After adding this code, the error message was always displayed, no matter what timeout value I was setting, 5 seconds or 1 hour. To add to the strangeness, the `Data retrieval done!` message was always printed right after the error message:

```bash
$> python2 app.py
JSON retrieval was too slow
Data retrieval done!
```

I tried with Python 3 and saw the correct behavior:

```bash
$> python3 app.py
Data retrieval done!
```

There was something very strange going on, so I decided to go down the rabbit hole.

# The isolation

I tried to isolate the problem and came to this minimal reproducing script, let's name this file `str_encode_thread.py`:

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

If you launch this script with Python 3, you'll see:

```bash
$> python3 str_encode_thread.py
1467046041.643346 Before encode
1467046041.643446 After encode
1467046041.643483 Event set
```

And with Python 2, you'll see:

```bash
$> python2 str_encode_thread.py
(1467046099.495261, 'Before encode')
(1467046099.495937, 'After encode')
(1467046099.495961, 'Event set')
```

So far, so good. The fun comes when we try to import this script. If you import the script with Python 3, everything will still be normal:

```bash
$> time python3 -c "import str_encode_thread"
1467046239.921475 Before encode
1467046239.921578 After encode
1467046239.921602 Event set
python3 -c "import str_encode_thread"  0,05s user 0,01s system 84% cpu 0,081 total
```

While with python 2, our nasty friend shows up:

```bash
$> time python2 -c "import str_encode_thread"
(1467046276.826755, 'Before encode')
(1467046286.830485, 'After encode') # <- 10 seconds (╯°□°）╯︵ ┻━┻
(1467046286.830519, 'Event set')
python2 -c "import str_encode_thread"  0,03s user 0,03s system 0% cpu 10,109 total
```

If you look closely the `After encode` message, you can see that it was printed 10 se[conds after the `Before encode` message, indicating that the thread was somehow frozen.

# Down the rabbit hole

Why whould a simple line like `value.encode('latin1', 'encode')` block the thread?

Fortunately, there is a [very nice package named faulthandler](https://pypi.python.org/pypi/faulthandler/2.4) which is very helpful in these situations. It can display a traceback on unix signals, on a user signal or on python fault. It's included in the standard library since python 3.3 and is installable via pip for other Python versions.

In our case, we will use [`dump_traceback_later`](http://faulthandler.readthedocs.io/#dumping-the-tracebacks-after-a-timeout) to have an idea where the tread is blocking.

The new script with faulthandler is not much more different (don't forget to `pip install faulthandler`):

```python
import time
import threading
import faulthandler


faulthandler.dump_traceback_later(5)


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

Let's run our new script again:

```bash
time python2 -c "import str_encode_thread"
(1468083195.13011, 'Before encode')
Timeout (0:00:05)!
Thread 0x0000700000401000 (most recent call first):
  File "/usr/local/Cellar/python/2.7.12rc1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/encodings/__init__.py", line 100 in search_function
  File "str_encode_thread.py", line 11 in to_latin_1
  File "/usr/local/Cellar/python/2.7.12rc1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.py", line 754 in run
  File "/usr/local/Cellar/python/2.7.12rc1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.py", line 801 in __bootstrap_inner
  File "/usr/local/Cellar/python/2.7.12rc1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.py", line 774 in __bootstrap

Current thread 0x00007fff71258000 (most recent call first):
  File "/usr/local/Cellar/python/2.7.12rc1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.py", line 359 in wait
  File "/usr/local/Cellar/python/2.7.12rc1/Frameworks/Python.framework/Versions/2.7/lib/python2.7/threading.py", line 614 in wait
  File "str_encode_thread.py", line 20 in <module>
  File "<string>", line 1 in <module>
(1468083205.133041, 'After encode')
(1468083205.133106, 'Event set')
python2 -c "import str_encode_thread"  0,03s user 0,02s system 0% cpu 10,043 total
```

What can we see with the traceback?

The main thread `0x00007fff71258000`, is blocked on line 20 of our module which is `event.wait(10)`, it's perfeclty normal.

The other thread `0x0000700000401000`, is running the line 11 of our module `value.encode('latin-1')` and called the function `search_function` of the builtin module `encoding` which freeze our thread. What does this function does and why does it blocks?

The [source code](https://hg.python.org/cpython/file/v2.7.12rc1/Lib/encodings/__init__.py#l99) tells us that it tries to import a module which begins with `encodings.`. Let's try to find which one:

```python
In [1]: import sys

In [2]: before_modules = sys.modules.keys()

In [3]: '42'.encode('latin-1')
Out[3]: '42'

In [4]: after_modules = sys.modules.keys()

In [5]: print("Imported modules", set(after_modules) - set(before_modules))
('Imported modules', set(['encodings.latin_1']))
```

Gotcha! `str.encode(X)` tries to import the module `encodings.X`. So the fix should be easy, import the module in the main thread before using the function, and it should work, easy, right?

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

Let's try again with python2:

```bash
$> time python2 -c "import str_encode_thread"
(1467049098.995541, 'Before encode')
(1467049109.000293, 'After encode')
(1467049109.000324, 'Event set')
python2 -c "import str_encode_thread"  0,03s user 0,02s system 0% cpu 10,054 total
```

We still have the problem! What's wrong?

# We have to go deeper

We know that `str.encode` tries to import a module and that's likely what is freezing our thread.

At this point, I went to my favorite IRC channel (#python-fr) and asked for help, and we finally found the explanation!

What does `import module` do? If we trust `[the Python documentation](https://docs.python.org/2/library/imp.html#examples), it basically does this:

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

Nothing very fancy here: return the module from `sys.modules` if it has already been imported or try to locate the source file for the module, load it and return it.

Nothing should block here either, but if we continue to read the `imp` module documentation, we can find a [very interesting function named `lock_help`](https://docs.python.org/2/library/imp.html#imp.lock_held):

    On platforms with threads, a thread executing an import holds an internal lock until the import is complete. This lock blocks other threads from doing an import until the original import completes, which in turn prevents other threads from seeing incomplete module objects constructed by the original thread while in the process of completing its import (and the imports, if any, triggered by that).

We are reaching the explanation!

# Locks and thread, a hate-love relation

When googling `python import lock`, we quickly can find this section about [Importing in threaded code](https://docs.python.org/2/library/threading.html#importing-in-threaded-code) that says that it's a terrible idea, and you can encounter deadlocks.

Let's sum up what we found:

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
$> python2 -c "import str_encode_thread"
(1467050582.565935, 'Before encode', 'Thread-1', True)
(1467050592.567373, 'After encode')
(1467050592.567418, 'Event set')
```

When we imported our module `str_encode_thread`, the import lock was acquired by the MainThread. When our custom thread tried to execute the `value.encode('latin-1')`, the stdlib tried to import the `encoding.latin_1` module which blocks as the import lock is already held by the MainThread. We then block the MainThread by doing `event.wait(10)` and both threads are blocked for 10 seconds, too bad... When the wait timeouts, the `import str_encode_thread` ends, releasing the import lock which awakens our custom thread that can continue.

If we hadn't set a maximum timeout for the event, we would have deadlocked our Python interpreter forever!

With threads, there is no magic solution. You may wonder what would happen if you manually imported a module without using the lock? It may "solve" the deadlocks but may also raise a new type of fun bugs.

For example, when you do `time.strptime`, it tries to import the `_strptime` module in a non-safe manner, which may trigger some hard-to-debug issues, like [this boto issue](https://github.com/boto/boto/issues/1898) and [the corresponding Python bug page](http://bugs.python.org/issue7980). This problem can be solved by doing `import _strptime` earlier in the main thread because the thread will not block due to the lock and not see an incomplete module.

# Python 3 is the future

But why did it works in Python 3? In addition to breaking all your imports name and forcing you to put `.encode` and `.decode` everywhere in your code (which is a good thing!), the import lock was refactored in Python 3 and [the global import lock is now per module](https://mail.python.org/pipermail/python-dev/2013-August/127902.html). You're now less likely to trigger the problem.

# We should all use Python 3

It was a very nasty bug, but it was also very fun to dig into the less known mechanisms of Python.

**TLDR**: In Python 2, there is a single reentrant lock that will block your threads if they try to do an import while the lock is held by another thread. In Python 3, the import lock is now per module.
