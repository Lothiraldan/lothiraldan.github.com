---
layout: post
title: GSOC 2011 preparation
tag:
    - GSOC
    - Python
---

A quick post to speak about my project for this summer. I would participate to the Google Summer Of Code. For those unfamiliar with GSOC, it's a project funded by google to improve open-source project. To summarize, google pays student to contribute to open-source project.

Let talk about the project that interest me : PYpi Testing Infrastructure (PYTI). This project has been proposed several years ago, and he was chosen by one student last year, but there is still work to finish.

The problem that motivated this project is : how detect harmful distributions and bugged one send on PYPI ? The idea to answer this question is to execute tests in a controlled environment in order to detect harmful behavior. These tests include installation, unit tests, quality indicators, signals send by unit test (like SIGFAULT). We decided that the controlled environment will be Virtual Machines rather than sandboxing, indeed if the VM is compromised, it is sufficient to recreate it; if sandboxing can not contain malicious codes, the host machine will be compromised and it will be more problematic. Moreover, VM will not have access to network, so malicious codes will not be able to make network attacks. We may note that detection of harmful behavior may resemble to virus detection, so this subject will be complex to implement and form the heart of the project.

I'll not detail every different parts that will be worked during the project, i invite you to take a look at my proposal at : <http://www.google-melange.com/gsoc/proposal/review/google/gsoc2011/lothiraldan/1>.

If you want to participate, you can contact this mailing-list : <the.fellowship.of.the.packaging@librelist.com>. We also need your help to get some statistics about your python distributions dependencies and about which indicators you want to have about them, thank you taking 2 minutes to fill out this google moderator : <http://www.google.com/moderator/#16/e=62dc1>.
