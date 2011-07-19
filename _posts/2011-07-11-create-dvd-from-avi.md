---
layout: post
title: Burning DVD images from AVI files
---

## General sequence ##

My understanding is that  the general sequence is like this:

    AVI -> MPEG2 -> DVD File System -> DVD ISO -> DVD Disk

In order to do all this, we need:

-   tools to do each of the steps;
-   codecs to encode/decode video and audio embedded in the AVI/MPEG2 files.

This may not be so easy, especially on Windows, but is manageable on Linux.

## On Windows ##

So you are on Windows and want to burn that AVI files your friends gave you to
a DVD and watch it on telly. From what I remember, this is all very painful.
The problems generally are:

-   The AVI files may use codecs which you don't have;

-   The players you use (e.g. Windows Media Player) doesn't tell you which
    codecs you need, it just refuses to play the stuff.

-   No idea where to take those codecs even if you knew which ones you need.

-   Even if you have the codecs, you still need the software to create the DVD!
    And they are not free and all of those I used were beyond crap.

What to do? Here is a quick solution:

1.  Get yourself [Oracle VirtualBox](http://www.virtualbox.org/).

2.  Get yourself a distribution of Linux. [Ubuntu](http://www.ubuntu.com/download)

3.  Install it and follow instructions for Linux. Easy!

And besides, the benefit is learning Linux too.


# Creating DVD images from AVI files on Linux #
## Tools ##

    Tool      Step                          Apt-get package
    ---------------------------------------------------------
    ffmpeg    (AVI -> MPEG2)                gstreamer0.10-*
    dvdauthor (MPEG2 -> DVD File System)    dvdauthor
    mkisofs   (DVD File System -> DVD ISO)  genisoimage
    growisofs (DVD ISO -> DVD)              growisofs

To install these do from command line:

    sudo apt-get install gstreamer0.10.-* dvdauthor genisoimage growisofs


## Details ##

1. Export the video format as an environmental variable, otherwise there may
   be problems with the DVD file system.

    export VIDEO_FORMAT=PAL

2. Encode _AVI_ file into _MPEG2_. This will take a while!

    ffmpeg -i <input_avi> -target pal-dvd -ps 2000000000 -aspect 16:9 output.mpg

3. Create a DVD file system from the MPEG2 file. (NOTE: delete the target directory first).

    dvdauthor -o <output_dir> -t -v "PAL+16:9" -f output.mpg

    (NOTE: I'm not sure this command line works properly, I think the
    switch should be "-T" instead of "-t" but I haven't checked this yet.)

4. Create a DVD ISO image from the directory.

    mkisofs -dvd-video -o dvd.iso dvd/

5. Burn the DVD ISO onto disk

    growisofs -dvd-compat -Z /dev/dvd=dvd.iso





