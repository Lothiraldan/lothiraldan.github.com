---
layout: post
title: Use line_profiler without magic
tag:
    - Python
---

When debugging performance issues, I usually rely on the good old [line_profiler](https://pypi.org/project/line_profiler/). It's very useful to easily identify which lines of a specific function are slow and needs to be investigated and/or fixed.

Using it is straightforward, you basically add the `@profile` decorator on the function you want to profile:

```python
@profile
def slow_function(a, b, c):
    ...
```

Then you launch your script with:

```bash
$ kernprof -l script_to_profile.py
```

And generate the report with:

```bash
$ python -m line_profiler script_to_profile.py.lprof
Timer unit: 1e-06 s

File: pystone.py
Function: Proc2 at line 149
Total time: 0.606656 s

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
   149                                           @profile
   150                                           def Proc2(IntParIO):
   151     50000        82003      1.6     13.5      IntLoc = IntParIO + 10
   152     50000        63162      1.3     10.4      while 1:
   153     50000        69065      1.4     11.4          if Char1Glob == 'A':
   154     50000        66354      1.3     10.9              IntLoc = IntLoc - 1
   155     50000        67263      1.3     11.1              IntParIO = IntLoc - IntGlob
   156     50000        65494      1.3     10.8              EnumLoc = Ident1
   157     50000        68001      1.4     11.2          if EnumLoc == Ident1:
   158     50000        63739      1.3     10.5              break
   159     50000        61575      1.2     10.1      return IntParIO
```

Which is great... until it's not anymore.

# Too much magic

Line profiler is great for profiling small scripts, you add the decorator, you run it with `kernprof` and voila. The problem is that you always need to launch it with `kernprof` as it's injecting the `profile` decorator. If you try to launch your script without `kernprof` you will have a nice `NameError: name 'profile' is not defined`.

Moreover, in some cases, you just cannot use `kernprof`, for example when trying to profile a web server or when the Python interpreter is launched by a bash/script or another process you cannot modify.

Luckily for us, it's not that hard to use `line_profiler` without `kernprof`.

# The magic trick

The kernprof magic trick is not that complicated as you'll see.

First it [instantiate the right object](https://github.com/rkern/line_profiler/blob/df8dfc88049effeac5fb77830bc9456052ea5c33/kernprof.py#L198-L199):

```python
import line_profiler
prof = line_profiler.LineProfiler()
```

Then it [inject it in the builtins](https://github.com/rkern/line_profiler/blob/df8dfc88049effeac5fb77830bc9456052ea5c33/kernprof.py#L208):

```python
builtins.__dict__['profile'] = prof
```

[Execute the Python script](execfile(script_file, ns, ns)):

```python
execfile(script_file, ns, ns)
```

And finally [save the stats](https://github.com/rkern/line_profiler/blob/df8dfc88049effeac5fb77830bc9456052ea5c33/kernprof.py#L228) and [print them if needed](https://github.com/rkern/line_profiler/blob/df8dfc88049effeac5fb77830bc9456052ea5c33/kernprof.py#L231) with:

```python
prof.print_stats()
```

No Rocket science involved here. With all these information we can now use it manually.

# Manual use

Let's use a simple Python script for showing you how to use it manually. The following script answer this exercise:

> Given a list of integers and a target integer, the function should answer True if the target could be created by adding exactly two integers from the list, False if not.

Here is a naive solution:


```python
def is_addable(l, t):
    for i, n in enumerate(l):
        for m in l[i:]:
            if n + m == t:
                return True

    return False


assert is_addable(range(20), 25) == True # 25 = 6 + 19
assert is_addable(range(20), 40) == False
```

The goal is to optimize this simple function. Let's create a line_profiler and decorate our function:

```python
import line_profiler
profile = line_profiler.LineProfiler()

@profile
def is_addable(l, t):
    for i, n in enumerate(l):
        for m in l[i:]:
            if n + m == t:
                return True

    return False

assert is_addable(range(20), 25) == True
assert is_addable(range(20), 40) == False
```

Launch the script, it should run a bit slower, it's normal as the script is now profiled. But you don't have the report either.

That's normal, we didn't call the `print_stats` function. But we need to call it at the end of the script. We could manually call it at the end of the script, but in some cases, it would be tedious to add it manually.

Instead, we can use the [`atexit` module](https://docs.python.org/3.7/library/atexit.html) to call it for us at the end of the current Python process:

```python
import line_profiler
import atexit
profile = line_profiler.LineProfiler()
atexit.register(profile.print_stats)

@profile
def is_addable(l, t):
    for i, n in enumerate(l):
        for m in l[i:]:
            if n + m == t:
                return True

    return False

assert is_addable(range(20), 25) == True
assert is_addable(range(20), 40) == False
```

Now let's run the script once again:

```bash
$ python script.py
Timer unit: 1e-06 s

Total time: 0.000171 s
File: script.py
Function: is_addable at line 6

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     6                                           @profile
     7                                           def is_addable(l, t):
     8        28         12.0      0.4      7.0      for i, n in enumerate(l):
     9       355         70.0      0.2     40.9          for m in l[i:]:
    10       329         87.0      0.3     50.9              if n + m == t:
    11         1          1.0      1.0      0.6                  return True
    12                                           
    13         1          1.0      1.0      0.6      return False
```

Hey much better! Optimizing the function is left as an exercise for the reader.

One last tip, if you want to profile several functions, only instantiate once the `LineProfiler` and import it in the other files. If you don't do that, you might have some issues and weird reporting.