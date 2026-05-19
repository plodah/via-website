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

## Creating rules

### Allow Everything

This rule will ***Just make it work*** in many cases. This gives the same results as [doing it maunally](#do-it-manually), so check there for more info.

If **allowing everything** sounds scary, you may prefer to take a more precise approach and only [allow specific devices](#allow-specific-devices).

Run the following in your terminal emulator:

```
export MY_PGID=`id -g`; sudo --preserve-env=MY_PGID sh -c 'echo "SUBSYSTEM==\"hidraw\", MODE=\"0660\", GROUP=\"$MY_PGID\", TAG+=\"uaccess\"" > /etc/udev/rules.d/55-via.rules && udevadm control --reload && udevadm trigger'
```

This specifically requires `sudo` to be installed and available to you. It will create a rule allowing your users primary group access to all `hidraw` devices.
> Running as root won't work, as it will pick up `root`'s group instead of yours. 

#### Do it manually

If you’d rather do it manually, or the above has failed, create a text file, and save it to `/etc/udev/rules.d/55-via.rules` with contents

```
# This rule allows access to *any* hidraw device for members of the users group.
SUBSYSTEM=="hidraw", MODE="0660", GROUP="users", TAG+="uaccess"
```

This assumes you're in the `users` group. Some distributions don't have a group by this name, so you may have to replace this with something else. See [what groups am i in?](#what-groups-am-i-in)

Once you’ve created the rule, reload udev and trigger it to apply rules.
```
sudo udevadm control --reload
sudo udevadm trigger
```

> A reboot would also work to reload & apply rules!

### Allow Specific devices

The main flaw in allowing everything is that it’s wider than it has to be. It’s relatively straightforward to add conditions to `udev` rules which limit their scope. 

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

If you’re really into keyboards of a particular brand, you could also be able to omit the `ATTRS{idProduct}=="0101"` condition and have a rule that applies the Vendor ID.

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
- Via may show you the Vendor ID (VID) and Product ID (PID) in error messages.
- run `lsusb`, and find the device in the list. <br />
```
[tim@tim-pc ~]$ lsusb

Bus 003 Device 004: ID 4653:0001 foostan crkbd
                         ^   ^
                        VID:PID              
```

### What groups an I in?

Just enter `groups` or `id -Gn` in your terminal emulator.

```
[tim@tim-pc ~]$ id -Gn
tim wheel 
```

### Why `55-via.rules`?

When adding rules outside of a package, as required with Via, you would place them in `/etc/udev/rules.d`. These rules are combined with packaged rules (perhaps found in `/usr/lib/udev/rules.d`) and are all applied in lexicographical order.

Advice on where to position Via udev rules will vary by who you ask, what distribution you're talking about, and when you ask. 

Examples here use `55-via.rules` because it's after `50-udev-default.rules` but before `70-uaccess.rules` and `73-seat-late.rules` 

Rules created for Via should tag the devices as `uaccess` which will be picked up by the latter.

### What about my distro?
The information here applies to many popular distributions, including Fedora, Debian (and therefore Ubuntu), Arch and many others. 

There will be exceptions. 

The information here might still help you to compose rules but documentation specific to your distribution would be a better source of information on what to do with them.

### Further Reading
[udev - Arch Linux Wiki](https://wiki.archlinux.org/title/Udev)
