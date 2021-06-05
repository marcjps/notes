# Linux Hardware Configuration


### Sound volume

Increase volume

	amixer -q sset Master 10%+

Decrease volume

	amixer -q sset Master 10%-

Mute/unmute

	amixer -q sset Master toggle

Alternatives:

	pacmd list-sinks
	pactl set-sink-mute 0 0
	pactl set-sink-volume 1 65%


### Screen brightness

The easiest way is with the command "light".

Increase brightness by 5%:

	light -A 5

Decrease brightness by 5%:

	light -U 5

Output current brightness level:

	light -G

An alternative way is edit these brightness files and set a value.

	/sys/class/backlight/radeon_b10/brightness 
	/sys/class/backlight/intel_backlight

### USB drives

Note that the location /media is for mounting local removable drives, while /mnt is for mounting network locations.

usb-mount.sh

	#!/bin/bash
	echo Mounting /dev/sb1 to /media/usb
	mount -t vfat /dev/sdb1 /media/usb

usb-umount.sh

	#!/bin/bash
	echo Unmounting /media/usb
	umount /media/usb


### Bluetooth devices

List bluetooth adapters.
	
	sudo hciconfig

Adapter on/off

	sudo hciconfig hci0 up
	sudo hciconfig hci0 down

Scan for bluetooth devices in range

	sudo hcitool scan

Scan for Bluetooth Low Energy (BLE) devices

	sudo hcitool lescan

Pair and connect

	bluetoothctl
	scan on
	pair FC:58:FA:A4:A7:B1
	connect FC:58:FA:A4:A7:B1

Use a connected bluetooth audio device

	pacmd list-sinks
	pactl set-sink-mute 2 0
	pactl set-sink-volume 2 100%
	pactl set-sink-volume 1 0%

Forget device:

	remove FC:58:FA:A4:A7:B1


### CPU frequency

View CPU frequency info:

	cpupower frequency-info

Limit the maximum CPU frequency:
	
	cpupower frequency-set -u 933MHz

Disable/Enable a core (use 0 or 1)

	sudo sh -c "echo '0' > /sys/devices/system/cpu/cpu3/online"


### Power button action

* edit /etc/systemd/logind.conf and change the power button action
* restart the systemd-logind service

### Touchpad (libinput) driver config

Find the touchpad configuration file at: /etc/X11/xorg.conf.d/70-synaptics.conf

```text
Section "InputClass"
    Identifier "SynPS/2 Synaptics TouchPad"
    Driver "synaptics"
    MatchIsTouchpad "on"

    Option "TapButton1" "1"
    Option "TapButton2" "3"
    Option "TapButton3" "2"
    Option "MinSpeed" "1.3"

    Option "AccelerationProfile" "0"
    Option "Tapping" "on"
	Option "AccelSpeed" "0.47"

EndSection
```

You can test out the options on the command line.  E.g.

```text
xinput set-prop device option-number setting
```

See also: https://jlk.fjfi.cvut.cz/arch/manpages/man/xorg.conf.5

### libinput-gestures

This will give you three finger swipe gestures when using libinput.  (e.g for backward and forward in browser)

Follow the steps here to install it: https://github.com/bulletmark/libinput-gestures


### BIOS updates

```text
sudo service fwupd start
sudo fwupdmgr refresh
sudo fwupdmgr update
```

### Changing the boot and virtual console fonts for 4k displays

https://martin.rpdev.net/2017/01/21/setting-console-font-size-on-hidpi-screens-in-fedora.html


### Switch to nvidia dGPU

```text
cp -p /usr/share/X11/xorg.conf.d/nvidia.conf /etc/X11/xorg.conf.d/nvidia.conf
```

Edit the file to use: Option "PrimaryGPU" "yes" 

### Suspend on lid-close

Edit /etc/systemd/logind.conf

Uncomment:

```text
HandleLidSwitch=suspend
```


