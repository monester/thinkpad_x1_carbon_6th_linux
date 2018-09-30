# Linux configuration guide for Thinkpad x1 Carbon 6th Gen (2018)

This guide workable for Lenovo ThinkPad Ultrabook X1 Carbon Gen6 (20KH006HRT) and Ubuntu 18.04.
After these steps, laptop works around 8-10 hours on medium load.

If you follow this guide, no one is responsible for any damage to your hardware or any other kind of harming your machine.

# Results

As a result of these actions I get power statistic like this:
![power_statistic](power_statistic.png)

## Install Ubuntu 18.04

If touchpad andr/or trackpoint don't work on Ubuntu installer, add 'psmouse.synaptics_intertouch=1' to loader.

## Install latest version of Linux Kernel

Firstly, after instalation, update Linux Kernel (because in kernels 4.17.x power management has improved) using UKUU:
```
$ sudo add-apt-repository ppa:teejee2008/ppa
$ sudo apt-get update
$ sudo apt-get install ukuu
$ sudo ukuu --check
$ sudo ukuu --install-latest
```

## Touchpad & Trackpoint

1. Edit the /etc/modprobe.d/blacklist.conf file and comment out following line:

```
# blacklist i2c_i801
```

2. Edit the /etc/default/grub file and change line:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```
to
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash psmouse.synaptics_intertouch=1"
```

3. update grub:
```
$ sudo update-grub
```

4. install xserver-synaptics:

```
$ sudo apt-get install xserver-xorg-input-synaptics
```

5. Copy wakeup-control script from this repo to /lib/systemd/system-sleep/ for wakeup touchpad/trackpoint after sleep.

## Deep sleep

Fixed in Firmware 1.30 https://twitter.com/hdevalence/status/1038305566517415936

## low cTDP and trip temperature in Linux

This problem is related to thermal throttling on Linux, that is set much below the Windows values. This will cause your laptop to run much slower than it could under heavy stress.

Before you attempt to apply this solution, please make sure that the problem still exists when you read it. To do so, open a Linux terminal and run following commands:
```
$ sudo apt-get install msr-tools
$ sudo rdmsr -f 29:24 -d 0x1a2
```
If you see 3 as a result value (or 15 when running on battery), you don’t have to do anything. Otherwise:
1. Disable Secure Boot in the BIOS (won’t work otherwise)
2. Run this command:
```
sudo apt install git virtualenv build-essential python3-dev \
  libdbus-glib-1-dev libgirepository1.0-dev libcairo2-dev
```
3. Install lenovo-throttling-fix:
```
$ cd lenovo-throttling-fix/
$ sudo ./install.sh
```
Check again, that the result from running the rdmsr command is 3

Personally, I use a bit lower temperature levels to preserve battery life in favor of performance. If you want to change the default values, you need to edit the /etc/lenovo_fix file and set the Trip_Temp_C for both battery and AC the way you want:
```
[BATTERY]
# Other options here...
PL2_Tdp_W: 40
Trip_Temp_C: 75

[AC]
# Other options here...
PL1_Tdp_W: 34
PL2_Tdp_W: 40
Trip_Temp_C: 90
```

## CPU undervolting

The amazing Lenovo Throttling fix script supports also the undervolting. To enable it, please edit the /etc/lenovo_fix.conf again and update the [UNDERVOLT] section. In my case, this settings proven to be stable:

```
[UNDERVOLT]
# CPU core voltage offset (mV)
CORE: -110
# Integrated GPU voltage offset (mV)
GPU: -90
# CPU cache voltage offset (mV)
CACHE: -110
# System Agent voltage offset (mV)
UNCORE: -90
# Analog I/O voltage offset (mV)
ANALOGIO: 0
```

## Battery charging thresholds

There are a lot of theories and information about ThinkPad charging thresholds. Some theories say thresholds are needed to keep the battery healthy, some think they are useless and the battery will work the same just as it is. In this article I will try not to settle that argument. 🙂 Instead I try to tell how and why I use them, and then proceed to show how they can be changed in different versions of Windows, should you still want to change these thresholds.

I always stick with following settings for my laptops (and somehow I feel that it works):

- Start threshold: 45%
- Stop threshold: 95%

This means that the charging will start only if the battery level goes down below 45% and will stop at 95%. This prevents battery from being charged too often and from being charged beyond a recommended level.

To achieve this for Linux based machines you need to install some packages by running:

```
$ sudo apt-get install tlp tlp-rdw acpi-call-dkms tp-smapi-dkms acpi-call-dkms
```

After that just edit the /etc/default/tlp file and edit following values:
```
# Uncomment both of them if commented out
START_CHARGE_THRESH_BAT0=45
STOP_CHARGE_THRESH_BAT0=95
```

Reboot, run:
```
sudo tlp-stat | grep tpacpi-bat
```

and check if the values are as you expect:
```
tpacpi-bat.BAT0.startThreshold          = 45 [%]
tpacpi-bat.BAT0.stopThreshold           = 95 [%]
```

You can change these thresholds anytime, and apply changes using command:
```
$ sudo tlp start
```

Note, that if you need to have your laptop fully charged, you can achieve that by running following command while connected to AC:
```
$ tlp fullcharge
```
