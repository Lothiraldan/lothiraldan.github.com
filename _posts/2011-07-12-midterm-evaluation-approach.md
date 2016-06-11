---
layout: post
title: Midterm evaluation approach
tag:
    - GSOC
    - Python
---

With the midterm evaluation approach, I found useful to summarize the first part of this GSOC.

What had I learnt?
------------------

This GSOC has been very instructive and I realized something that may seem obvious to some people; Open Source is not only code. It's also communication and mainly collaboration. I already wrote some open source programs and participated at a sprint for distutils2, but I never realized that communication represented such an import part in open-source. This is also true in Software Engineering I guess.

Communication and collaboration are something which are not taught at school, or at least poorly taught. Who has never write "dirty code" for a school project, knowing that the teacher will not even read the code and nobody will try to launch it after the last of the defense. If your teacher, who don't care about coding conventions and don't really master what he teachs, say you that you should add more docs for your project and if your partner, who will really use your program or will collaborate to it, say you the same thing, it will not have the same impact I am sure.

I also learnt a lot about python programming tips and tricks and I could summarize in four words: "Use the standard library!". If you must implements something that is quite trivial, you have great chances that "There's a module for that in stdlib". If you really master the standard library, you could simplify all your codes, it's really impressive.

What had been implemented?
--------------------------

A lot of work has been done. We choose to start from scratch with new ideas about how to combine all parts of the project. I worked on the execution part and I'm proud to say you that we've something working for midterm. In other words, we can extract a distribution archive, install this distribution then launch the tests. The environment part seems working too (http://advencode.wordpress.com/2011/07/12/my-experience-so-far/\_), we just have to see how we will connect both parts.

If I take my last week todo list:

Done:

-   A functional "simple" recipe (extract, build, install and launch tests).

    > -   Implies dependencies computing.

-   Some logging (at least output to stdout). Still need a little quantity of work.

Remains to be done:

-   Documentation about what is clear:

    > -   Creating tasks.
    > -   Creating recipes.
    > -   Configure one recipe in order to execute it.
    > -   Explications about task execution result (status, debug infos).
    > -   How Item are managed.

Yes, I must write the doc, I am not particularly fan of this job, but it's necessary.

We are close to be ready for midterm, but I think I would not be able to write one more post about the execution part this week, so you can follow the future evolutions of the documentation here: http://pyti.readthedocs.org/\_.

Wish me luck for the evaluation!
