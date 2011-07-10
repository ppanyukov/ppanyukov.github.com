Here is a sequence how to create a DVD image from AVI file on Linux (Ubuntu).

The sequence is:

        AVI -> MPEG2 -> DVD File System -> DVD ISO -> DVD Disk

Tools:

    -   ffmpeg    (AVI -> MPEG2)
    -   dvdauthor (MPEG2 -> DVD File System)
    -   mkisofs   (DVD File System -> DVD ISO)
    -   growisofs (DVD ISO -> DVD)


1. Export the video format as an environmental variable, otherwise there may
   be problems with the DVD file system.

        export VIDEO_FORMAT=PAL

2. Encode _AVI_ file into _MPEG2_. This will take a while!

        ffmpeg -i <input_avi> -target pal-dvd -ps 2000000000 -aspect 16:9 output.mpg

3. Create a DVD file sytem from the MPEG2 file. (NOTE: delete the target directory first).

        dvdauthor -o <output_dir> -t -v "PAL+16:9" -f output.mpg

4. Create a DVD ISO image from the directory.

        mkisofs -dvd-video -o dvd.iso dvd/

5. Burn the DVD ISO onto disk

        growisofs -dvd-compat -Z /dev/dvd=dvd.iso


