---
layout: post
title: GSOC It's all over
tag:
    - GSOC
    - Python
---

Finally, it's over. It was an amazing experience with amazing people, I just couldn't list everything I've learned. The beginning and end were quite complicated, but the remaining time was well spent, believe me.

As usual, PYTI is not fully working right now (but I do not think it happens often during GSOC :p) but we are very close. Lots of details must still be added, but the infrastructure is done and the workflow is clear.

We hope be able to put an online hosted version of PYTI before end of the year (the master part of infrastructure is [already online](http://master.pyti.org).)

Concerning the project itself, it has been split into 3 differents parts:

> -   [PYTI-Worker](https://bitbucket.org/swamiyeswanth/pyti-worker): The 'infrastructure' part of PYTI, it consists on get PYPI notifications and dispatch test jobs to slaves.
> -   PYTHIA: It handles the virtual machines management.
> -   [Goatlib](https://bitbucket.org/marmoute/goatlib): The task-manager library we use for launching tests and quality tools.
> -   [Goatlog](https://bitbucket.org/marmoute/goatlog): It's a rich logging mechanism intended to be primarily used for Continuous Integration.

All these projects are tested and documented (surely not enough :p), you are welcome if you want to participate, we're looking particularly people to make a great web-interface for PYTI as we don't focuss at all on User Interface during GSOC.

Finally, yeswanth (the other student who works on PYTI with me) has made a [great presentation](http://urtalk.kpoint.in/kapsule/gcc-b54c0556-bb9c-465d-8dd9-6f7c80ef5521) of the project at PYCON India, congratulations!

Thanks again to all people involved in PYTI:

> -   [Alexis](https://twitter.com/#!/ametaireau): My GSOC mentor.
> -   [Yeswanth](https://twitter.com/#!/s1thsv): My colleague.
> -   [Marmoute](https://twitter.com/#!/marmoute): Yeswant's mentor.

Stay tuned for more information about PYTI future.
