---
layout: post
title: "How-To: Configure number of concurrent Hive/Tez tasks in Hadoop and Azure HDInsight"
excerpt: "A list of settings which affect the number of Hive/Tez tasks which can run concurrently in Hadoop/HDInsight."
---

This directly affects Azure HDInsight but will apply to any other Hadoop installations.

The number of concurrent map/reduce tasks which are able to run on a Hadoop cluster
is determined by these things:

1. Available resources in the cluster:

    - vCores
    - RAM

2. The resources requested by the application:

    - vCores for each task;
    - RAM for each task.


The way to tweak how many concurrent map/reduce tasks you can run is to tweak
these paramteres somehow. Obviously we can't change the amount of available RAM
but we can change the number of vCores and the amount of RAM that applications request.

The problem is there are *lots and lots* of settings everywhere with some interesting
relationships and it's difficult to figure out which of these need to be changed
without breaking anything.  Forgetting to change something will either have no effect
at all, or will lead to jobs failing in mysterious ways.


The following is the list of settings which need to be changed in order to
be able to run more concurrent **Hive tasks on Tez**. I have confirmed these
to actually work reliably on a single-node A4 cluster in Azure HDInsight.


These settings do:

- Increase the number of vCores available on each node. This allows to run more
  tasks than there are actual physical cores on the node.

- Decrease the amount of RAM requested by the Hive/Tez application. This allows
  run more containers on each node.


The effect of these:

- More than one tasks can be launched per physical CPU core;
- The tasks are given less RAM and so more of them can run at the same time.



These are the settings to change in Ambari Web.
The changes are:

    - vCores: from 8 to 24
    - RAM for tasks: from 1536MB to 768

Tweak these accordingly to you own purpose.

NOTE: These will affect Hive queries running on *Tez*. For MapReduce style
of queries and other applications more tweaking may be required.

NOTE2: These probably can be changed on a per-query basis using `set <propName>=<propValue>` directly in Hive query.
I haven't tried this yet.


    YARN:
        Scheduler:
            yarn.scheduler.minimum-allocation-mb: 1536 -> 768
            yarn.nodemanager.resource.cpu-vcores: 8 -> 24

        Advanced yarn-site:
            YARN Container Maximum vCores: 8 -> 24

    Tez:
        General:
            tez.am.resource.memory.mb: 1536 -> 768
            tez.task.resource.memory.mb: 1536 -> 768

        Custom tez-site:
            (change JVM memory options, very important!)
            tez.am.java.opts: -Xmx1G -Xms1G  -> -Xmx560m -Xms560m 

    Hive:
        General:
            hive.tez.container.size: 1536 -> 768

            (change JVM memory options, very important!)
            hive.tez.java.opts: -Xmx1G -Xms1G  -> -Xmx560m -Xms560m



