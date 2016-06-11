---
layout: post
title: GSOC sprint in Paris - The end
tag:
    - GSOC
    - Python
---

It's over, first gsoc sprint has end today. Many works has been done:

-   A functional task-manager - Yes it works, every idea has not been implemented, but a simple recipe (extract, build, install and launch tests) should works.
-   Some simples tasks (build, install) - There are written, but need to be tested in the recipe.
-   A VM manager able to launch something - Yes.
-   Dependencies computing - No, but we decided to use PEP-345 (http://www.python.org/dev/peps/pep-0345/\_) metadata file: "setup.cfg"; it requires that we can only test distributions which includes both setup.cfg, for retrieving dependencies and setup.py in order to install the distribution.

All works has not been done, but a lot of work has been done and there were a lot of discussions, and now a lot of ideas are both clear and implemented. It's a good thing in my opinion to have something clear for everyone before the midterm evaluation. Some ideas/concepts are still in discussions (and subject to trolls).

We've also clarified what must be ready for midterm-evaluation:

-   A functional "simple" recipe (extract, build, install and launch tests).

    > -   Implies dependencies computing.

-   Some logging (at least output to stdout).
-   Documentation about what is clear:

    > -   Creating tasks.
    > -   Creating recipes.
    > -   Configure one recipe in order to execute it.
    > -   Explications about task execution result (status, debug infos).
    > -   How Item are managed.

For midterm-evaluation, we must be able to install a "simple" distribution, launch tests and make the results available for the data storage or the master.

I will try to make a blog post about task-manager details and how to use it (will be surely quite similar with the documentation).

Current documentation is available here: http://pyti.readthedocs.org/\_ which explains some ideas and terms.
