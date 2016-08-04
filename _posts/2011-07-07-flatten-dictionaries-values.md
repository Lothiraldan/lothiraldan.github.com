---
layout: post
title: "Python trick : how to flatten dictionaries values composed of iterables ?"
tag:
    - GSOC
    - Python
---

During yesterday GSOC sprint, I found a python trick, and as it's a python trick, it's a one line trick.

What we want to do?
-------------------

When you have a dictionary which looks like:

```python
{"key1" : ["value1", "value2", "value3"], "key2" : ["value4", "value5", "value6"]}
```

Some times, you want to have a list of all values contained in the dictionary, with the previous example:

```python
["value1", "value2", "value3", "value4", "value5", "value6"]
```

How to do it?
-------------

First we need to get the values with the "values" method of dictionary:

```python
>>> x = {"key1" : ["value1", "value2", "value3"], "key2" : ["value4", "value5", "value6"]}
>>> x.values()
<<< [['value1', 'value2', 'value3'], ['value4', 'value5', 'value6']]
```

The problem is that you get a list of list, so you want to flatten it. You can do it "by hand", but python contains all the necessary. We will use ["itertools.chain" from the "itertools" module](http://docs.python.org/library/itertools.html\#itertools.chain). This function will flatten all iterable given in parameter, so in example:

```python
>>> chain('ABC', 'DEF')
<<< A B C D E F
```

If you try to use chain directly with your values, result may be surprising:

```python
>>> list(chain(x.values()))
<<< [['value1', 'value2', 'value3'], ['value4', 'value5', 'value6']]
```

It's normal as you give only one argument to "chain", so you need to give each element of "x.values()" and it's here that it's the real trick. We will use the "\*args" magic ([explained here](http://www.saltycrane.com/blog/2008/01/how-to-use-args-and-kwargs-in-python/), part "Using \*args and \*\*kwargs when calling a function").

Final function
--------------

```python
>>> from itertools import chain
>>> def flatten_dict_values(dictionary):
...     return chain(*dictionary.values())
>>> flatten_dict_values(x)
<<< ['value4', 'value5', 'value6', 'value1', 'value2', 'value3']
```

In addition, this function will works with all types of iterables, in example:

```python
>>> y = {"key1" : ["list"], "key2" : ("tuple",), "key3" : set(("set",)), "key4" : frozenset(("frozenset",))}
>>> flatten_dict_values(y)
<<< ['set', 'tuple', 'list', 'frozenset']
```

Update
------

As mentioned in the comments, we can realize this trick in different manner.

First, we can use the "chain.from\_iterable" method which take only one iterable, so our function become:

```python
>>> from itertools import chain
>>> def flatten_dict_values(dictionary):
...     return chain.from_iterable(dictionary.values())
>>> flatten_dict_values(x)
<<< ['value4', 'value5', 'value6', 'value1', 'value2', 'value3']
```

This method solves our problem, but it's only available since python2.6.

The second method is to use both "reduce" and "operator.add". Reduce apply function of two arguments cumulatively to the items of iterable. It works great with lists and tuples, but not for set and frozenset as they do not support the addition.

The function is then:

```python
>>> from operator import add
>>> def flatten_dict_values(dictionary):
...     return reduce(operator.add, dictionary.itervalues())
>>> flatten_dict_values(x)
<<< ['value4', 'value5', 'value6', 'value1', 'value2', 'value3']
```

Sum-up
------

We have now three functions, let's compare them:

| Function             | Available in python2.5 | Works with any type of iterable | Flatten the values |
|----------------------|------------------------|---------------------------------|--------------------|
| chain(\*)            | Yes                    | Yes                             | Yes                |
| chain.from\_iterable | No                     | Yes                             | Yes                |
| reduce               | Yes                    | No                              | Yes                |

And now let's benchmark them with this script:

```python
from itertools import chain
from operator import add
import timeit

def flatten1(d):
    return chain(*d.itervalues())

def flatten2(d):
    return chain.from_iterable(d.itervalues())

def flatten3(d):
    return reduce(add, d.itervalues())

print "Benchmark with lists:"
print "-"*21

x = {"key1" : ["value1", "value2", "value3"], "key2" : ["value4", "value5", "value6"]}

print "Flatten1"
s = "list(flatten1(x))"
t = timeit.Timer(s, "from __main__ import (flatten1, x)")
print "%.5f usec/pass" % (min(t.repeat(3, 1000000)))

print "Flatten2"
s = "list(flatten2(x))"
t = timeit.Timer(s, "from __main__ import (flatten2, x)")
print "%.5f usec/pass" % (min(t.repeat(3, 1000000)))

print "Flatten3"
s = "list(flatten3(x))"
t = timeit.Timer(s, "from __main__ import (flatten3, x)")
print "%.5f usec/pass" % (min(t.repeat(3, 1000000)))

print "\nBenchmark with any sort of iterable:"
print "-"*36

y = {"key1" : ["list"], "key2" : ("tuple",), "key3" : set(("set",)), "key4" : frozenset(("frozenset",))}

print "Flatten1"
s = "list(flatten1(y))"
t = timeit.Timer(s, "from __main__ import (flatten1, y)")
print "%.5f usec/pass" % (min(t.repeat(3, 1000000)))

print "Flatten2"
s = "list(flatten2(y))"
t = timeit.Timer(s, "from __main__ import (flatten2, y)")
print "%.5f usec/pass" % (min(t.repeat(3, 1000000)))

print "Flatten3"
s = "list(flatten3(y))"
t = timeit.Timer(s, "from __main__ import (flatten3, y)")
print "%.5f usec/pass" % (min(t.repeat(3, 1000000)))
```

Result are:

-   With lists:

    > -   Flatten1: 3.25976 usec/pass
    > -   Flatten2: 2.30001 usec/pass
    > -   Flatten3: 1.59202 usec/pass

-   With any sort of iterables:

    > -   Flatten1: 3.80276 usec/pass
    > -   Flatten2: 2.78879 usec/pass
    > -   Flatten3: Error

We can see that flatten3 is more efficient than flatten2 and flatten2 is more efficient than flatten1.
