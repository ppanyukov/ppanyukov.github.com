---
layout: post
title: "Performance: Remember, Azure VMs are not your local dev machines"
excerpt: "Performance benchmarks for different types of Azure VMs vs local dev machine and CPU information for each VM type."
---


------------

*(IMPORTANT: Everything here is based on **data from February 2015**. Things move fast so it may be different today.)*



**TL;DR:** VMs in Azure are 2x-4x slower than my local quad-core i7 dev machine. 
And they have really different CPUs depending what kind of VM you choose.
Only expensive G1 instance came close and even marginally outperformed in some cases.
Moral of the story: don't use your local dev machine to measure how fast your stuff is;
always use target production-like environment to measure performance
(kinda obvious but we often forget about this).

For comments, suggestions etc, please [open an issue here](https://github.com/ppanyukov/PerfMicroBenchmarks/issues?&q=).

Source code for benchmarks: [here](https://github.com/ppanyukov/PerfMicroBenchmarks/tree/1b6f6b6b3a0d45c9a3abe6483914da604312d73f/EnumDictionaryBenchmarks).


-------------

## The slowness

If you are a developer chance are you have a pretty powerful local dev machine: 
fast CPU, lots of RAM, SSD, and whatnot.

If you also run your stuff in the cloud (say Azure), 
it may be worth remembering that those (virtual) machines are probably
nothing like your super fast local dev box -- chances are those Azure VMs
are much, much slower.

It is surprising how surprising this may be to many of us devs as we probably 
don't tend to think about these matters a lot :)

It may also be surprising how different performance may depend on the the
kind of the VM you choose (A4, A4, D2, G1 etc).

So **back in February 2015** I did some rather specialised micro-benchmarks of .NET C# code
and as part of that I ran it on a bunch of different VMs in Azure

Ignoring the specifics of the benchmarks, the results are roughly: Azure VMs are 2x to 4x
times slower than my local dev box on these benchmarks.  There are also differences between
x86 and x64 code in some cases.

And the results are:

**Ops per second - x64**

<pre style="font-size:small">
| Machine                           | Array         | Array + Method call   | Enum          | String        | Byte          |
|---------------------------------  |-------------  |---------------------  |-----------    |-----------    |------------   |
| Local - Dell - x64                | 310,559,103   | 80,963,413            | 4,625,365     | 8,533,679     | 16,679,009    |
| Local - VM Win 8.1 - x64          | 311,254,883   | 78,005,950            | 4,392,417     | 7,977,267     | 12,429,547    |
| Azure - A4 - x64                  | 139,032,401   | 44,965,772            | 1,791,762     | 2,324,881     | 4,984,948     |
| Azure - D2 - x64                  | 193,927,058   | 51,473,355            | 2,828,105     | 4,769,452     | 8,525,123     |
| Azure - G1 - x64                  | 268,654,639   | 105,247,448           | 4,558,606     | 8,562,353     | 16,172,684    |
| Azure - A2 VS2015 Preview - x64   | 124,845,628   | 40,677,422            | 5,370,280     | 2,416,338     | 5,016,827     |
| Azure - D2 VS2015 Preview - x64   | 203,151,241   | 49,826,795            | 9,593,304     | 4,765,592     | 9,386,226     |
</pre>


**Ops per second - x86**

<pre style="font-size:small">
| Machine                           | Array         | Array + Method call   | Enum          | String        | Byte          |
|---------------------------------  |-------------  |---------------------  |-----------    |-----------    |------------   |
| Local - Dell - x86                | 193,840,636   | 82,422,700            | 6,069,700     | 8,075,636     | 17,371,795    |
| Local - VM Win 8.1 - x86          | 193,665,549   | 75,740,924            | 5,563,796     | 7,302,632     | 16,366,795    |
| Azure - A4 - x86                  | 80,264,062    | 33,080,094            | 2,050,821     | 1,992,817     | 5,379,848     |
| Azure - D2 - x86                  | 113,727,798   | 48,343,017            | 3,470,515     | 4,648,138     | 9,841,217     |
| Azure - G1 - x86                  | 213,396,112   | 88,654,455            | 6,257,989     | 8,363,788     | 17,001,400    |
| Azure - A2 VS2015 Preview - x86   | 81,389,090    | 32,912,895            | 4,990,778     | 2,032,477     | 5,208,128     |
| Azure - D2 VS2015 Preview - x86   | 108,289,273   | 49,189,911            | 9,338,231     | 4,304,893     | 9,159,962     |
</pre>


The results look pretty consistent across all benchmarks.

[The benchmarks were run off this version of code on GitHub.](https://github.com/ppanyukov/PerfMicroBenchmarks/tree/1b6f6b6b3a0d45c9a3abe6483914da604312d73f/EnumDictionaryBenchmarks)
If you spot any bugs or have any questions regarding these, please [open an issue here](https://github.com/ppanyukov/PerfMicroBenchmarks/issues?&q=)


----------

## Those pesky CPUs

I also took a peek at what kind of CPU each of the machines has.

Obviously these may be completely different now.


**Local machine: Dell Precision T1650**

![Local machine: Dell Precision T1650]({{ site.images }}/2015-07-23-azure-cpus/local.png)


<br/>
**Azure: A4**

![Azure: A4]({{ site.images }}/2015-07-23-azure-cpus/a4.png)


<br/>
**Azure: D2**

![Azure: D2]({{ site.images }}/2015-07-23-azure-cpus/d2.png)



<br/>
**Azure: G1**

![Azure: G1]({{ site.images }}/2015-07-23-azure-cpus/g1.png)


<br/>
**Azure: A2 (01 Aug 2015)**

![Azure: A2]({{ site.images }}/2015-07-23-azure-cpus/a2-01aug2015.png)

--------
<br/>
For comments, suggestions etc, please [open an issue here](https://github.com/ppanyukov/PerfMicroBenchmarks/issues?&q=).

