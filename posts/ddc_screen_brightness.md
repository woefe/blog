---
title: Control brightness of external monitors on Linux
date: 2018-10-09
---

Until now I only knew the convenient feature from my laptop, where I can simply press a hotkey to adjust the brightness of the built in screen.
But what about external monitors?
Can I adjust the brightness without fiddling with the on-screen menu?

<!--more-->

Turns out I can, and the secret sauce I need is called [Display Data Channel (DDC)](https://en.wikipedia.org/wiki/Display_Data_Channel).


# The hardware

Whether DDC works properly depends on many different variables like the monitor, display connection, GPU,...
Here is the list of hardware that worked for me:

1. Desktop:
    * GPU: Nvidia GTX 1070 (with proprietary driver)
    * Monitor 1: Dell U2715H
    * Monitor 2: Dell U2414H
    * Both monitors are connected via DisplayPort (HDMI did not work)
2. Laptop:
    * Dell XPS 13 9370
    * Anker USB-C to HDMI Adapter (Model A8306)
    * No-name USB-C hub with HDMI (couldn't find a modelnumber; Uses a VIA chip according to `dmesg`)
    * Monitor 1: Dell U2715H
    * Monitor 2: Dell U2414H
    * All monitors connected via HDMI

Of course, DDC has to be enabled in the on-screen menu of the monitor ("Others" -> "DDC/CI" on my Dells).


# The software

The tool you need is [ddcutil](https://www.ddcutil.com/).
It is available in the Ubuntu repositories and on Arch Linux via the AUR.
On Arch Linux I had to make a few adjustments to get it working:

1. Load the `i2c-dev` kernel module with `modprobe i2c-dev`.
    To make this change persistent across reboots:
    ```
    echo i2c_dev >> /etc/modules-load.d/ddc.conf
    ```
2. Create group `i2c`:
    ```
    groupadd i2c
    ```
3. Create udev rules to allow group `i2c` read and write access to `/dev/i2c-*`:
    ```
    cp /usr/share/ddcutil/data/45-ddcutil-i2c.rules /etc/udev/rules.d/
    ```
3. Add your user to the `i2c` group:
    ```
    usermod $USER -aG i2c
    ```

After a reboot `ddcutil detect` should detect the monitors. If it does not, check the [documentation](https://www.ddcutil.com/config/).
Some devices need special tweaking, e.g. the [Raspberry Pi](https://www.ddcutil.com/raspberry/).
One more tip: The output of `ddcutil environment` suggests that it is not necessary to load `i2c-dev` when using proprietary Nvidia drivers.
However, without `i2c-dev` (and with [special Nvidia driver settings](ddcutil environment)) ddcutil could not detect any monitors.


# Examples

Settings like color corrections, orientation of the on-screen menu and the input source can be adjusted through DDC.
`ddcutil capabilities` shows a list of supported options.
The following examples adjust and read the brightness of the monitor:

* Get the brightness of the first display: `ddcutil getvcp 10 --display 1`
* Set the brightness of the second display to 100%: `ddcutil setvcp 10 100 --display 2`
* Decrease the brightness of the second display by 5%: `ddcutil setvcp 10 - 5 --display 2`
* Increase the brightness of the first display by 5%: `ddcutil setvcp 10 + 5 --display 1`

I mapped [these](https://github.com/woefe/dotfiles/blob/a8882e631eb84992d93610168bf4ec3d3f005556/i3/.config/i3/config#L178) commands to some hotkeys in my i3 config.
