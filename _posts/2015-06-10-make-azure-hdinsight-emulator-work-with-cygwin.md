---
layout: post
title: Make Azure HDInsight Emulator work with cygwin on Windows
---


If you want to use Azure HDInsight (Hadoop) Emulator on Windows with
cygwin's bash, it won't work. Even though the emulator comes with
all legitimately-looking *nix scripts and everything.

The reason it doesn't work is those *nix things assume too much of 
*nix stuff which is not true on a Windows machine.

The easiest fix is to create `hadoop2` bash script which will call out to proper
Windows `hadoop.cmd` script.

Here is the source:

	#!/usr/bin/env bash
	 
	# A hack to make HDInsight Emulator work with cygwin bash.
	# Basically, we don't use bash things which come with the installation.
	# Instead, we call out to the Windows cmd scripts to invoke hadoop stuff.
	 
	cmd /c "hadoop" $*


Just create `hadoop2` file, put it somewhere in the `PATH`, give it execute permissions 
using `chmod u+x hadoop2` and you are good to go. 

