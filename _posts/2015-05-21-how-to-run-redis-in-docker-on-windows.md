---
layout: post
title: "How-To: Run Redis in Docker on Windows"
excerpt: "A walkthrough for installing and running Redis in Docker container on Windows."
---

Contents:

- [Summary](#summary)
- [Walkthough](#walkthrough)
	1. [Run Boot2Docker](#w-run-Boot2Docker)
	2. [Run Redis images](#w-run-redis-images)
	3. [Confirm the images are running](#w-confirm-images)
	4. [Get Redis client for Windows](#w-get-redis-client)
	5. [Connect to Redis running in Docker](#w-connect-to-redis)
		1. [Find out the IP address of the Docker VM](#w-connect-to-redis-ip)
		2. [Find out the ports to use](#w-connect-to-redis-ports)
		3. [Finally -- connect](#w-connect-to-redis-connect)
	
- [Why Docker](#why-docker)


## Summary
<a name="summary"></a>

This is a very condensed summary.

**1. Run Redis instances**

From Docker command line:

	$ docker run -d -p 6379:6379 redis:3.0.1
	$ docker run -d -p 6380:6379 redis:2.8.20
	$ docker run -d -p 6381:6379 redis:2.8.12

	
**2. Connect to them using redis-cli**

Find out Docker VM IP:

		$ boot2docker.exe ip
		192.168.59.103

Connect using `redis-cli`:

		# Redis 3.0.1
		redis-cli -h 192.168.59.103 -p 6379
		
		# Redis 2.8.20
		redis-cli -h 192.168.59.103 -p 6380
		
		# Redis 2.8.12
		redis-cli -h 192.168.59.103 -p 6381




## Walkthrough
<a name="walkthrough"></a>

A more detailed version.

Assuming Docker is already installed. 
If not, here is [How to install Docker on Windows](/2015/05/21/how-to-install-docker-on-windows.html) post.

Here is what I want to do:

- Run 3 instances of Redis on Linux:
	- v 2.8.12
	- v 2.8.20
	- v 3.0.1
	
- Use `redis-cli` on Windows to connect to them

Here are the steps:

<a name="w-run-Boot2Docker"></a>
**1. Run Boot2Docker**

This will get us the Docker command prompt.


<a name="w-run-redis-images"></a>
**2. Run Redis images**

From the Docker command prompt, run these

	$ docker run -d -p 6379:6379 redis:3.0.1
	$ docker run -d -p 6380:6379 redis:2.8.20
	$ docker run -d -p 6381:6379 redis:2.8.12

The `-p` switch is very important. It has format `[external IP : internal IP]` and 
allows us to get to things inside Docker container.

For example, `"-p 6381:6379"` means that Docker VM will listen on port `6381` and forward
all traffic on that port to whatever is listening on port `6379` in the container.  

In case of our Redis instances, all three of them listen on port 6379 *inside the container*. 
But to get to them from outside, we need to tell Docker to forward exteranl traffic to them
and we do this by providing this mapping.

If you don't specify this mapping, you won't be able to get to the Redis instance
from outside (at least I couldn't).

The `-d` switch is not important. It just means "run in background".


<a name="w-confirm-images"></a>
**3. Confirm the images are running**

From the Docker command prompt, run this:

`$ docker ps -a`

If everything is well, you should see output like this:


	CONTAINER ID        IMAGE               COMMAND                CREATED             STATUS                           PORTS                    NAMES
	3dcb8275035f        redis:2.8.12        "/entrypoint.sh redi   7 minutes ago       Up 7 minutes                     0.0.0.0:6381->6379/tcp   admiring_pike
	bdf186ade5f3        redis:2.8.20        "/entrypoint.sh redi   7 minutes ago       Up 7 minutes                     0.0.0.0:6380->6379/tcp   serene_brattain
	bb7b03c03c21        redis:3.0.1         "/entrypoint.sh redi   8 minutes ago       Up 8 minutes                     0.0.0.0:6379->6379/tcp   nostalgic_kirch

Note the columns:

- `IMAGE`. This has the name of the image we asked Docker to run earlier, together with version.
- `PORTS`. This is the effect of the `-p` switch. It shows the external->internal mapping of IP ports.
- `STATUS`. Tells whether the image is actually running.
- `NAMES`. These are auto-generate by Docker. You use these to control the running images with other Docker commands.


<a name="w-get-redis-client"></a>
**4. Get Redis client for Windows**

Two options:

- Using `nuget` from command prompt (assuming you have nuget.exe command line tool). E.g.:

		c:>cd \temp
		
		C:\Temp>nuget install Redis-64
		Installing 'Redis-64 2.8.19'.
		Successfully installed 'Redis-64 2.8.19'
		
		C:\Temp>cd Redis-64.2.8.19
		


- From MSOpenTech github repository here: https://github.com/MSOpenTech/redis/releases.

	Get the zip file or MSI.


		
<a name="w-connect-to-redis"></a>
**5. Connect to Redis running in Docker**

What you need to know:

- The IP address of the Docker VM (see above).
- The port of the Redis instance you want to connect to.

These two steps in turn.

<a name="w-connect-to-redis-ip"></a>
**5.1 Find out the IP address of the Docker VM**


This is the IP address to which we will need to point our `redis-cli` client.

Two options:	

- from `Boot2Docker` command prompt run `boot2docker.exe ip`. This will show the IP address:

		$ boot2docker.exe ip
		192.168.59.103

- Look for `DOCKER-HOST` string when you start `Boot2Docker`:
 
		...
		starting...
		Waiting for VM and Docker daemon to start...
		.o
		Started.
		...
		
		To connect the Docker client to the Docker daemon, please set:
		    export DOCKER_HOST=tcp://192.168.59.103:2376


<a name="w-connect-to-redis-ports"></a>
**5.2 Find out the ports to use**

Two options:

- Remeber the `-p` switch we used to run Redis instance.
	
		$ docker run -d -p 6381:6379 redis:2.8.12

	In this case we will need to use port `6381`.
	
	
- Use `"docker ps -a"` command from Docker command line and look at `PORTS` column.

	In our case, we will need the first port.
	
		IMAGE               PORTS                  
		redis:2.8.12        0.0.0.0:6381->6379/tcp 
		redis:2.8.20        0.0.0.0:6380->6379/tcp 
		redis:3.0.1         0.0.0.0:6379->6379/tcp 


<a name="w-connect-to-redis-connect"></a>
**5.3 Finally -- connect**

With all this information, to connect to Redis instances:

	# Version 3.0.1
	redis-cli -h 192.168.59.103 -p 6379
	
	# Version 2.8.20
	redis-cli -h 192.168.59.103 -p 6380
	
	# Version 2.8.12
	redis-cli -h 192.168.59.103 -p 6381

That's it :)

## Why Docker?
<a name="why-docker"></a>

If all of the above sounded complicated, consider these scenarios:

1. I'm on Windows and want to run Redis 3.0.
2. I'm on Windows and want to run a Linux version of Redis.
3. I want to run multiple instances of different versions of Redis on Linux.  



**Life without Docker**

Without docker those scenarios would require quite a bit more work:
- building a separate Linux VM, 
- understanding how it works, 
- then getting that particular version of Redis onto it 
(may be not so trivial depending on the distribution of Linux). 

It will require time plus knowledge of a particular Linux distribution and how to get Redis binaries onto it.

Now consider Docker.

**Life with Docker**

- Run multiple versions of Redis (or anything really) at the same time;
- No knowledge of Linux required;
- No need to figure out where to get distributions and Redis from;
- Takes only a few seconds to get up and running.

There are still a few things to learn but it's still much easier than
doing it yourself without Docker.



