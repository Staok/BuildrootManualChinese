### 6.1.3. Build an external toolchain with Buildroot

The Buildroot internal toolchain option can be used to create an external toolchain. Here are a series of steps to build an internal toolchain and package it up for reuse by Buildroot itself (or other projects).

Create a new Buildroot configuration, with the following details:

- Select the appropriate ***\*Target options\**** for your target CPU architecture
- In the ***\*Toolchain\**** menu, keep the default of ***\*Buildroot toolchain\**** for ***\*Toolchain type\****, and configure your toolchain as desired
- In the ***\*System configuration\**** menu, select ***\*None\**** as the ***\*Init system\**** and ***\*none\**** as ***\*/bin/sh\****
- In the ***\*Target packages\**** menu, disable ***\*BusyBox\****
- In the ***\*Filesystem images\**** menu, disable ***\*tar the root filesystem\****

Then, we can trigger the build, and also ask Buildroot to generate a SDK. This will conveniently generate for us a tarball which contains our toolchain:

```
make sdk
```

This produces the SDK tarball in `$(O)/images`, with a name similar to `arm-buildroot-linux-uclibcgnueabi_sdk-buildroot.tar.gz`. Save this tarball, as it is now the toolchain that you can re-use as an external toolchain in other Buildroot projects.

In those other Buildroot projects, in the ***\*Toolchain\**** menu:

- Set ***\*Toolchain type\**** to ***\*External toolchain\****
- Set ***\*Toolchain\**** to ***\*Custom toolchain\****
- Set ***\*Toolchain origin\**** to ***\*Toolchain to be downloaded and installed\****
- Set ***\*Toolchain URL\**** to `file:///path/to/your/sdk/tarball.tar.gz`

#### External toolchain wrapper

When using an external toolchain, Buildroot generates a wrapper program, that transparently passes the appropriate options (according to the configuration) to the external toolchain programs. In case you need to debug this wrapper to check exactly what arguments are passed, you can set the environment variable `BR2_DEBUG_WRAPPER` to either one of:

- `0`, empty or not set: no debug
- `1`: trace all arguments on a single line
- `2`: trace one argument per line

## 6.2. /dev management

On a Linux system, the `/dev` directory contains special files, called *device files*, that allow userspace applications to access the hardware devices managed by the Linux kernel. Without these *device files*, your userspace applications would not be able to use the hardware devices, even if they are properly recognized by the Linux kernel.

Under `System configuration`, `/dev management`, Buildroot offers four different solutions to handle the `/dev` directory :

- The first solution is ***\*Static using device table\****. This is the old classical way of handling device files in Linux. With this method, the device files are persistently stored in the root filesystem (i.e. they persist across reboots), and there is nothing that will automatically create and remove those device files when hardware devices are added or removed from the system. Buildroot therefore creates a standard set of device files using a *device table*, the default one being stored in `system/device_table_dev.txt` in the Buildroot source code. This file is processed when Buildroot generates the final root filesystem image, and the *device files* are therefore not visible in the `output/target` directory. The `BR2_ROOTFS_STATIC_DEVICE_TABLE` option allows to change the default device table used by Buildroot, or to add an additional device table, so that additional *device files* are created by Buildroot during the build. So, if you use this method, and a *device file* is missing in your system, you can for example create a `board/<yourcompany>/<yourproject>/device_table_dev.txt` file that contains the description of your additional *device files*, and then you can set `BR2_ROOTFS_STATIC_DEVICE_TABLE` to `system/device_table_dev.txt board/<yourcompany>/<yourproject>/device_table_dev.txt`. For more details about the format of the device table file, see [Chapter 25, *Makedev syntax documentation*](https://buildroot.org/downloads/manual/manual.html#makedev-syntax).
- The second solution is ***\*Dynamic using devtmpfs only\****. *devtmpfs* is a virtual filesystem inside the Linux kernel that has been introduced in kernel 2.6.32 (if you use an older kernel, it is not possible to use this option). When mounted in `/dev`, this virtual filesystem will automatically make *device files* appear and disappear as hardware devices are added and removed from the system. This filesystem is not persistent across reboots: it is filled dynamically by the kernel. Using *devtmpfs* requires the following kernel configuration options to be enabled: `CONFIG_DEVTMPFS` and `CONFIG_DEVTMPFS_MOUNT`. When Buildroot is in charge of building the Linux kernel for your embedded device, it makes sure that those two options are enabled. However, if you build your Linux kernel outside of Buildroot, then it is your responsibility to enable those two options (if you fail to do so, your Buildroot system will not boot).
- The third solution is ***\*Dynamic using devtmpfs + mdev\****. This method also relies on the *devtmpfs* virtual filesystem detailed above (so the requirement to have `CONFIG_DEVTMPFS` and `CONFIG_DEVTMPFS_MOUNT` enabled in the kernel configuration still apply), but adds the `mdev` userspace utility on top of it. `mdev` is a program part of BusyBox that the kernel will call every time a device is added or removed. Thanks to the `/etc/mdev.conf` configuration file, `mdev` can be configured to for example, set specific permissions or ownership on a device file, call a script or application whenever a device appears or disappear, etc. Basically, it allows *userspace* to react on device addition and removal events. `mdev` can for example be used to automatically load kernel modules when devices appear on the system. `mdev` is also important if you have devices that require a firmware, as it will be responsible for pushing the firmware contents to the kernel. `mdev` is a lightweight implementation (with fewer features) of `udev`. For more details about `mdev` and the syntax of its configuration file, see http://git.busybox.net/busybox/tree/docs/mdev.txt.
- The fourth solution is ***\*Dynamic using devtmpfs + eudev\****. This method also relies on the *devtmpfs* virtual filesystem detailed above, but adds the `eudev` userspace daemon on top of it. `eudev` is a daemon that runs in the background, and gets called by the kernel when a device gets added or removed from the system. It is a more heavyweight solution than `mdev`, but provides higher flexibility. `eudev` is a standalone version of `udev`, the original userspace daemon used in most desktop Linux distributions, which is now part of Systemd. For more details, see http://en.wikipedia.org/wiki/Udev.

The Buildroot developers recommendation is to start with the ***\*Dynamic using devtmpfs only\**** solution, until you have the need for userspace to be notified when devices are added/removed, or if firmwares are needed, in which case ***\*Dynamic using devtmpfs + mdev\**** is usually a good solution.

Note that if `systemd` is chosen as init system, /dev management will be performed by the `udev` program provided by `systemd`.

## 6.3. init system

The *init* program is the first userspace program started by the kernel (it carries the PID number 1), and is responsible for starting the userspace services and programs (for example: web server, graphical applications, other network servers, etc.).

Buildroot allows to use three different types of init systems, which can be chosen from `System configuration`, `Init system`:

- The first solution is ***\*BusyBox\****. Amongst many programs, BusyBox has an implementation of a basic `init` program, which is sufficient for most embedded systems. Enabling the `BR2_INIT_BUSYBOX` will ensure BusyBox will build and install its `init` program. This is the default solution in Buildroot. The BusyBox `init` program will read the `/etc/inittab` file at boot to know what to do. The syntax of this file can be found in http://git.busybox.net/busybox/tree/examples/inittab (note that BusyBox `inittab` syntax is special: do not use a random `inittab` documentation from the Internet to learn about BusyBox `inittab`). The default `inittab` in Buildroot is stored in `package/busybox/inittab`. Apart from mounting a few important filesystems, the main job the default inittab does is to start the `/etc/init.d/rcS` shell script, and start a `getty` program (which provides a login prompt).
- The second solution is ***\*systemV\****. This solution uses the old traditional *sysvinit* program, packed in Buildroot in `package/sysvinit`. This was the solution used in most desktop Linux distributions, until they switched to more recent alternatives such as Upstart or Systemd. `sysvinit` also works with an `inittab` file (which has a slightly different syntax than the one from BusyBox). The default `inittab` installed with this init solution is located in `package/sysvinit/inittab`.
- The third solution is ***\*systemd\****. `systemd` is the new generation init system for Linux. It does far more than traditional *init* programs: aggressive parallelization capabilities, uses socket and D-Bus activation for starting services, offers on-demand starting of daemons, keeps track of processes using Linux control groups, supports snapshotting and restoring of the system state, etc. `systemd` will be useful on relatively complex embedded systems, for example the ones requiring D-Bus and services communicating between each other. It is worth noting that `systemd` brings a fairly big number of large dependencies: `dbus`, `udev` and more. For more details about `systemd`, see http://www.freedesktop.org/wiki/Software/systemd.

The solution recommended by Buildroot developers is to use the ***\*BusyBox init\**** as it is sufficient for most embedded systems. ***\*systemd\**** can be used for more complex situations.

------

[[3\] ](https://buildroot.org/downloads/manual/manual.html#idm381)This terminology differs from what is used by GNU configure, where the host is the machine on which the application will run (which is usually the same as target)