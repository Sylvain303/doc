# Build ARM image

This tutorial's goal is to build a plug-and-play image for YunoHost for ARM boards.

It could be used on many ARM board (Rasberry Pi, Olimex, Cubieboard…).

This tutorial is based on [Yunocubian](https://github.com/M5oul/Yunocubian).

### Download minimal Debian Jessie
Download a Debian Jessie image compatible with the hardware **without desktop environnement** installed:

* [ARMbian](http://www.armbian.com/download/) (Olimex, Cubieboard, Banana Pi…)
* [Raspbian Jessie Lite](https://www.raspberrypi.org/downloads/raspbian/)

### Copy image and install YunoHost
<a class="btn btn-lg btn-default" href="/copy_image">Copy image to the SD card</a>

You must have enough disk space on the flashcard, ensure you have extended the ext4 filesystem.

You forget to expand the filesystem? There's an option inside raspi-config

~~~
sudo raspi-config
~~~

~~~
│    1 Expand Filesystem                         Ensures that all of the SD card storage is a         │
~~~

Here the systèm will reboot when you quit the menu.

<a class="btn btn-lg btn-default" href="/plug_and_boot">Plug & boot</a>

* Connect via [SSH](ssh): **root@exemple.tld/ip_address** with the password which you could find on respectives documentations. (raspbian default user/pass: pi/raspberry)
* You should be **root** for next operations.
* fix locale warning if any, it looks like:
~~~
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
[…]
~~~

As root:
~~~
sed -i -e '/fr_FR.UTF-8/ s/#//' -e '/en_GB.UTF-8/ s/#//' -e '/en_US.UTF-8/ s/#//' /etc/locale.gen
locale-gen
~~~

<a class="btn btn-lg btn-default" href="/install_manually">Install YunoHost</a>

<div class="alert alert-danger">Do not proceed to **post-installation**.</div>

### Clean image
* Update image:
```bash
apt-get update && apt-get dist-upgrade && apt-get autoremove
```
* Change hostname:
```bash
echo YunoHost > /etc/hostname
hostname -F /etc/hostname
```
* Set new SSH key generation at first lauching:

```bash
# Delete SSH keys
rm -f /etc/ssh/ssh_host_*

# Add script to regenerate SSH keys at first boot
nano /etc/init.d/ssh_gen_host_keys
---
#!/bin/sh
### BEGIN INIT INFO
# Provides:          Generates new ssh host keys on first boot
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:
# Short-Description: Generates new ssh host keys on first boot
# Description:       Generatesapt-get --purge clean new ssh host keys on $
### END INIT INFO
ssh-keygen -f /etc/ssh/ssh_host_rsa_key -t rsa -N ""
ssh-keygen -f /etc/ssh/ssh_host_dsa_key -t dsa -N ""
insserv -r /etc/init.d/ssh_gen_host_keys
rm -f \$0
---

# Give executable right
chmod a+x /etc/init.d/ssh_gen_host_keys

# Make it execute at next boot
insserv /etc/init.d/ssh_gen_host_keys
```

* Delete logs:
```bash
find /var/log -type f -exec rm {} \;
```

* [optional] If you want to publish your image for others, put back qwerty keyboard layout if you changed for something else:
~~~bash
# to be completed
~~~

* Turn off your board:
```bash
shutdown
```

### Copy image
Plug your SD card on your desktop computer and copy it:
<div class="alert alert-danger">Be carefull to not erase your data.</div>

```bash
sudo dd bs=1M if=/dev/sdd of=~/yunohost-jessie-board-year-month-day.img
```

### Verify image
<a class="btn btn-lg btn-default" href="/copy_image">Copy image to the SD card</a>

<a class="btn btn-lg btn-default" href="/plug_and_boot">Plug & boot</a>

<a class="btn btn-lg btn-default" href="/postinstall">Post-install</a>

<div class="alert alert-info">If everything is alright, you could publish your image.</div>

### Publish image
* Shrink image size by resizing image read from the SD card.
 * an example of script is here: http://sirlagz.net/2013/03/10/script-automatic-rpi-image-downsizer/
* Reduce size by zipping the image:
```bash
zip yunohost-jessie-board-year-month-day.img.zip yunohost-jessie-board-year-month-day.img
```

* Publish: you could post your image on the [forum](https://forum.yunohost.org/).
