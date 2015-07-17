---
layout: post
title: "Performance: Entity Framework 7 vs. Dapper.net vs. raw ADO.NET"
excerpt: "Performance comparison of the upcoming Microsoft Entity Framework 7 (beta 4) against dapper.net and manual ADO.NET in ASP.NET 5. With some performance advice using Entity Framework."
---

*(Update 17 July 2015: Added benchmark for EF using `AsNoTracking` as per [this advice](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1). Yes, it makes things faster :))*

## TL;DR

- 	Entity Framework 7 (EF7) is coming soon, it's currently in Beta 4. See 
	[here](https://github.com/aspnet/EntityFramework/wiki/What-is-EF7-all-about) 
	and 
	[here](http://blogs.msdn.com/b/adonet/archive/2014/11/12/visual-studio-2015-preview-and-entity-framework.aspx).

- 	And it's still not very fast. More specifically:

- 	It's **6x to 3x slower** than either [dapper.net] or raw manual ADO.NET

- 	DO use `AsNoTracking` as per [this advice](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1) for read-only operations. See [how you can do it here](https://github.com/ppanyukov/EntityFrameworkBenchmarks/commit/3fb139a0a7fcb5e3e97ac75c6c191cfd3d9e2732).

-	(aside): [dapper.net] is still great.

-	The source code for benchmarks is [here on github][project].

-	View and leave commends [here on github][comments].



## The benchmark

I don't think Entity Framework ever had a great reputation from performance point of view.

I wanted to see if the upcoming EF7 has made sudden improvements and compare it with raw 
ADO.NET (writing code by hand) and [dapper.net].

The bechmark:

- 	A simple ASP.NET 5 Web Api project with several endpoints each doing a simple
	`select name, type ... from sys.objects` and returing data in JSON format. 

	Endpoints implemented using:

	- 	Entity Framework 7; 
	- 	[dapper.net];
	- 	raw ADO.NET code written by hand; and
	- 	serving hard-coded JSON response from memory.
<br/>
<br/>


-	I used [`ab`][ab] benchmarker to see how many requests per second I get from each endpoint.

- 	All running on local (and pretty powerful) local dev machine.


**Brief benchmark results (best recorded run)**

- 	Entity Framework 7 (beta 4): 50 req/sec
- 	Entity Framework 7 (beta 4) [with **AsNoTracking**](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1): 114 req/sec
-	[Dapper.net][dapper.net]: 340 req/sec
-	Raw ADO.NET (all code writting by hand): 380 req/sec
- 	Serving JSON from memory: 1800 req/sec



### Full benchmark results

<!-- Re-ran on 17 July -->

**Entity Framework 7 (beta 4)**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /ef/sys.objects
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   39.482 seconds
	Complete requests:      2000
	Failed requests:        0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    50.66 [#/sec] (mean)
	Time per request:       19.741 [ms] (mean)
	Time per request:       19.741 [ms] (mean, across all concurrent requests)
	Transfer rate:          832.90 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:    15   20  18.3     17     396
	Waiting:       14   19  18.3     16     396
	Total:         15   20  18.2     17     396

	Percentage of the requests served within a certain time (ms)
	  50%     17
	  66%     17
	  75%     17
	  80%     18
	  90%     19
	  95%     28
	  98%     38
	  99%     79
	 100%    396 (longest request)


**Entity Framework 7 - using AsNoTracking (beta 4)** 

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /ef/sys.objects/asnotracking
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   17.485 seconds
	Complete requests:      2000
	Failed requests:        0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    114.39 [#/sec] (mean)
	Time per request:       8.742 [ms] (mean)
	Time per request:       8.742 [ms] (mean, across all concurrent requests)
	Transfer rate:          1880.77 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     6    9  16.7      7     447
	Waiting:        6    8  13.6      7     418
	Total:          6    9  16.7      7     447

	Percentage of the requests served within a certain time (ms)
	  50%      7
	  66%      8
	  75%      8
	  80%      8
	  90%      8
	  95%      9
	  98%     11
	  99%     21
	 100%    447 (longest request)


**Dapper.net**


	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /dapper/sys.objects
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   5.766 seconds
	Complete requests:      2000
	Failed requests:        0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    346.84 [#/sec] (mean)
	Time per request:       2.883 [ms] (mean)
	Time per request:       2.883 [ms] (mean, across all concurrent requests)
	Transfer rate:          5702.90 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     2    3   4.5      2     158
	Waiting:        1    2   4.5      2     158
	Total:          2    3   4.5      3     158

	Percentage of the requests served within a certain time (ms)
	  50%      3
	  66%      3
	  75%      3
	  80%      3
	  90%      3
	  95%      4
	  98%      4
	  99%      6
	 100%    158 (longest request)


**Raw manual ADO.NET**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /manual/sys.objects
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   5.151 seconds
	Complete requests:      2000
	Failed requests:        0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    388.25 [#/sec] (mean)
	Time per request:       2.576 [ms] (mean)
	Time per request:       2.576 [ms] (mean, across all concurrent requests)
	Transfer rate:          6383.80 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     2    2   0.6      2      11
	Waiting:        1    2   0.5      2      10
	Total:          2    3   0.7      2      11
	WARNING: The median and mean for the total time are not within a normal deviation
	        These results are probably not that reliable.

	Percentage of the requests served within a certain time (ms)
	  50%      2
	  66%      3
	  75%      3
	  80%      3
	  90%      3
	  95%      3
	  98%      3
	  99%      4
	 100%     11 (longest request)



**Serving hardcoded string from memory**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /memory/sys.objects
	Document Length:        19446 bytes

	Concurrency Level:      1
	Time taken for tests:   1.065 seconds
	Complete requests:      2000
	Failed requests:        0
	Total transferred:      39198000 bytes
	HTML transferred:       38892000 bytes
	Requests per second:    1877.82 [#/sec] (mean)
	Time per request:       0.533 [ms] (mean)
	Time per request:       0.533 [ms] (mean, across all concurrent requests)
	Transfer rate:          35940.91 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     0    0   0.6      0      15
	Waiting:        0    0   0.5      0       1
	Total:          0    0   0.6      0      15

	Percentage of the requests served within a certain time (ms)
	  50%      0
	  66%      1
	  75%      1
	  80%      1
	  90%      1
	  95%      1
	  98%      1
	  99%      1

<hr/>

See all [changes to this post here](https://github.com/ppanyukov/ppanyukov.github.com/commits/master/_posts/2015-05-20-entity-framework-7-performance.md).



[dapper.net]: https://github.com/StackExchange/dapper-dot-net "dapper.net github project"
[ab]: http://en.wikipedia.org/wiki/ApacheBench "Wikipedia - ApacheBench tool"
[project]: https://github.com/ppanyukov/EntityFrameworkBenchmarks "Source code for benchmarks"
[comments]: https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues?q= "Comments for the post and code"

