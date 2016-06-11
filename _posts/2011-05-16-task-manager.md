---
layout: post
title: Python task manager comparative
tag:
    - GSOC
    - Python
---

In order to create a task manager for the PYTI project, I have to compare which libraries exists, compare them and analysis how they can match our needs.

Our needs
---------

From my [Proposal](http://www.google-melange.com/gsoc/proposal/review/google/gsoc2011/lothiraldan/1):  
Running the tests on package content may be split into different independent tasks. These tasks can be written independently, but they must all be executed anyway. They cannot be performed in any order, as one specific task may depends on an another one and if a task fail, the system should not performed other tasks that depend on that which has failed.

This is the role of the execution manager, it will manage all the different tasks and execute them in the right order, ensuring that tasks success. Execution manager must manage these scenario:

Task 'A' depends on 'B' and 'B' one depends on 'C'. Execution order must be: 'C', 'B' and finally 'A'. If task 'B' fail, execution manager must mark 'A' as skipped due to error in one of it's dependencies. Execution manager must also provide a way for tasks to exchange data, and so we can imagine that tasks will depends on data instead of traditional dependencies. Moreover, as tasks should generate raw data, it makes more sense to manage raw data dependencies than tasks dependencies. For example:

Task 'A' need 'D' data. Task 'B' produce 'D' data during his execution. Execution order must be: 'B' and 'A'.

Scoring
-------

First we need to check that solution can match our needs (dependencies, clean shutdown on task fail, communications).

Then every solution should ideally be fully tested and documented.

Next, solution may be available as separated package or not (see <http://wiki.python.org/moin/PyPITestingInfrastructure> \#Task-manager for more details about that).

Finally, is the project famous and used.

If the solution is not available as separated package and program whose include it is very dependent on another project, we should add some penalties as it will require more work.

Pony-build
----------

Pony-build is the continuous integration tools used last year for PYTI. Characteristics:

-   No dependencies between tasks, task-manager take a list of task to be executed task by task.
-   Task manager stop on fail of one task, cannot mark a task as skipped and don't mark dependencies as skipped.
-   No communication between tasks.
-   No tests.
-   No documentation.
-   Part of pony-build, but seems easy to extract.

Score:

-   No dependencies (0/4).
-   Stop on fail (1/3).
-   No communication (0/1)
-   No tests (0/2).
-   No documentation (0/2).
-   Part of pony-build, easy to extract (2/3).
-   Slighty used (1/4).

Total: 4

PythonTasks
-----------

PythonTasks is one of my personal project, it's still in beta and no doc is written for the moment. Characteristics:

-   Dependencies is either direct dependencies (task A depends on B) or output dependencies (task A need D input, B produces D, B must be executed before A). For the output dependencies mode, some code must be written as it will not be part (for the moment) of PythonTasks.
-   Tasks can communicate between them, all of them receive the same Communication object which registered all the output of tasks.
-   When a task fail, all others tasks, which depends on it, are marked as skipped before it failed (if B depends on A; A failed, B is marked as skipped before A has failed) and so recursively. Works the same way when a task has skipped.
-   Unittest and code coverage.
-   No documentation.

Score:

-   Dependencies (direct or output) but requires works for output dependencies mode (3/4).
-   Stop on fail, mark dependencies as skipped, but no fine configuration of tasks (like buildbot) (2/3).
-   Communication between tasks (1/1).
-   Unittests (2/2).
-   No documentation (0/2).
-   Separated package (3/3).
-   Not used (0/3).

Total: 11

Narval
------

Narval is a task-manager library written by logilab and is used by apycot, a continuous integration tool written by logilab. Characteristics: \* Seems complex. \* Some part of the API seems not adapted, a POO approach seems better than the existing one. \* Tasks are associated in a recipe module.

Score:

-   Dependencies: direct and output mode supported (4/4).
-   Fine configuration of what doing if a step failed or emits warnings (3/3).
-   Communications (1/1).
-   No tests (0/2).
-   No specific documentation (0/2).
-   Part of narval, hard to extract (2/3).
-   Slighty used (1/3).

Total: 11

Buildbot
--------

Buildbot is the reference of continuous integration tool in python world. Characteristics:

-   In buildbot vocabulary, steps are what we called tasks.
-   Fine configuration of what doing if a step failed or emits warnings.
-   Steps seems registered into two different places (master and slave).
-   All buildbot system seems really dependent on twisted.
-   Part of buildbot, intermediate quantity of work to extract it.

Score:

-   No dependencies between tasks, scheduler take a list of builder identifier (0/4).
-   Fine configuration of what doing on fail, warning (3/3).
-   No communication between tasks (0/1).
-   Unittests (2/2).
-   Good documentation (2/2).
-   Part of buildbot (1/3).
-   Very used (3/3).
-   Dependencies on twisted (-1).

Total: 10

Conclusion
----------

One thing I've not mentioned in this comparison, most of integrated solutions have already some tasks written that we could use for PYTI, but most of them launch system commands and parse output. As I consider that launch os commands is road to hell, I've not add it in comparison.

With the actual point system, here is an ordered list of which solution we could use:

-   PythonTasks (11/18).
-   Narval (11/18).
-   Buildbot (10/18).
-   Pony-build (4/18).

The final choice will be between PythonTasks, buildbot and narval. As my mentor advised me, we will probably take all the good idea of theses solutions and we will combine them into a final library. As PythonTasks is available as a separated package and as I am the main developer, I recommend to use PythonTasks as a base for our task-manager solution.

If you're take part in one of the solution's project and want to correct me about solution, do not hesitate, comments are open.
