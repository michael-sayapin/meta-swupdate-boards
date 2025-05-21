# meta-swupdate-boards

Please see the corresponding sections below for details.

Dependencies
============

This layer depends on:

* URI: git://git.openembedded.org/bitbake
  * branch: master
  * revision: HEAD

* URI: git://git.openembedded.org/openembedded-core
  * branch: master
  * revision: HEAD

or

* URI: git://git.yoctoproject.org/poky
  * branch: master
  * revision: HEAD

and

* URI: git://github.com/sbabic/meta-swupdate.git
  * branch: master
  * revision: HEAD

For usage with Raspberry Pi boards additional layer is required:

* URI: git://github.com/agherzan/meta-raspberrypi.git
  * branch: master
  * revision: HEAD

For usage with tegra boards, see usage instructions below

Usage
-----

The layer contains examples on how to use the SWUpdate project. Examples
on how to deploy a "dual-copy" update strategy with some common boards
(Beaglebone Black, Raspberry Pi, Sama5d27-som1-ek-sd and Wandboard) are
provided.

Setup your environment accordingly and update `MACHINE` to desired target.

To build simply run:

	bitbake update-image

Above will generate a `swu` file suitable for usage with SWUpdate on
your device.

Note that `update-image` depends on `ext4.gz` and you must make sure
that it is part of `IMAGE_FSTYPES`.

For usage with Raspberry Pi one must add the following to `local.conf`

	RPI_USE_U_BOOT = "1"

Above will enable U-boot which Raspberry Pi does not default to, and
instead boots straight to Linux. U-boot is required to do the "swapping"
of partitions in the "dual-copy" layout.

Usage for Nvidia Jetson (Tegra) Platforms
------

The simplest way to use on tegra is to add to a forked copy of
[tegra-demo-distro](https://github.com/OE4T/tegra-demo-distro) which
also contains relevant images and submodules used to build tegra images.
However, this is not strictly required.  The only dependency required
is the [meta-tegra](https://github.com/OE4T/meta-tegra) layer.

Use these instructions to add to tegra-demo-distro on whatever
branch you'd like to target (kirkstone and later are supported).

```
cd repos
git submodule add https://github.com/sbabic/meta-swupdate-boards.git
git submodule add https://github.com/sbabic/meta-swupdate
cd ../layers
ln -s ../repos meta-swupdate-boards
ln -s ../repos meta-swupdate
```

Then use the `setup-env` script to start a bitbake shell, and add
these layers to your build
```
bitbake-layers add-layer ../layers/meta-swupdate
bitbake-layers add-layer ../layers/meta-swupdate-boards
```

In your local.conf (or distro/machine configuration) add
these lines:
```
IMAGE_INSTALL:append = " swupdate"
USE_REDUNDANT_FLASH_LAYOUT = "1"
SWUPDATE_CORE_IMAGE_NAME = "demo-image-base"
IMAGE_FSTYPES:append = " tar.gz"
```
substituting any image in the "SWUPDATE_CORE_IMAGE_NAME" variable.

Finally, build the update image using:
```
bitbake update-image-tegra
```

After deploying the resulting .swu file to target, run the
update with:
```
swupdate -i </path/to/fie>
```
Then reboot to apply the update.  The root partition should change
and the `nvbootctrl dump-slots-info` output should show boot
from the alternate boot slot with `Capsule update status:1`.

Maintainer
----------

Stefano Babic <sbabic@denx.de>

Submitting patches
------------------

You can submit your patches or post questions regarding
this layer to the swupdate Mailing List:

	swupdate@googlegroups.com

When creating patches, please use something like:

    git format-patch -s --subject-prefix='meta-swupdate-boards][PATCH' <revision range>

Please use 'git send-email' to send the generated patches to the ML
to bypass changes from your mailer.
