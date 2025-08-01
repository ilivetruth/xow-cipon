## About

## This is a fork of the original project, to use the older drivers (from 2015), which are required to make this work with the Cipon 3rd party adapter

xow is a Linux user mode driver for the Xbox One wireless dongle.
It communicates with the dongle via `libusb` and provides joystick input through the `uinput` kernel module.
The input mapping is based on existing kernel drivers like [xpad](https://github.com/paroj/xpad).

## Releases

### Linux distributions

[![Packaging status](https://repology.org/badge/vertical-allrepos/xow.svg)](https://repology.org/project/xow/versions)

- [RPM packaging instructions](https://gitlab.com/yajoman/xow-rpm) (for distributions like Fedora)

### Third-party hardware

- EmuELEC (starting with [version 3.3](https://github.com/EmuELEC/EmuELEC/releases/tag/v3.3))
- Steam Link (starting with [build 747](https://steamcommunity.com/app/353380/discussions/0/1735510154204276395))

Feel free to package xow for any Linux distribution or hardware you like.
Any issues regarding the packaging should be reported to the respective maintainers.

## Installation

### Prerequisites

- Linux (kernel 4.5 or newer)
- curl (for proprietary driver download)
- cabextract (for firmware extraction)
- libusb (libusb-1.0-0-dev for Debian)
- systemd (version 232 or newer)

### Guide

1. Clone the repository:

```
git clone https://github.com/medusalix/xow
```

2. Build and install xow:

```
cd xow
make BUILD=RELEASE
sudo make install
```

**NOTE:** Please use `BUILD=DEBUG` when asked for your debug logs.

3. Download the firmware for the wireless dongle:

```
sudo xow-get-firmware.sh
```

**NOTE:** The `--skip-disclaimer` flag might be useful for scripting purposes.

4. Enable and start the `systemd` service (runs xow at boot time):

```
sudo systemctl enable xow
sudo systemctl start xow
```

**NOTE:** Running xow manually is **strongly discouraged**. A reboot might be required for xow to work correctly.

### Updating

Make sure to completely uninstall xow before updating:

```
sudo systemctl stop xow
sudo systemctl disable xow
sudo make uninstall
```

## Interoperability

You can enable the dongle's pairing mode by sending the `SIGUSR1` signal to xow:

```
sudo systemctl kill -s SIGUSR1 xow
```

**NOTE:** Signals are only handled *after* a dongle has been plugged in. The default behavior for `SIGUSR1` is to terminate the process.

## Troubleshooting

### Error messages

- `InputException`: No such file or directory
    - The `/dev/uinput` device has to be available. The `uinput` kernel module needs to be loaded.
- `InputException`: Permission denied
    - The permissions for `/dev/uinput` have to allow read and write access. The `udev` rules need to be installed and any conflicts with existing rules have to be resolved.
- `Mt76Exception`: Failed to load firmware
    - Another driver might have already loaded the dongle's firmware. The dongle needs to be unplugged to reset its internal memory, followed by a restart of xow's `systemd` service.
- `LIBUSB_ERROR_TIMEOUT`
    - See the [USB incompatibilities](#usb-incompatibilities) section.
- `LIBUSB_ERROR_BUSY` or `LIBUSB_ERROR_NO_DEVICE`
    - Only a single program can communicate with the dongle at once. Any existing drivers that might interfere with xow need to be disabled. This includes running multiple instances of xow.
- `LIBUSB_ERROR_ACCESS`
    - The permissions for the dongle's USB device have to be set correctly. This is also handled by the `udev` rules.

Using an outdated version of `libusb` can cause various issues. Make sure to update `libusb` to the latest version.

### Pairing problems

The controller only remembers the *last* device it was connected to. It will not automatically establish a connection to the dongle if it was previously plugged into a USB port or paired via bluetooth, even if the same computer was used.

### Configuration issues

- Certain games do not detect wireless controllers
    - Enable the *compatibility mode* in the service configuration, reload the `systemd` daemon and restart the service.
    Controllers connected to the dongle will appear as Xbox 360 controllers.
- Buttons/triggers/sticks are mapped incorrectly
    - Try the options listed on [this page](https://wiki.archlinux.org/index.php/Gamepad#Setting_up_deadzones_and_calibration) to remap your inputs.
- Input from the sticks is jumping around
    - Try the options listed on [this page](https://wiki.archlinux.org/index.php/Gamepad#Setting_up_deadzones_and_calibration) to set your deadzones.

### USB incompatibilities

Some USB controllers are known to cause issues with xow. Plugging your dongle into a USB port that uses an `ASMedia` controller will lead to problems. Most `Intel` USB controllers work well with xow.
Power management issues can arise when using a USB 3 controller. These can lead to timeouts of the USB communication. The use of a USB hub can mitigate these problems.

### Other problems

In case of any other problems, please open an issue with all the relevant details (dongle version, controller version, logs, captures, etc.).

**NOTE:** Please refrain from creating issues concerning input remapping, deadzones or game compatibility as these topics are outside the scope of this project.

## How it works

The dongle's wireless chip (MT76xx) handles the WLAN connection with individual controllers.
The packet format follows Microsoft's undisclosed GIP (Game Input Protocol) specification.
Most of the reverse engineering was done by capturing the communication between the dongle and a Windows PC using [`Wireshark`](https://www.wireshark.org).
As no datasheets for this chip are publicly available, I have used datasheets of similar wireless radios for assistance.
Special thanks to the authors of OpenWrt's [`mt76`](https://github.com/openwrt/mt76) kernel driver.
It would have been impossible for me to create this driver without `mt76`'s source code.
If anyone has a greater understanding of the GIP or the weird quirks I had to add to make the driver work, please contact me.

## License

xow is released under the [GNU General Public License, Version 2](LICENSE).

```
Copyright (C) 2019 Medusalix

This program is free software; you can redistribute it and/or
modify it under the terms of the GNU General Public License
as published by the Free Software Foundation; either version 2
of the License, or (at your option) any later version.
```
