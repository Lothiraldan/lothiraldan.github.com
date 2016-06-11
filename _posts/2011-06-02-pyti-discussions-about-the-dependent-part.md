---
layout: post
title: PYTI discussions about the dependent part
tag:
    - GSOC
    - Python
---

Last week has been dedicated to discussions about the dependent part of PYTI.

The PYTI project has been divided into two projects and we tried to make them independent, but they should communicate at one time.

These communication can be separated into two different types, which are resumed on next picture:

![schema](http://feldboris.alwaysdata.net/pictures/dependent_part.png) During this week, they were two discussions in parallel. We thought we can send the setup part (distributions + configuration) by sending a big archive using ftp or sftp. For the raw data api, we thought sending raw data using a jsonrpc server. During Thursday meeting, our mentors say that we should use a shared folder for sending data from slave to vm and in the other way.

I would like to discuss this choice for both API.

Setup API
---------

### Using FTP/SFTP

#### Pro:

> -   When the archive is sent, vm cannot access slave anymore.

#### Cons:

> -   Archive must be sent and read by the vm, slower than the "shared folder" method.

### Using shared folder

#### Pro:

> -   Archives are not copied, they're directly read on the "shared folder".

#### Cons:

> -   No cons if the "shared folder" is read-only. If not same cons than Raw Data API (below).

Raw Data API
------------

### Using sockets

#### Pro:

> -   Data must be formatted.

#### Cons:

> -   Requires signatures in order to avoid tasks which send bad request to server.

### Using shared folder

#### Pro:

> -   Easy to implement, easy to use.

#### Cons:

> -   Cons, nothing prevent tasks to create a lot of files (if "shared folder" is limited, tasks can reaches this limit).
> -   Shared folder may suffer security problems (IE http://kb.vmware.com/selfservice/microsites/search.do?language=en\\\_US&cmd=displayKC&externalId=1004034\_).

Conclusion
----------

My feeling about last week discussions is that we've done twice the job, first time between me and yeswanth and second time between our mentors. We must organize ourself in order to be more efficient for next discussions.

For a detailed summary of Thursday meeting: http://titanpad.com/BLy0Ww35Rr\_ and what we've discussed with yeswanth: http://titanpad.com/MqCJzL6xRI\_.
