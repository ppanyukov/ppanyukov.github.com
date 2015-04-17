---
layout: post
title: Resolving host names on Windows, Linux and Mac
---

# Resolving host names on Windows, Linux and Mac #

Here is the setup. We have a Windows-based network with AD, DHCP, WINS and DNS
servers. We have Windows, Linux and Mac hosts on wired and wireless networks.
The hosts acquire IP addresses using DHCP. 

The task. Be able to ping all hosts using host name.

When Windows hosts acquire IP address using DHCP, a record for the host is
created in the DNS. So from any host we can do `nslookup mywindowshost` and
it works, and so `ping` works too. Note, these Windows hosts are joined into
the domain so that may have some bearing on this, don't know.

The problem is with the Linux and Mac hosts. By default when these get IP
address from DHCP server they don't get the DNS record created for them. As
a consequence, if I do `nslookup ubuntuhost` or `nslookup machost` I get back
nothing. Also, by default we can't ping those from Windows machines either.

It is possible to use bonjour to resolve the Linux and Mac hosts but this does
not work between wireless and wired hosts because I think those two networks
are joined by switches and I guess bonjour only works on the same segment?
I don't know much about this though, I may be wrong. Another problem with
bonjour is that by default it's not present on Windows hosts.

So here is the summary:

    Source host     Target host     Command            Status
    ---------------------------------------------------------
    Windows         Windows         ping host          OK
    Windows         Windows         ping host.local    XX 

    Ubuntu          Windows         ping host          OK
    Ubuntu          Windows         ping host.local    XX

    Ubuntu          Ubuntu          ping host          XX
    Ubuntu (wired)  Ubuntu (wired)  ping host.local    OK
    Ubuntu (wired)  Ubuntu (wi-fi)  ping host.local    XX

    Mac             Mac             ping host          XX
    Mac (wired)     Mac (wired)     ping host.local    OK
    Mac (wired)     Mac (wi-fi)     ping host.local    XX

    Ubuntu          Mac             ping host          XX
    Mac             Ubuntu          ping host          XX
    Ubuntu (wired)  Mac    (wired)  ping host.local    OK
    Mac    (wired)  Ubuntu (wired)  ping host.local    OK
    Ubuntu (wired)  Mac    (wi-fi)  ping host.local    XX
    Mac    (wired)  Ubuntu (wi-fi)  ping host.local    XX
    ---------------------------------------------------------
    NOTE: host.local means using bonjour protocol

The picture is quite patchy, and I want to fix this.

The idea is to use NetBIOS-to-IP name resolution and the task consists of two parts:

1.  Make host pingable using NetBIOS name from Windows.
2.  Make Linux and Mac hosts ping other hosts using NetBIOS.

Interestingly, the job turned out much easier on Ubuntu than on a Mac.

## Linux (Ubuntu) ##

First, make the Linux host pingable from Windows. Simply install `samba`
package. It will install `nmbd` daemon which seems to be the one that makes
this work.

    # get samba
    sudo apt-get install samba

    # verify the thing is running
    ps -ef | grep nmbd

Now doing `ping ubuntuhost` from Windows should work. But this is not
sufficient to make ping work on Linux. The second step is required.

Second step is to install a WINS resolution thing which is provided by
`winbind` package and bit of config file fiddling. Just follow these steps:

    # install winbind
    sudo apt-get install winbind

    # verify the winbindd daemon is running
    ps -ef | grep winbindd

    # add WINS to the list of resolves in /etc/nsswitch.conf file
    # here is the line to modify, simply add "wins" to the end.
    hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4 wins

    # now we should be able to ping using NetBIOS name:
    ping netbioshost

After installing `winbind` I can now ping Linux host which have samba installed
on them, or Mac hosts which have the same.


## Mac OS X Snow Leopard ##

This is surprising hard to get the same thing working on Mac. The first
step is easy and the second step, well, I still don't have a solution.

First, to make a Mac host pingable using NetBIOS name, I had to enable file
sharing and tick the "SMB" option. Here is exactly what I did:

    1. Open System Preferences
    2. Click "Sharing"
    3. Enable "File Sharing" by ticking the checkbox
    4. Click "Options" button there, and
    5. Make sure the "Share files and folders using SMB (Windows)" is ticked.

Now you should have `nmbd` daemon running. Verify this by opening terminal and:

    ps -ef | grep nmbd

After this I can ping my Mac host from Windows and Linux which has samba installed.

Now to be able to ping hosts using NetBIOS name from Mac -- that's a completely
different story. Basically, there is no `/etc/nsswitch.conf` file. And there is
no `winbindd` daemon running (even though it's there). I don't have sufficient
knowledge of OS X to get `winbindd` start at boot, but even starting manually
I got some weird errors (which I fixed) but then there is no
`/etc/nsswitch.conf` file so the ping still didn't work. But anyway here is
what I did:


    # first set the file permissions, as otherwise things don't work
    sudo chmod 750 /private/var/samba/winbindd_privileged

    # start winbindd without becoming a daemon and with output to terminal
    sudo winbindd -i

    # try ping my ubuntu host, doesn't work!
    ping ubuntuhost

I have spent some time to try and find what's going on with the `nsswitch.conf`
file and how to add wins to the resolvers on Snow Leopard and couldn't find
anything. The only thing I can think of is use `nmblookup` to get the IP
address and then use IP address, e.g.:

    nmblookup ubuntuhost

If anyone knows how to get this working properly, drop me a line.


## Summary ##

On Linux, install `samba` and `winbind` packages, then edit
`/etc/nsswitch.conf` file. This will make the host pingable using NetBIOS names
and also it will be possible to ping other NetBIOS hosts.

    # get packages
    sudo apt-get install samba
    sudo apt-get install winbind
    
    # add wins to the /etc/nsswitch.conf
    hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4 wins



On Mac, make the host pingable by enabling file sharing for Windows clients.
This will start `nmbd` daemon. I couldn't figure out what needs to be done to
ping from Mac using NetBIOS. Still work in progress.

Here is the summary of the achievement:

    Source host     Target host     Command            Status
    ---------------------------------------------------------
    Windows         Windows         ping host          OK
    Windows         Ubuntu          ping host          OK
    Windows         Mac             ping host          OK

    Ubuntu          Ubuntu          ping host          OK
    Ubuntu          Windows         ping host          OK
    Ubuntu          Mac             ping host          OK

    Mac             Ubuntu          ping host          XX
    Mac             Mac             ping host          XX
    Mac             Windows         ping host          OK
    ---------------------------------------------------------
    NOTE: I didn't include bonjour here, as there is no point
    NOTE: Assume that Windows hosts are resolved by nslookup

If you know how to get this completed on a Mac, drop me a line at `ppanyukov at
googlemail.com`.


