---
layout: post
title: "Performance: Entity Framework 7 vs. Dapper.net vs. raw ADO.NET"
excerpt: "Performance comparison of the upcoming Microsoft Entity Framework 7 (beta 4) against dapper.net and manual ADO.NET in ASP.NET 5."
---

## TL;DR

- 	Entity Framework 7 (EF7) is coming soon, it's currently in Beta 4. See 
	[here](https://github.com/aspnet/EntityFramework/wiki/What-is-EF7-all-about) 
	and 
	[here](http://blogs.msdn.com/b/adonet/archive/2014/11/12/visual-studio-2015-preview-and-entity-framework.aspx).

- 	And it's still not very fast. More specifically:

- 	It's **6x slower** than either [dapper.net] or raw manual ADO.NET

-	(aside): [dapper.net] is still great.

-	The source code for benchmarks is [here on github][project].



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


**Brief benchmark results**

- 	Entity Framework 7 (beta 7): 50 req/sec
-	[Dapper.net][dapper.net]: 340 req/sec
-	Raw ADO.NET (all code writting by hand): 337 req/sec
- 	Serving JSON from memory: 1495 req/sec



### Full benchmark results


**Entity Framework 7 (beta 4)**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /ef/sys.objects
	Document Length:        19446 bytes

	Concurrency Level:      1
	Time taken for tests:   38.820 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      39198000 bytes
	HTML transferred:       38892000 bytes
	Requests per second:    51.52 [#/sec] (mean)
	Time per request:       19.410 [ms] (mean)
	Time per request:       19.410 [ms] (mean, across all concurrent requests)
	Transfer rate:          986.06 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:    14   19  25.6     17     530
	Waiting:       14   18  25.6     16     530
	Total:         15   19  25.6     17     530

	Percentage of the requests served within a certain time (ms)
	  50%     17
	  66%     17
	  75%     17
	  80%     17
	  90%     18
	  95%     19
	  98%     37
	  99%     52
	 100%    530 (longest request)



**Dapper.net**


	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /dapper/sys.objects
	Document Length:        19446 bytes

	Concurrency Level:      1
	Time taken for tests:   5.859 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      39198000 bytes
	HTML transferred:       38892000 bytes
	Requests per second:    341.33 [#/sec] (mean)
	Time per request:       2.930 [ms] (mean)
	Time per request:       2.930 [ms] (mean, across all concurrent requests)
	Transfer rate:          6533.03 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     2    3   0.9      3      31
	Waiting:        1    2   0.9      2      30
	Total:          2    3   0.9      3      31

	Percentage of the requests served within a certain time (ms)
	  50%      3
	  66%      3
	  75%      3
	  80%      3
	  90%      3
	  95%      4
	  98%      4
	  99%      5
	 100%     31 (longest request)


**Raw manual ADO.NET**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /manual/sys.objects
	Document Length:        19446 bytes

	Concurrency Level:      1
	Time taken for tests:   5.922 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      39198000 bytes
	HTML transferred:       38892000 bytes
	Requests per second:    337.70 [#/sec] (mean)
	Time per request:       2.961 [ms] (mean)
	Time per request:       2.961 [ms] (mean, across all concurrent requests)
	Transfer rate:          6463.53 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     2    3   0.8      3      18
	Waiting:        1    2   0.7      2      18
	Total:          2    3   0.8      3      18

	Percentage of the requests served within a certain time (ms)
	  50%      3
	  66%      3
	  75%      3
	  80%      3
	  90%      3
	  95%      4
	  98%      4
	  99%      4
	 100%     18 (longest request)



**Serving hardcoded string from memory**

	Server Software:        Microsoft-HTTPAPI/2.0
	Server Hostname:        localhost
	Server Port:            5000

	Document Path:          /memory/sys.objects
	Document Length:        19446 bytes

	Concurrency Level:      1
	Time taken for tests:   1.337 seconds
	Complete requests:      2000
	Failed requests:        0
	Keep-Alive requests:    0
	Total transferred:      39198000 bytes
	HTML transferred:       38892000 bytes
	Requests per second:    1495.81 [#/sec] (mean)
	Time per request:       0.669 [ms] (mean)
	Time per request:       0.669 [ms] (mean, across all concurrent requests)
	Transfer rate:          28629.37 [Kbytes/sec] received

	Connection Times (ms)
	              min  mean[+/-sd] median   max
	Connect:        0    0   0.3      0       1
	Processing:     0    0   0.8      0      25
	Waiting:        0    0   0.7      0      25
	Total:          0    1   0.8      1      25

	Percentage of the requests served within a certain time (ms)
	  50%      1
	  66%      1
	  75%      1
	  80%      1
	  90%      1
	  95%      1
	  98%      1
	  99%      2
	 100%     25 (longest request)



[dapper.net]: https://github.com/StackExchange/dapper-dot-net "dapper.net github project"
[ab]: http://en.wikipedia.org/wiki/ApacheBench "Wikipedia - ApacheBench tool"
[project]: https://github.com/ppanyukov/EntityFrameworkBenchmarks "Source code for benchmarks"