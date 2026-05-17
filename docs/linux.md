---
id: linux
title: Linux Considerations
sidebar_label: Linux Considerations
---

# Permissions issues

To get Via to connect to your keyboard successfully, you would usually need to set up a `udev` rule. 
Without this in place, the error below may appear when authorizing a keyboard in Via. 

>  NotAllowedError: Failed to open the device.
> 
>  Device: crkbd
>  Vid: 0x4653
>  Pid: 0x0001


The reason for this is that standard linux user accounts are not permitted to access to the `hidraw` device that VIA uses to communicate with your keyboard by default, so you have to create a rule to allow it.

## `udev` rules
Rules are found in the folder `/etc/udev/rules.d` and are applied in order of filename.
For [reasons](https://github.com/systemd/systemd/issues/39056), rules allowing access to `hidraw` should be numbered 51-59, and examples below use the file name `/etc/udev/rules.d/55-via.rules`

### 



```
# This rule is filtered on the usb id's of a Corne (crkbd) and allows access to the users group.
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="4653", ATTRS{idProduct}=="0001", MODE="0660", GROUP="users", TAG+="uaccess"
```

> **--NOTE--**: **Always use lower case for the VID and PID values** 
> ID's will appear capitalized in some contexts but should always be lower case within udev rules. 
> e.g. chrome://usb-internals may show `0xCA75`. 
> use `ATTRS{idVendor}=="ca75"` within udev rules.

### Allow Everything
*Just make it work!*

#### Add a rule Manually 
Create a text file, and save it to `/etc/udev/rules.d/55-via.rules`
```
# This rule allows access to *any* hidraw device for members of the users group.
SUBSYSTEM=="hidraw", MODE="0660", GROUP="users", TAG+="uaccess"
```
The above assumes you're in the `users` group. Some distributions don't have a group by this name, so you may have to replace this with something else. See [what groups am i in?](#what-groups-am-i-in)

#### Copy and paste this one-liner command
```
export MY_PGID=`id -g`; sudo --preserve-env=MY_PGID sh -c 'echo "SUBSYSTEM==\"hidraw\", MODE=\"0660\", GROUP=\"$MY_PGID\", TAG+=\"uaccess\"" > /etc/udev/rules.d/55-via.rules && udevadm control --reload && udevadm trigger'
```
This will 
 - Check your primary group
 - Create /etc/udev/rules.d/55-via.rules to allow that group
 - Reload and re-evaluate udev rules

### Find a keyboards USB identifiers
USB devices have a Vendor and Product ID. 
Conventionally, all devices made by one company (the vendor!) would have the same **Vendor ID**. Each model then has a **Product ID**. The combination of the two are combined to uniquely identify a keyboard type.

Here are a few ways to find these ID's:
- Use your browsers usb-internals page 
  browse to  `chrome://usb-internals/`, click "Devices" at the top of the page and find your keyboard in the list
- run `lsusb`, and find the device in the list. 
  `Bus 003 Device 004: ID 4653:0001 foostan crkbd`
  `                        VID:PID               `
- Via may show you the Vendor ID (VID) and Product ID (PID) in error messages.

### What groups an I in?
Just enter `groups` or `id -Gn` in your terminal emulator.
```
[tim@tim-pc ~]$ id -Gn
tim wheel 
```