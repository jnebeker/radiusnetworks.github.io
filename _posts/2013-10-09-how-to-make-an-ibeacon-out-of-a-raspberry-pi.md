---
author: James Nebeker and David G. Young
layout: post
title: How to Make an iBeacon Out of a Raspberry Pi
---

Want a standalone iBeacon to use for development?  Here's a procedure you can use to build your own for under $100.  If you don't want to go to the trouble, we can ship you a pre-built [development kit](http://developer.radiusnetworks.com/ibeacon/ibeacon-development-kit.html) including everything below (except the cell phone charger -- you probably have one already!)

<img style="margin:10px; height: 200px; border: thin solid #333;float:right;" src='/img/pibeacon.jpg'>

If you want to build one yourself, you'll need basic Linux command line skills.  In the instructions, commands you type on the Raspberry Pi look like the block below.  Lines starting with $ are things you type (you do not actually type the $) and other lines are responses from the computer.

```
$ echo hello
hello
```

If you ever get asked for a password after entering a command, always enter raspberry in response.

###iBeacon parts list:

1. [A Raspberry Pi computer board](http://www.raspberrypi.org/) (Model A or Model B)
2. A 4GB or larger [SD card](http://www.amazon.com/SanDisk-Class-Flash-Memory-SDSDB-008G-AFFP/dp/B007JRB0TC)
3. [A Bluetooth LE dongle.](http://www.amazon.com/dp/B007GFX0PY/ref=pe_385040_30332190_pe_175190_21431760_M3T1_ST1_dp_1)
4. A standard miro-USB 5V cell phone charger

###Tools you will need:

1. A USB Keyboard
2. A USB Mouse
3. A computer monitor with HDMI input
4. An HDMI cable
5. A network cable for internet access
6. An iOS/Android device with AirLocate/iBeaconLocate, or some other tool you can use to see iBeacons

###Procedure

####Step 1.  Set up your Raspberry Pi

The Raspberry Pi is a nifty little computer board that is great for hobbyists.  In order to make it into an iBeacon you'll have to go through its basic setup, hooking it up to a keyboard, mouse and monitor, and plugging it into your router for internet access.  The biggest part of the setup involves preparing the SD card with its operating system.  To do this, follow [this guide](http://www.raspberrypi.org/wp-content/uploads/2012/04/quick-start-guide-v2_1.pdf) and select the "raspbian" operating system option during install.

####Step 2. Verify your connections to your Pi

Unplug the mouse, and plug in the keyboard, Bluetooth LE dongle, monitor, and ethernet cable.  The other end of the ethernet cable must be connecte to a router that can give you internet access.

####Step 3. Log in to your Raspberry Pi

Using the keyboard, log in using the credentials, username: pi, password: raspberry

####Step 4. Verify you have internet access:

```
$ ping www.google.com
PING www.google.com (74.125.228.116): 56 data bytes
64 bytes from 74.125.228.116: icmp_seq=0 ttl=55 time=34.267 ms
64 bytes from 74.125.228.116: icmp_seq=1 ttl=55 time=30.908 ms
```

If you see output like above, your internet access is working.  Hit Ctrl-c to stop the ping.

#### Step 5: Install Build Packages

```
$ sudo apt-get install libusb-dev libdbus-1-dev libglib2.0-dev libudev-dev libical-dev libreadline-dev
```

####Step 6: Download and Uncompress BlueZ

This is the the official Bluetooth stack for Linux and the 5.x series has introduced Bluetooth LE support.

```
$ sudo wget www.kernel.org/pub/linux/bluetooth/bluez-5.8.tar.xz
$ sudo unxz bluez-5.8.tar.xz
$ sudo tar xvf bluez-5.8.tar
$ cd bluez-5.8
```

#### Step 7: Configure and Make BlueZ

Note: the second command below will take the better part of an hour.  Better find something to do!

```
$ sudo ./configure --disable-systemd
$ sudo make
$ sudo make install
```

#### Step 8: Configure Bluetooth Dongle

With your Bluetooth dongle plugged in, running the following command will give detail about the device:

```
$ hciconfig
```

The output should look something like this:

```
hci0:	Type: BR/EDR  Bus: USB
         BD Address: 00:01:0A:39:D4:06  ACL MTU: 1021:8  SCO MTU: 64:1
         DOWN
         RX bytes:1000 acl:0 sco:0 events:47 errors:0
         TX bytes:1072 acl:0 sco:0 commands:47 errors:0
```

This indicates the device is in a down state.  Issue the following command to bring it up:

```
$ sudo hciconfig hci0 up
```

Now it should look like:

```
$ hciconfig
hci0:	Type: BR/EDR  Bus: USB
         BD Address: 00:02:72:3F:4D:60  ACL MTU: 1021:8  SCO MTU: 64:1
         UP RUNNING
         RX bytes:1000 acl:0 sco:0 events:47 errors:0
         TX bytes:1072 acl:0 sco:0 commands:47 errors:0
```

Next, execute the following example command to configure the advertising data to be sent by the iBeacon:

```
$ sudo hcitool -i hci0 cmd 0x08 0x0008 1e 02 01 1a 1a ff 4c 00 02 15 e2 c5 6d b5 df fb 48 d2 b0 60 d0 f5 a7 10 96 e0 00 00 00 00 c5 00 00 00 00 00 00 00 00 00 00 00 00 00
```

The setting in this example corresponds to an iBeacon broadcasting Profile UUID E2C56DB5-DFFB-48D2-B060-D0F5A71096E0 with a major of 0 and a minor of 0.

#### Step 9: Enable Advertising

Use the following command to activate advertising on the dongle, this will allow the device to be detected and recognized as an iBeacon:

```
sudo hciconfig hci0 leadv 0
```

Now take out your cell phone and verify you can see the iBeacon advertisements using AirLocate or iBeaconLocate.

You can then disable advertising using the following command, and see them stop:

```
$ sudo hciconfig hci0 noleadv
```

### Putting it all together (optional)

The steps above show you how you can log in to your Raspberry Pi and make it start advertising like an iBeacon by entering a series of commands.
Next, you can optionally combine
these commands into a script and a config file so that you can power on your Raspberry Pi without a keyboard or monitor and it will start
working as an iBeacon as soon as it boots up.  To do this, you will need to use a text editor to create three files and edit a fourth.

It's not really realistic to type this all in manually at a keyboard connected to the Raspberry Pi.  Instead, you should use a ssh client to connect
remotely to the Raspberry Pi over your network.   In order to do this, though, you'll need to know how to use an ssh client, and you'll need to be able to
figure out the ip address of your Raspberry Pi so you can log into it.  If you are connected to the keyboard, you can find the ip address by typing:

```
$ sudo ifconfig
eth0      Link encap:Ethernet  HWaddr 01:00:22:38:c6:44
          inet addr:192.168.1.80  Bcast:192.168.1.255  Mask:255.255.255.0
```

In the response above, the Raspberry Pi is on IP address 192.168.1.80.  Whatever answer you get, use IP address to connect to your PI from your
workstation using an ssh client.  Then you can copy and paste the contents of the files below to get them on your Raspberry Pi.

In the commands below, we use the
vi editor, but you can use whatever text editor you want.  If using vi, you will need to type the i key to go into insert mode, paste in the contents of the file, then type the esc key followed by ZZ to save the file.


#### Step 10: Create an ibeacon.conf file


```
$ cd
$ vi ibeacon.conf
```

And paste in the contents of this block:

```
# All values must be in hex form separated by spaces between every two hex digits
export BLUETOOTH_DEVICE=hci0
export UUID="e2 c5 6d b5 df fb 48 d2 b0 60 d0 f5 a7 10 96 e0"
export MAJOR="00 00"
export MINOR="00 00"
export POWER="c9"
```

#### Step 11: Create the start script

```$ vi start```

And paste in the contents of this block:

```
#!/bin/sh
. ./ibeacon.conf
echo "Launching virtual iBeacon..."
sudo hciconfig $BLUETOOTH_DEVICE up
sudo hciconfig $BLUETOOTH_DEVICE noleadv
sudo hcitool -i hci0 cmd 0x08 0x0008 1e 02 01 1a 1a ff 4c 00 02 15 $UUID $MAJOR $MINOR $POWER 00 00 00 00 00 00 00 00 00 00 00 00 00
sudo hciconfig $BLUETOOTH_DEVICE leadv 0
echo "Complete"
```

#### Step 12: Create the stop script

```$ vi stop```

And paste in the contents of this block:

```
#!/bin/sh
. ./ibeacon.conf
echo "Disabling virtual iBeacon..."
sudo hciconfig $BLUETOOTH_DEVICE noleadv
echo "Complete"
```

#### Step 13: Make your start and stop scripts executable

```
chmod 777 start
chmod 777 stop
```

#### Step 14: Make the start script execute automatically at boot

```$ sudo vi /etc/init.d/ibeacon```

And paste in the contents of this block:

```
#!/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:$PATH

DESC="iBeacon Application Software"
PIDFILE=/var/run/ibeacon.pid
SCRIPTNAME=/etc/init.d/ibeacon

case "$1" in
start)
        printf "%-50s" "Starting ibeacon..."
        cd /home/pi
        ./start
;;
stop)
        printf "%-50s" "Stopping ibeacon..."
        cd /home/pi
        ./stop
;;
restart)
        $0 stop
        $0 start
;;
*)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
esac

```

Finally, type:

```
$ sudo chmod 777 /etc/init.d/ibeacon
$ sudo update-rc.d ibeacon defaults
```

### That's It!

With these files in place, you can edit your iBeacon's identifiers by manipulating the ibeacon.conf file, and start and stop it with the ./start and ./stop commands.

Now you have a fully functional iBeacon.  Unplug the ethernet cable, unplug the keyboard, and cycle power to your Raspberry Pi.  After about 60 seconds of boot time, you should see it start advertising automatically.

