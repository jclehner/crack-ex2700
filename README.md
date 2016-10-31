Netgear EX2700 OpenWrt flash procedure
======================================

**For up-to-date images, use official [OpenWRT](https://downloads.openwrt.org/snapshots/trunk/ramips/mt7620/openwrt-ramips-mt7620-ex2700-squashfs-factory.bin) / [LEDE](https://downloads.lede-project.org/snapshots/targets/ramips/mt7620/lede-ramips-mt7620-ex2700-squashfs-factory.bin) images**

**For up-to-date information, refer to the [OpenWRT wiki entry](https://wiki.openwrt.org/toh/netgear/netgear_ex2700).**


**The information posted below is not relevant anymore and should be ignored**, as the new images can be
flashed from the router's stock interface. 

----------------------------------------------------------------------------------
----------------------------------------------------------------------------------
----------------------------------------------------------------------------------





























First, using the web interface, update your EX2700 to the latest firmware
(version 1.0.1.8 as of 2016-01-17) - older versions *may* work though.

For the rest of this howto, we'll assume that your computer uses `192.168.1.2`,
while your EX2700 uses `192.168.1.1`.

The device runs a modified version of OpenWrt (KAMIKAZE, bleeding edge,
r18571) by the way, but we want vanilla, obviously!

## Enabling telnet

By default, telnet is disabled on this device, but there are numerous
`telnetenable` utilities, many of which don't work with this device. I've
used [NetgearTelnetEnable](https://github.com/insanid/NetgearTelnetEnable)
(binaries for Linux and Windows available 
[here](https://github.com/insanid/NetgearTelnetEnable/tree/master/binaries)).

Make sure to reset the router configuration before using `telnetenable`.
Replace `001122AABBCC` with the MAC address of your router.

```
$ ./telnetenable 192.168.1.1 001122AABBCC admin password
$ telnet 192.168.1.1
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
 === IMPORTANT ============================
  Use 'passwd' to set your login password
  this will disable telnet and enable SSH
 ------------------------------------------


BusyBox v1.4.2 (2015-06-05 10:24:33 CST) Built-in shell (ash)
Enter 'help' for a list of built-in commands.

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 KAMIKAZE (bleeding edge, r18571) ------------------
  * 10 oz Vodka       Shake well with ice and strain
  * 10 oz Triple sec  mixture into 10 shot glasses.
  * 10 oz lime juice  Salute!
 ---------------------------------------------------
root@EX2700:/# 
```

###### Permanently enable telnet in the stock firmware

```
root@EX2700:/# config set enable_telnet=1
root@EX2700:/# config set GUI_Region=English
root@EX2700:/# config commit
```

###### Skip stupid registration page

```
root@EX2700:/# config set have_registered=1
root@EX2700:/# config set http_username=<username>
root@EX2700:/# config set http_passwd=<password>
root@EX2700:/# config commit
```

## Downloading neccessary files

The `wget` command does not support HTTPS, so you'll need to host the
[ex2700.tar.gz](https://github.com/jclehner/ex2700/raw/master/ex2700.tar.gz)
file somewhere else. You could either setup a simple file server on your 
computer, or use online services such as [dropcanvas](http://dropcanvas.com).

Be warned that `wget` crashes if not called with `-T <timeout>`.
```
root@EX2700:/# cd /tmp
root@EX2700:/# alias wget="wget -T 1"
root@EX2700:/tmp# wget -O firmware.bin http://downloads.openwrt.org/snapshots/trunk/ramips/mt7620/openwrt-ramips-mt7620-ex2700-squashfs-sysupgrade.bin
root@EX2700:/tmp# wget -O ex2700.tar.gz http://dropcanvas.com/<YOUR LINK ID>
root@EX2700:/tmp# tar -xzf ex2700.tar.gz
root@EX2700:/tmp# cd ex2700
```

## Unlocking the bootloader

**!! WARNING !! Modifying the bootloader environment is dangerous, and has
the potential of bricking<sup>[1](#fn1)</sup> your device! Proceed with extreme caution!**

First, we'll make a backup of the bootloader environment partition (be
careful not to overwrite `/tmp/ex2700/u-boot-env.bin` when using a different name for
the backup file).

```
root@EX2700:/tmp/ex2700# dd if=/dev/mtd1 of=/tmp/u-boot-env.bak
128+0 records in
128+0 records out
```

Then, update the bootloader environment:

```
root@EX2700:/tmp/ex2700# insmod mtd-rw.ko i_want_a_brick=1
root@EX2700:/tmp/ex2700# mtd write u-boot-env.bin u-boot-env
Unlocking u-boot-env ...
Writing from u-boot-env.bin to u-boot-env ... [w]
```

Optional: you can use `./fw_printenv` and `./fw_setenv` to adjust
the bootloader environment.

## Flashing OpenWrt

Flashing OpenWrt is the easy part:

```
root@EX2700:/tmp/ex2700# cd /tmp
root@EX2700:/tmp/# mtd write firmware.bin firmware
Unlocking firmware ...
Writing from firmware.bin to firmware ... [w]
```

If everything is ok, we can reboot. Keep your fingers crossed!

```
root@EX2700:/tmp/# reboot
```
------------------------------------------------------------
<a name="fn1">1</a>: In most cases, you'll be able to unbrick your device using
a serial console.

<a name="fn2">2</a>: It is entirely possible to create an image that can be flashed
via the router's stock web interface, and which passes the bootloader's integrity
checks, but the kernel in that case is constrained to 982976 bytes (960 kilobytes, 
minus 64 bytes), because the bootloader (u-boot) expects a uImage header in the last
64 bytes of the stock "kernel" mtd partition.
