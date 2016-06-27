---
layout: post
title: Dependencies are the root of all complexity
bigimg: /img/wire.jpeg
---

# External dependencies

When I say `dependency`, what do you think about?

If you thought about your `package.json` file or `setup.py` file (or whatever file you put your project dependency on), bravo but let's try to go deeper.

If your python project ask `requests` in its requirements.txt file and you use it like this in your project:

{% highlight python %}
import requests
requests.get('https://google.com/...')
{% endhighlight %}

What do you really depends on?

Let's take it line by line:

{% highlight python %}
import requests
{% endhighlight %}

Here you're asking to import (require in another language) a package named requests. It could be local (love these nasty errors), provided by the `requests` pypi package or any other package.

{% highlight python %}
requests.get('https://google.com/...')
{% endhighlight %}

Now your code call a get function with a single string parameter on this package.

If we try to sum up, your code is actually depending on a get function on the `requests` namespace accepting at least one string parameter. You essentially depends on this API and anything that provides a compatible API should be compatible with your code. 

The `request==2.9.1` you put in your `requirements.txt` just tell your package manager where it could find a code archive matching this requirement.

Why pinning a version is important then? Because we don't want our code to break if the `requests` package evolve, right?

But we still want some security update, bug fixes and performances enhancements, right?

Luckily, it's the purpose of [Semantic Versionning](http://semver.org/). What is says is that of you use a package, for example `requests` in your code in version 2.9.1 for example, your code should works without modifications in the next minor version, 2.9.2.

If `requests` release a new minor version, 2.10.0 for example, your code should still works but the release could contains some new features that may be useful to you.

Finally, if `requests` release a new major version, 3.0.0 for example, your code is likely to require some modification to use the new functionality and match to the new usage.

Let's try to define what is a definition:

> If A depends on B and B change, A should change or break

External dependencies are the most known and obvious kind of dependencies you have in your projects.

Each time, you add a dependency, more than putting a new line in your `package.json`, `setup.py` or `Gemfile`, you also link a part of your code to the new dependency API.

# Internal dependencies

# MVC

# Hexagonal Architecture

# Micro-Services

# Abstraction

# Dependencies in tests
