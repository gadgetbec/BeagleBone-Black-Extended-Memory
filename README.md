# PimpMyBeagleBoneBlack
How to super setup the BeagleBone Black from a Mac.
Runs from the SD card and uses the rest of the card as extended memory for the file system.

##Step 1: Flash the latest Debian Image##

- Format your SD card with [SDFormatter][1] for Mac
- Don't download the "eMMC flasher" version - this will run from the BBB without an SD card present
- Do download from [beagleboard.org/latest-images][2] the version that says "BeagleBone and BeagleBone Black via microSD card"
- After download, unpack the image with [The Unarchiver][3]
- Download [PiFiller][4] and use it to install the Debian image onto the SD card. 
  - You have to unplug the SD card first and then follow the prompts to point to the unpacked Debian image
- Take the SD card out of the Mac. Make sure the BBB is unplugged from everything (USB, power, and ethernet) and insert the SD card.
- Hold down the Boot button and power up the BBB with the external power adapter.
  - wait 3 to 4 seconds after all of the LEDs come on.
  - this should take 20-45min
- When you think it's done, try to connect to the BBB with ssh using the command:
    ssh -l root@192.168.7.2
- If you receive this error message:

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
86:fc:00:83:cd:eb:78:5b:b5:d5:ab:47:58:25:6a:40.
Please contact your system administrator.
Add correct host key in /Users/gadgetbec/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/gadgetbec/.ssh/known_hosts:4
RSA host key for 192.168.7.2 has changed and you have requested strict checking.
Host key verification failed.

- do the following:
- Go to home directory in host computer and create a config file in the below path:
cd ~/.ssh/ 
nano config
- Type the following:
Host 192.168.7.*
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no 
- Save and exit (Ctrl+X, Y, <Enter>)
- SSH into the device and login as root.
ssh 192.168.7.2 -l root
- Verify the new installation
"root@beaglebone:/sys/class/gpio# cat /etc/dogtag 
Cloud9 GNOME Image 2013.09.04"

###References###
Adafruit Learing System - BeagleBone Black: [Installing Operating Sytems (Mac)][5]
Adafruit Learing System - BeagleBone Black: [Flasing the BeagleBone Black][6]
[My BeagleBone Black Findings][8]

##Step 2: Extend the BeagleBone's memory##
- Type the folowing command:
df -h 
- You should see the total root filesystem size "rootfs", the SD card will show up as /dev/mmcblk0p2

"Filesystem      Size  Used Avail Use% Mounted on
rootfs          3.5G  1.8G  1.6G  54% /
udev             10M     0   10M   0% /dev
tmpfs           100M  644K   99M   1% /run
/dev/mmcblk0p2  3.5G  1.8G  1.6G  54% /
tmpfs           249M     0  249M   0% /dev/shm
tmpfs           249M     0  249M   0% /sys/fs/cgroup
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           100M     0  100M   0% /run/user
/dev/mmcblk1p1   96M   65M   32M  68% /media/BEAGLEBONE
/dev/mmcblk0p1   96M   66M   31M  69% /media/BEAGLEBONE_
/dev/mmcblk1p2  3.4G  1.4G  1.9G  43% /media/rootfs

- Repartition the memory card with the following commands
  fdisk /dev/mmcblk0 
  p
- You will get a window that says: 
"        Device Boot      Start         End      Blocks   Id  System
/dev/mmcblk0p1   *        2048      198655       98304    e  W95 FAT16 (LBA)
/dev/mmcblk0p2          198656     7577599     3689472   83  Linux"
- Enter the following commands in order
d
n
p
2
enter 
enter
w

- will look like this:

"Command (m for help): d
Partition number (1-4): 2

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (1-4, default 2): 2
First sector (198656-31116287, default 198656): 
Using default value 198656
Last sector, +sectors or +size{K,M,G} (198656-31116287, default 31116287): 
Using default value 31116287

Command (m for help): w
The partition table has been altered!"
- You will get a message like this:
"Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks."

- reboot the BeagleBone
reboot
- repartition the memory card into the file system with the command:
resize2fs /dev/mmcblk0p2
- Check to see if it worked
df -h
- This is what a 16GByte SD card looks like:

"root@beaglebone:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
rootfs           15G  1.8G   13G  13% /
udev             10M     0   10M   0% /dev
tmpfs           100M  616K   99M   1% /run
/dev/mmcblk0p2   15G  1.8G   13G  13% /
tmpfs           249M     0  249M   0% /dev/shm
tmpfs           249M     0  249M   0% /sys/fs/cgroup
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           100M     0  100M   0% /run/user
/dev/mmcblk1p1   96M   65M   32M  68% /media/BEAGLEBONE
/dev/mmcblk0p1   96M   66M   31M  69% /media/BEAGLEBONE_
/dev/mmcblk1p2  3.4G  1.4G  1.9G  43% /media/rootfs"

###References###
Noob's Guide: [How to Increase Memory of BeagleBone Black][7]







[1]: https://www.sdcard.org/downloads/formatter_4/eula_mac/
[2]: http://beagleboard.org/latest-images
[3]: https://itunes.apple.com/us/app/the-unarchiver/id425424353?mt=12
[4]: http://ivanx.com/raspberrypi/
[5]: https://learn.adafruit.com/beaglebone-black-installing-operating-systems/mac-os-x
[6]: https://learn.adafruit.com/beaglebone-black-installing-operating-systems/flashing-the-beaglebone-black
[7]: http://noobtechiespeaks.blogspot.com/2014/10/ya-i-know-previous-versions-of.html
[8]: http://mybeagleboneblackfindings.blogspot.com/
