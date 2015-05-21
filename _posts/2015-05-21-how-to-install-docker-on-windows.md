---
layout: post
title: "How-To: Install Docker on Windows"
excerpt: "A walkthrough for installing Docker container on Windows."
---

My environment: Windows Server 2012 R2. Slightly different steps may be required with other 
versions of Windows.

Pretty straightforward, the only tricky parts here are about Hyper-V and finding out the IP 
address of the VM.

With regards to the latter there is a lot of misleading information on the web. 

**1. Make sure Hyper-V is not running.** 

If it does, it will prevernt the VirtualBox VM to start and you will get mysterious errors.
Same goes if you use VMWare or any other hypervisor other than VirtualBox. 

If you do need to have both Hyper-V and VirtualBox, [Scott Hanselman wrote about some voodoo magic
you can do to easily switch between the two](http://www.hanselman.com/blog/SwitchEasilyBetweenVirtualBoxAndHyperVWithABCDEditBootEntryInWindows81.aspx). I haven't tried it.

**2. Download and install Docker for Windows.**

- 	Download and install Boot2Docker from here: https://github.com/boot2docker/windows-installer/releases/latest

-	Install everthing

-	Tick "yes I trust that thing" when asked to install network driver. It's required by VirtualBox.


-	You should have two icons added to the desktop:

	- Boot2Docker Start
	- Oracle VM VirtualBox
	
	![Desktop icons]({{ site.images }}/2015-05-21-how-to-install-docker-on-windows/01-docker-desktop-icons.PNG)


**3. Run Boot2Docker**

If everything goes well, you should see:

- bash command window should open
- after a bit, it should have ouput similar to the one shown below
- take note of the line which says `DOCKER-HOST`, it should have the IP address of the docker VM.
- if you open Virtual Box, you should see `boot2docker-vm` running there.

	![Bash window]({{ site.images }}/2015-05-21-how-to-install-docker-on-windows/02-01-bash-window.PNG)
	
	![Running docker vm]({{ site.images }}/2015-05-21-how-to-install-docker-on-windows/02-02-running-docker-vm.png)


**4. Post Install**

A couple of useful things to know:

- 	You can run as many instances of `Boot2Docker` as you want. It's just a command prompt.

- 	Finding out the IP address of the docker VM (and things which run within it). Several options:	

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

- 	Security and the images. One thing worth remembering is that when you say something like
	`docker run some-image` by default it will try and `some-image` from *external repository*.
	Which repository it is, and who built that image with what is a good question. So probably a
	good idea to install only "trusted" images.

