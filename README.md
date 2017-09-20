Experimental lx branded zone for openSUSE Leap 42.2

Current status: **it works**

# SmartOS Requirements

* SmartOS: > 20170913T233706Z
* https://us-east.manta.joyent.com/Joyent_Dev/public/SmartOS/smartos.html#20170913T233706Z
* https://smartos.org/bugview/OS-6326

# Requirements

Install kiwi3 from the Virtualization:Appliances repository:

```
sudo zypper ar -f obs://Virtualization:Appliances/openSUSE_Leap_42.2 Virtualization:Appliances"
zypper in -f python3-kiwi
```

# Build instructions

Perform the following command inside of the git repository checkout:

```
sudo kiwi-ng system prepare --description . --root /tmp/lx-zone
```

This will build the root filesystem of the zone.

At the end of the process just run:

```
sudo tar czf opensuse-zone.tar.gz /tmp/lx-zone
```

### SmartOS lx-brand guest tools

This repository contains a small fork of the [upstream guest tools](https://github.com/joyent/sdc-vmtools-lx-brand).

Some changes have been made to make the installation script work with KIWI. This
is needed because KIWI `system prepare` will run the script straight from a chroot
session.

TODO: submit the changes upstream once the lx branded zone works.

# Image and manifest creation

Copy the `opensuse-lx-zone.tar.gz` file to a SmartOS host.

Checkout [this](https://github.com/joyent/debian-lx-brand-image-builder) repository
and then execute:

```
./create-lx-image -t /zones/opensuse-zone.tar.gz -k 3.13.0 -i lx-opensuse-leap-42-2 -d "openSUSE Leap 42.2 64-bit lx-brand image." -u http://opensuse.org -m 20150316T201553Z
```

This will produce the `.zfs` image and its manifest.

The can be imported via:

```
imgadm install -m lx-opensuse-leap-42-2-20170406.json -f lx-opensuse-leap-42-2-20170406.zfs.gz
```

The names of the image and of the manifest are going to change according to your
local time.

# Run the zone

Get the uuid of the image by doing: `imgadm list | grep opensuse`, then
create a `.json` file like this one:

```json
{
  "alias": "lxtest",
  "brand": "lx",
  "kernel_version": "4.3.0",
  "max_physical_memory": 2048,
  "quota": 10,
  "image_uuid": "82e018c4-1ada-11e7-9b00-17458a170ca3"
}
```

Replace `image_uuid` with the actual `uuid` of your image.

Then start the zone by doing:

```
vmadm create -f opensuse.json
```

