## CEC introduction

Quoting from the Wikipedia page of the [CEC](https://en.wikipedia.org/wiki/Consumer_Electronics_Control):

> Consumer Electronics Control (CEC) is a feature of HDMI designed to allow users to command and control devices connected through HDMI by using only one remote control. For example by using the remote control of a television set to control a set-top box and or DVD player. Up to 15 devices can be controlled. CEC also allows for individual CEC-enabled devices to command and control each other without user intervention.

UDOO X86’s HDMI 1.4b supports CEC so we decided to develop the driver for the CEC of our board by adopting the new Linux standard **CEC Framework**.

## CEC Framework
The Linux Kernel includes, since the 4.8 version, a standard HDMI-CEC framework to provide a unified kernel interface for use with HDMI-CEC hardware.  

Quoting from the [official CEC framework documentation](https://www.kernel.org/doc/html/v4.11/media/kapi/cec-core.html#):
> The framework also gives the option to decide what to do in the kernel driver and what should be handled by userspace applications. In addition it integrates the remote control passthrough feature into the kernel’s remote control framework.
This new HDMI-CEC framework just acts like an API that allows manufacturers to easily write 'device drivers' that can talk to all different type of CEC hardware. The framework exposes CEC as a remote control input device allowing to any generic hardware to have standard syscalls to user-space application.

With the new Linux kernels, most of the future hardware as well as the current ones, will be supported directly.
This CEC Framework sets itself as a standard that will be adopted by all manufacturers to support CEC in their hardware and should be adopted by media applications soon.  

Here you can find [details on the framework](http://events.linuxfoundation.org/sites/events/files/slides/cec-wide_1.pdf) by the developer Hans Verkuil from Cisco.


### State of Art - libCEC
[libCEC](http://libcec.pulse-eight.com/) is a well established user-space library that media applications needed to use to let CEC work with their media application.
Lots of graphical media applications, like Kodi, still uses the libCEC platform in its own code to interact with CEC protocol.  
Now that a standard HDMI-CEC Framework is integrated into Linux Kernel, future hardware will adopt the standard, and will do away with the need to use LibCEC in Linux. Pulse-Eight, the Developer, could add the support for this new HDMI-CEC framework inside libCEC, as if it was just another CEC device to get best of both.

libCEC issue on github about adding the CEC Framework support - [GitHub link](https://github.com/Pulse-Eight/libcec/issues/67).  
Kodi thread about using the CEC Framework alternatively to libCEC - [KODI Forum thread](http://forum.kodi.tv/showthread.php?tid=293256).

## Compile and mount the seco-cec driver for UDOO X86

### Compile

<span class="label label-warning">Heads up!</span> Make sure you have Linux
**kernel headers >= 4.14** (if you want to compile with headers >= 4.10 you
need to revert this [commit][commit414]) and make sure you have installed an
updated FW version (**>= 1.04**) on your UDOO X86 (check this in BIOS Main Page).

[commit414]: https://github.com/ektor5/secocec/commit/3874d5ef2139b982878aac9b3d18ad2db1ce47e1

Install dependencies (make, git, ecc..)

    sudo apt install git build-essential linux-headers-`uname -r`

Download the source code from git

    git clone https://github.com/ektor5/secocec.git

Enter into the dir just downloaded:

    cd secocec

compile the driver:

    make

The module should appear as file called `seco-cec.ko`.

### Mount

This module depends on the Linux CEC Framework Module, and it needs to be loaded first.

    modprobe cec

*Note:* this module conflicts with the official SMBus driver `i2c-i801` and therefore it needs to be blacklisted. On Ubuntu should be already blacklisted, if you're using

    echo "blacklist i2c-i801" > /etc/modprobe.d/i2c-i801.conf

Load the module:

    insmod seco-cec.ko

Done, the module is now loaded and working!

## How to use the driver with file system applications
The CEC Framework expose all the CEC functionalities of the CEC protocol through a file system device `/dev/cec0` so the applications can access directly this file.
Since the CEC Framework is quite new there are still few applications that exploits all functions provided by the Framework.

To test the device from userspace (`/dev/cec0`), there are some programs ready to use in *v4l-utils* tools package (available to install via apt):

* **cec-ctl**: An application to control cec devices
* **cec-compliance**: An application to verify remote CEC devices
* **cec-follower**: An application to emulate CEC followers  

You can check the official [man pages](https://www.mankier.com/package/v4l-utils) for info.

Install v4l-utils

    apt install v4l-utils

Following you can find some examples.

Register the CEC device (UDOO X86) in the CEC network using the physical and logical address:
```
cec-ctl --phys-addr=2.0.0.0 \
        --playback
```

Power on tv

    cec-ctl --active-source=phys-addr=2.0.0.0 --image-view-on -t0

Power off tv

    cec-ctl --standby -t0

Enable TV remote (it depends on manufacturers protocol)
E.g. SimpLink by LG.
```
cec-ctl --phys-addr=2.0.0.0 \
        --playback \
        --custom-command=cmd=0x89,payload=0x02:0x05 \
        --deck-status=deck-info=0x01 \
        --menu-status=menu-state=0x00 \
        --custom-command=cmd=0x89,payload=0x05:0x01 \
        --report-power-status=pwr-state=0 -t0
```
With this command the TV remote keys are interpreted by the OS as a standard keyboard and you can use it to interact with the OS applications like Kodi for examples.

### Edit keymaps
Sometimes, not all the remote keys are set like we want it to be.  
E.g: the *OK* button is not understood as *Enter* by the X server input module (Xinput), so it is necessary to remap it.  

To edit the keymap do the following:

Install ir-keytable package:

    apt install ir-keytable

Get the current keytable:

    ir-keytable -r > seco-cec-keytable.conf

Edit the file with a text editor (e.g. vim):

    vim seco-cec-keytable.conf

Rewrite it to the driver

    ir-keytable -w seco-cec-keytable.conf

Done!

## Milestones

The driver is still in development to add features and bugfixing.  
The next steps will be: add the CEC notifier support, add the SMBus communication via i2c-i801 module.

The driver will be published mainline, once the driver will be insert in the mainline kernel, compile and mount the secocec driver manually won't be needed. A patch is already under review.

Regarding the CEC Framework state, you can see [this status update](https://hverkuil.home.xs4all.nl/cec-status.txt) from Hans Verkuil, the developer of the Framework.
The CEC Framework is still in the process to be implemented for more different types of hardware. The 4.14 kernel will be a big step in that regard and when the list of supported hardware will be complete, the Framework will really becomes interesting for graphical media applications.
