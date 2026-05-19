---
id: linux
title: Linux Considerations
sidebar_label: Linux Considerations
---

# Permissions issues

To allow Via to connect to your keyboard successfully from a Linux-based operating system, a `udev` rule would usually be required.

Without a rule in place, you may see the error below after authorizing a keyboard in Via.

> NotAllowedError: Failed to open the device.  
>   
> Device: crkbd   
> Vid: 0x4653 <br />Pid: 0x0001 

## What are `udev` rules?

`udev` is responsible for device management on systemd based Linux-based operating systems. By default, only `root` users can access devices but many software packages will include rules to allow standard users access to devices as appropriate (e.g. in driver packages).

When adding rules outside of a package, as required with Via, you would place them in `/etc/udev/rules.d`. These rules are combined with packaged rules and applied in lexicographical order.

Advice on where to position Via udev rules will vary by speaker, distribution and time. Examples here will use `55-via.rules` which is after `50-udev-default.rules` but before `70-uaccess.rules` and `73-seat-late.rules` 

Rules created for Via should tag the devices as `uaccess` which will be picked up by the latter.

## Example rules

### Allow Everything

This rule will ***Just make it work!*** It allows your user group to access all `rawhid` devices. You may wish to [create a more limited rule](#allow-specific-devices) instead.

Run the following in your terminal emulator:

```
export MY_PGID=`id -g`; sudo --preserve-env=MY_PGID sh -c 'echo "SUBSYSTEM==\"hidraw\", MODE=\"0660\", GROUP=\"$MY_PGID\", TAG+=\"uaccess\"" > /etc/udev/rules.d/55-via.rules && udevadm control --reload && udevadm trigger'
```

This will

- Require `sudo` access
- Find the primary group of your user
- Create `/etc/udev/rules.d/55-via.rules` with a rule that allows your user group access to **all** hidraw devices.
- Reload and re-evaluate udev rules

#### Do it manually

If you’d rather do it manually, or the above has failed, create a text file, and save it to `/etc/udev/rules.d/55-via.rules` and add the content

```
# This rule allows access to *any* hidraw device for members of the users group.
SUBSYSTEM=="hidraw", MODE="0660", GROUP="users", TAG+="uaccess"
```

The above assumes you're in the `users` group. Some distributions don't have a group by this name, so you may have to replace this with something else. See [what groups am i in?](#what-groups-am-i-in)

Once you’ve created the rule

- re-load udev rules with `sudo udevadm control --reload`
- Re-apply udev rules with `sudo udevadm trigger`

A reboot would also achieve the same thing!

### Allow Specific devices

The obvious flaw in the above approach is that it’s broad. It’s relatively straightforward to add conditions to `udev` rules which limit their scope.

You’ll need to know:

- Your keyboards VID and PID  ([find them](#what-groups-am-i-in))
- What group you want this to apply to ([what groups am i in?](#what-groups-am-i-in)) 

Then use this info to put a rule together.

```
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="4653", ATTRS{idProduct}=="0001", MODE="0660", GROUP="users", TAG+="uaccess"
```

Replace `ATTRS{idVendor}=="4653"` with your keyboards Vendor ID (VID)  
Replace `ATTRS{idProduct}=="0001"` with your keyboards Product ID (PID)  
Replace `GROUP=”users”` with a group you’re in, if necessary.

Then create the file `/etc/udev/rules.d/55-via.rules` with your rule.

> **--NOTE--**: **Always use lower case for the VID and PID values** ID's will appear capitalized in some contexts but should always be lower case within udev rules. e.g. chrome://usb-internals may show `0xCA75`. use `ATTRS{idVendor}=="ca75"` within udev rules.

If you have more than one keyboard (of different models), you’ll have to create a rule for each. They can be in the same file, just put them on separate lines.

If you’re really really into one brands keyboards, you may also be able to omit the `ATTRS{idProduct}=="0101"` condition and have a rule that applies just to one Vendor ID.

```
# Allows all keyboards with a VID of 3434, commonly used by Keychron.
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="3434", MODE="0660", GROUP="users", TAG+="uaccess"
```

## Appendices

### Find USB ID’s

USB devices have a Vendor and Product ID.
Conventionally, all devices made by one company (the vendor!) would have the same **Vendor ID**. Each model then has a **Product ID**. The combination of the two are combined to uniquely identify a keyboard type.

Here are a few ways to find these ID's:

- Use your browsers usb-internals page  
  browse to  `chrome://usb-internals/`, click "Devices" at the top of the page and find your keyboard in the list
- run `lsusb`, and find the device in the list.  
  `Bus 003 Device 004: ID 4653:0001 foostan crkbd                       VID:PID              `
- Via may show you the Vendor ID (VID) and Product ID (PID) in error messages.

### What groups an I in?

Just enter `groups` or `id -Gn` in your terminal emulator.

```
[tim@tim-pc ~]$ id -Gn
tim wheel 
```
