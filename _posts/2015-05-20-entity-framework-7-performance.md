---
layout: post
title: "Performance: Entity Framework 7 vs. Dapper.net vs. raw ADO.NET"
excerpt: "Performance comparison of the upcoming Microsoft Entity Framework 7 (beta 7) against dapper.net and manual ADO.NET in ASP.NET 5. With some performance advice using Entity Framework."
---

*(Update 06 Aug 2015): Updated benchmarks with Beta-7 versions of everything)*

*(Update 17 July 2015: Added benchmark for EF using `AsNoTracking` as per [this advice](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1). Yes, it makes things faster :))*


## TL;DR

- 	Entity Framework 7 (EF7) is coming soon, it's currently in Beta 7. See 
	[here](https://github.com/aspnet/EntityFramework/wiki/What-is-EF7-all-about) 
	and 
	[here](http://blogs.msdn.com/b/adonet/archive/2014/11/12/visual-studio-2015-preview-and-entity-framework.aspx).

- 	And it's still not very fast. More specifically:

- 	It's **6x to 3x slower** than either [dapper.net] or raw manual ADO.NET

- 	DO use `AsNoTracking` as per [this advice](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1) for read-only operations. See [how you can do it here](https://github.com/ppanyukov/EntityFrameworkBenchmarks/commit/3fb139a0a7fcb5e3e97ac75c6c191cfd3d9e2732).

-	(aside): [dapper.net] is still great.

-	The source code for benchmarks is [here on github][project].

-	View and leave commends [here on github][comments].


## Brief benchmark results (best recorded run)

**Beta 7 everything (ran on 6 Aug 2015):**

(Note there does not seem to be EF-specific improvements in benchmarks, but everything is a little faster comparing with Beta 4. Looks like improvements are due to Asp.Net/CLR.)

- 	Entity Framework 7 (beta 7): 62 req/sec
- 	Entity Framework 7 (beta 7) [with **AsNoTracking**](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1): 120 req/sec
-	[Dapper.net][dapper.net]: 425 req/sec
-	Raw ADO.NET (all code writting by hand): 432 req/sec
- 	Serving JSON from memory: 2,666 req/sec



**Beta 4:**

- 	Entity Framework 7 (beta 4): 50 req/sec
- 	Entity Framework 7 (beta 4) [with **AsNoTracking**](https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues/1): 114 req/sec
-	[Dapper.net][dapper.net]: 340 req/sec
-	Raw ADO.NET (all code writting by hand): 380 req/sec
- 	Serving JSON from memory: 1,800 req/sec



## The benchmark

I don't think Entity Framework ever had a great reputation from performance point of view.

I wanted to see if the upcoming EF7 has made sudden improvements and compare it with raw 
ADO.NET (writing code by hand) and [dapper.net].

The bechmark:

- 	A simple ASP.NET 5 Web Api project with several endpoints each doing a simple
	`select name, type ... from sys.objects` and returing data in JSON format. 

	Endpoints implemented using:

	- 	Entity Framework 7 (beta 7); 
	- 	[dapper.net];
	- 	raw ADO.NET code written by hand; and
	- 	serving hard-coded JSON response from memory.
<br/>
<br/>


-	I used [`ab`][ab] benchmarker to see how many requests per second I get from each endpoint.

- 	All running on local (and pretty powerful) local dev machine.




### Full benchmark results - Beta 7

<!-- Re-ran on 06 Aug 2015-->

**Entity Framework 7 (beta 6)**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /ef/sys.objects
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   32.150 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    62.21 [#/sec] (mean)
	Time per request:       16.075 [ms] (mean)
	Time per request:       16.075 [ms] (mean, across all concurrent requests)
	Transfer rate:          1022.84 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.9      0      16
	Processing:     0   16   3.8     16      31
	Waiting:        0   16   4.2     16      31
	Total:          0   16   3.7     16      31

	Percentage of the requests served within a certain time (ms)
	  50%     16
	  66%     16
	  75%     16
	  80%     16
	  90%     16
	  95%     16
	  98%     31
	  99%     31
	 100%     31 (longest request)


**Entity Framework 7 - using AsNoTracking (beta 7)** 

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /ef/sys.objects/asnotracking
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   16.820 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    118.91 [#/sec] (mean)
	Time per request:       8.410 [ms] (mean)
	Time per request:       8.410 [ms] (mean, across all concurrent requests)
	Transfer rate:          1955.14 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   1.0      0      16
	Processing:     0    8   8.9     16     156
	Waiting:        0    8   9.0      0     156
	Total:          0    8   8.9     16     156

	Percentage of the requests served within a certain time (ms)
	  50%     16
	  66%     16
	  75%     16
	  80%     16
	  90%     16
	  95%     16
	  98%     16
	  99%     16
	 100%    156 (longest request)


**Dapper.net**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /dapper/sys.objects
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   4.705 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    425.04 [#/sec] (mean)
	Time per request:       2.353 [ms] (mean)
	Time per request:       2.353 [ms] (mean, across all concurrent requests)
	Transfer rate:          6988.61 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.7      0      16
	Processing:     0    2   5.5      0      17
	Waiting:        0    2   5.1      0      17
	Total:          0    2   5.5      0      17

	Percentage of the requests served within a certain time (ms)
	  50%      0
	  66%      0
	  75%      0
	  80%      0
	  90%     16
	  95%     16
	  98%     16
	  99%     16
	 100%     17 (longest request)


**Raw manual ADO.NET**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /manual/sys.objects
	Document Length:        16684 bytes

	Concurrency Level:      1
	Time taken for tests:   4.624 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      33674000 bytes
	HTML transferred:       33368000 bytes
	Requests per second:    432.56 [#/sec] (mean)
	Time per request:       2.312 [ms] (mean)
	Time per request:       2.312 [ms] (mean, across all concurrent requests)
	Transfer rate:          7112.27 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   1.1      0      16
	Processing:     0    2   5.4      0      16
	Waiting:        0    2   4.9      0      16
	Total:          0    2   5.5      0      16

	Percentage of the requests served within a certain time (ms)
	  50%      0
	  66%      0
	  75%      0
	  80%      0
	  90%     16
	  95%     16
	  98%     16
	  99%     16
	 100%     16 (longest request)

**Serving hardcoded string from memory**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /memory/sys.objects
	Document Length:        19446 bytes

	Concurrency Level:      1
	Time taken for tests:   0.750 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      39198000 bytes
	HTML transferred:       38892000 bytes
	Requests per second:    2666.66 [#/sec] (mean)
	Time per request:       0.375 [ms] (mean)
	Time per request:       0.375 [ms] (mean, across all concurrent requests)
	Transfer rate:          51038.93 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   1.0      0      16
	Processing:     0    0   2.0      0      16
	Waiting:        0    0   1.6      0      16
	Total:          0    0   2.2      0      16

	Percentage of the requests served within a certain time (ms)
	  50%      0
	  66%      0
	  75%      0
	  80%      0
	  90%      0
	  95%      0
	  98%     16
	  99%     16
	 100%     16 (longest request)


<hr/>

See all [changes to this post here](https://github.com/ppanyukov/ppanyukov.github.com/commits/master/_posts/2015-05-20-entity-framework-7-performance.md).



[dapper.net]: https://github.com/StackExchange/dapper-dot-net "dapper.net github project"
[ab]: http://en.wikipedia.org/wiki/ApacheBench "Wikipedia - ApacheBench tool"
[project]: https://github.com/ppanyukov/EntityFrameworkBenchmarks "Source code for benchmarks"
[comments]: https://github.com/ppanyukov/EntityFrameworkBenchmarks/issues?q= "Comments for the post and code"

