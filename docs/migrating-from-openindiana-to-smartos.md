+--------------------------------------------------------------------------+
<div class="pageheader">

<span class="pagetitle"> SmartOS Documentation : Migrating from
OpenIndiana to SmartOS </span>

</div>

<div class="pagesubheading">

This page last changed on Mar 15, 2013 by
<font color="#0050B2">calmh</font>.

</div>

The Goal
============

To migrate my main [OpenIndiana](http://openindiana.org) box to
[Joyent's](http://joyent.com) [SmartOS](http://smartos.org).

### The Motivation

My reason? I want to run an ﻿[illumos](http://illumos.org) distribution
with clear focus on a server OS with active development.\
Also the people at Joyent went with ﻿[NetBSDs](http://netbsd.org)
**pkgsrc(7)** for building and maintaining third-party software instead
of IPS.\
Having also run ﻿[FreeBSD](http://freebsd.org) for many years,
﻿[SmartOS](http://smartos.org) just felt like a perfect match for my
needs and interests :)

### History

When I decided to run OpenIndiana as my storage box a few years ago,
SmartOS wasn't around.\
Quickly it became evident that the development of OpenIndiana became
slow and the IPS package repositories didn't give me the packages I was
looking for.

The Excercise (a.k.a. getting busy)
=======================================

<div class="panelMacro">

  ---------------------------------------------------------------------
------------------------------------------------------------------------
----------------
  ![](images/icons/emoticons/information.gif){width="16" height="16"}
**Used smartos-20130222T000747Z**\

All the steps described below are performed using the smartos-20130222T0
00747Z DVD ISO
  ---------------------------------------------------------------------
------------------------------------------------------------------------
----------------

</div>

This weekend I finally started the migration from OpenIndiana to
[SmartOS](http://smartos.org).\
I was running OI on a single vdev mirrored rpool of 2 disks.\
I also use a single vdev 3-disk raidz zpool for storage and a single
vdev single disk backup pool (sata drive in a usb enclosure).

After making the necessary backups to my backup zpool it was finally
time to start my journey :)\
Under OI my zpools used the following controller and targets:

c3t1d0s0 and c3t0d0s0 was my mirrored rpool (rpool)\
c3t2d0, c3t3d0 and c3t4d0 was my raidz pool (pool1)\
c4t0d0p0 was my external backup pool (backup)

I went with the latest SmartOS image available which at this time is
smartos-20130222T000747Z and can be downloaded from
﻿[download.joyent.com](http://download.joyent.com/pub/iso/).\
My system has problems booting from usb so I got the DVD ISO, burned it
and booted from it.\
I chose the no-install option from the SmartOS menu to verify on which
controller and targets SmartOS would find my disks.

zpool import showed the following:

rpool using c1t0d0s0 and c1t1d0s0\
pool1 using c1t2d0 , c1t3d0 and c1t4d0\
backup using c0t0d0p0

I find it interesting why SmartOS found the disks to use different
controller ids than OI!

Since I wanted a path back to OI in case SmartOS on my box wouldn't
properly install or run I decided to split my OI rpool.\
So rebooting back into OI, then it was time to detach disk 2 of the
rpool mirror leaving disk 1 intact:

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    # zpool detach rpool c3t0d0s0

</div>

</div>

<div class="panelMacro">

  ------------------------------------------------------------------- --
------------------------------------------------------------------------
------------------------------------------------------------------------
-------
  ![](images/icons/emoticons/forbidden.gif){width="16" height="16"}   **
Warning**\
                                                                      Ch
eck and double check which disk you are detaching to not wipe out the wr
ong disk and thus your OI install by giving this disk to the SmartOS ins
tall\
                                                                      If
 SmartOS finds a zpool on a disk it might have difficulty or simply won'
t allow this disk to be installed onto.
  ------------------------------------------------------------------- --
------------------------------------------------------------------------
------------------------------------------------------------------------
-------

</div>

Before rebooting back to SmartOS I decided to write out some zeroes to
the beginning of the disk so the disk would be clear to install to.

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    # dd if=/dev/zero of=/dev/rdsk/c3t0d0s0 bs=1024 count=5

</div>

</div>

Now we've taken care of that is was time to reboot back into SmartOS
again to choose the install option.\
Now to provide my choices for the very few questions SmartOS asks which
is super simplistic and great :)

- assigning a static IP to the admin nic\
- install to c1t0d0\
- enter a password for root

It's trivial to attach a 2nd disk turning the SmartOS zones pool into a
mirror so I installed SmartOS to just 1 disk for now.

After a short while everything was setup and it was time to reboot into
the SmartOS live image.\
No problems booting and soon I was presented with the console login of
the live image.

Once I was logged into the global zone, a zpool list showed that all of
the zpools had been imported, including the now single disk OI rpool\
I decided to destroy the rpool of the OI install and attach that disk to
the zones pool ASAP.

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    [root@00-1b-21-98-51-c1 ~]# zpool destroy rpool

</div>

</div>

With that gone, we can turn the zones pool into a mirror.

This IMHO is imperative for a type 1 hypervisor such as SmartOS since
it's running entirely off of a ramdisk!

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    [root@00-1b-21-98-51-c1 ~]# zpool attach zones c1t0d0 c1t1d0

</div>

</div>

After the resilver of disk 2 was complete here's the status of the zones
pool:

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    [root@00-1b-21-98-51-c1 ~]# zpool status zones
      pool: zones
     state: ONLINE
      scan: resilvered 1.36G in 0h0m with 0 errors on Sat Mar  2 16:16:1
5 2013
    config:

            NAME        STATE     READ WRITE CKSUM
            zones       ONLINE       0     0     0
              mirror-0  ONLINE       0     0     0
                c1t0d0  ONLINE       0     0     0
                c1t1d0  ONLINE       0     0     0

</div>

</div>

Resilvering the newly attached disk completes quite quickly at this
point, since there is very little data on it.

I went for a reboot to verify if the system would come up properly.\
Reboot completed without any hitch, here's the complete listing of all
zpools on the system:

<div class="preformatted panel" style="border-width: 1px;">

<div class="preformattedContent panelContent">

    [root@00-1b-21-98-51-c1 ~]# zpool list
    NAME     SIZE  ALLOC   FREE  EXPANDSZ    CAP  DEDUP  HEALTH  ALTROOT
    backup  1.36T   112G  1.25T         -     8%  1.00x  ONLINE  -
    pool1   5.44T  2.62T  2.81T         -    48%  1.00x  ONLINE  -
    zones    232G  1.40G   231G         -     0%  1.00x  ONLINE  -

</div>

</div>

I noticed that, despite having zpool upgraded my zpools on the latest
development release of OI (oi\_151a7) to use **zpool-features(5)** ,
running zpool upgrade on SmartOS still showed that pools pool1 and
backup could be upgraded!\
Recently some excellent work has gone into illumos implementing lz4
compression and filesystem\_limits into **ZFS(1M)** which were
implemented by Joyent to their SmartOS builds.\
This just warranted my decision from migrating from OI to SmartOS even
more.

### The verdict

I feel the migration was a success!\
Since the migration I've setup a couple of zones for mail and irc and
will be installing several more.

All zones are using the 12.4.1 multiarch
(<http://pkgsrc.smartos.org/packages/SmartOS/2012Q4-multiarch/All>)
repository provided by Joyent's ﻿[@jperkin](http://twitter.com/jperkin)
(<http://www.perkin.org.uk/posts/multiarch-package-support-in-smartos.ht
ml>)
+--------------------------------------------------------------------------+

  ----------------------------------------------------------------------------------
  ![](images/border/spacer.gif){width="1" height="1"}
  <font color="grey">Document generated by Confluence on Jul 07, 2019 00:15</font>
  ----------------------------------------------------------------------------------


