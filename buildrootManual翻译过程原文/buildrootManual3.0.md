## Chapter 7. Configuration of other components

Before attempting to modify any of the components below, make sure you have already configured Buildroot itself, and have enabled the corresponding package.

- BusyBox

  If you already have a BusyBox configuration file, you can directly specify this file in the Buildroot configuration, using `BR2_PACKAGE_BUSYBOX_CONFIG`. Otherwise, Buildroot will start from a default BusyBox configuration file.To make subsequent changes to the configuration, use `make busybox-menuconfig` to open the BusyBox configuration editor.It is also possible to specify a BusyBox configuration file through an environment variable, although this is not recommended. Refer to [Section 8.6, “Environment variables”](https://buildroot.org/downloads/manual/manual.html#env-vars) for more details.

- uClibc

  Configuration of uClibc is done in the same way as for BusyBox. The configuration variable to specify an existing configuration file is `BR2_UCLIBC_CONFIG`. The command to make subsequent changes is `make uclibc-menuconfig`.

- Linux kernel

  If you already have a kernel configuration file, you can directly specify this file in the Buildroot configuration, using `BR2_LINUX_KERNEL_USE_CUSTOM_CONFIG`.If you do not yet have a kernel configuration file, you can either start by specifying a defconfig in the Buildroot configuration, using `BR2_LINUX_KERNEL_USE_DEFCONFIG`, or start by creating an empty file and specifying it as custom configuration file, using `BR2_LINUX_KERNEL_USE_CUSTOM_CONFIG`.To make subsequent changes to the configuration, use `make linux-menuconfig` to open the Linux configuration editor.

- Barebox

  Configuration of Barebox is done in the same way as for the Linux kernel. The corresponding configuration variables are `BR2_TARGET_BAREBOX_USE_CUSTOM_CONFIG` and `BR2_TARGET_BAREBOX_USE_DEFCONFIG`. To open the configuration editor, use `make barebox-menuconfig`.

- U-Boot

  Configuration of U-Boot (version 2015.04 or newer) is done in the same way as for the Linux kernel. The corresponding configuration variables are `BR2_TARGET_UBOOT_USE_CUSTOM_CONFIG` and `BR2_TARGET_UBOOT_USE_DEFCONFIG`. To open the configuration editor, use `make uboot-menuconfig`.

## Chapter 8. General Buildroot usage

## 8.1. *make* tips

This is a collection of tips that help you make the most of Buildroot.

**Display all commands executed by make:** 

```
 $ make V=1 <target>
```



**Display the list of boards with a defconfig:** 

```
 $ make list-defconfigs
```



**Display all available targets:** 

```
 $ make help
```



Not all targets are always available, some settings in the `.config` file may hide some targets:

- `busybox-menuconfig` only works when `busybox` is enabled;
- `linux-menuconfig` and `linux-savedefconfig` only work when `linux` is enabled;
- `uclibc-menuconfig` is only available when the uClibc C library is selected in the internal toolchain backend;
- `barebox-menuconfig` and `barebox-savedefconfig` only work when the `barebox` bootloader is enabled.
- `uboot-menuconfig` and `uboot-savedefconfig` only work when the `U-Boot` bootloader is enabled and the `uboot` build system is set to `Kconfig`.

**Cleaning:** Explicit cleaning is required when any of the architecture or toolchain configuration options are changed.

To delete all build products (including build directories, host, staging and target trees, the images and the toolchain):

```
 $ make clean
```

**Generating the manual:** The present manual sources are located in the *docs/manual* directory. To generate the manual:

```
 $ make manual-clean
 $ make manual
```

The manual outputs will be generated in *output/docs/manual*.

**Notes**

- A few tools are required to build the documentation (see: [Section 2.2, “Optional packages”](https://buildroot.org/downloads/manual/manual.html#requirement-optional)).

**Resetting Buildroot for a new target:** To delete all build products as well as the configuration:

```
 $ make distclean
```

**Notes.** If `ccache` is enabled, running `make clean` or `distclean` does not empty the compiler cache used by Buildroot. To delete it, refer to [Section 8.13.3, “Using `ccache` in Buildroot”](https://buildroot.org/downloads/manual/manual.html#ccache).

**Dumping the internal make variables:** One can dump the variables known to make, along with their values:

```
 $ make -s printvars VARS='VARIABLE1 VARIABLE2'
 VARIABLE1=value_of_variable
 VARIABLE2=value_of_variable
```

It is possible to tweak the output using some variables:

- `VARS` will limit the listing to variables which names match the specified make-patterns - this must be set else nothing is printed
- `QUOTED_VARS`, if set to `YES`, will single-quote the value
- `RAW_VARS`, if set to `YES`, will print the unexpanded value

For example:

```
 $ make -s printvars VARS=BUSYBOX_%DEPENDENCIES
 BUSYBOX_DEPENDENCIES=skeleton toolchain
 BUSYBOX_FINAL_ALL_DEPENDENCIES=skeleton toolchain
 BUSYBOX_FINAL_DEPENDENCIES=skeleton toolchain
 BUSYBOX_FINAL_PATCH_DEPENDENCIES=
 BUSYBOX_RDEPENDENCIES=ncurses util-linux
 $ make -s printvars VARS=BUSYBOX_%DEPENDENCIES QUOTED_VARS=YES
 BUSYBOX_DEPENDENCIES='skeleton toolchain'
 BUSYBOX_FINAL_ALL_DEPENDENCIES='skeleton toolchain'
 BUSYBOX_FINAL_DEPENDENCIES='skeleton toolchain'
 BUSYBOX_FINAL_PATCH_DEPENDENCIES=''
 BUSYBOX_RDEPENDENCIES='ncurses util-linux'
 $ make -s printvars VARS=BUSYBOX_%DEPENDENCIES RAW_VARS=YES
 BUSYBOX_DEPENDENCIES=skeleton toolchain
 BUSYBOX_FINAL_ALL_DEPENDENCIES=$(sort $(BUSYBOX_FINAL_DEPENDENCIES) $(BUSYBOX_FINAL_PATCH_DEPENDENCIES))
 BUSYBOX_FINAL_DEPENDENCIES=$(sort $(BUSYBOX_DEPENDENCIES))
 BUSYBOX_FINAL_PATCH_DEPENDENCIES=$(sort $(BUSYBOX_PATCH_DEPENDENCIES))
 BUSYBOX_RDEPENDENCIES=ncurses util-linux
```

The output of quoted variables can be reused in shell scripts, for example:

```
 $ eval $(make -s printvars VARS=BUSYBOX_DEPENDENCIES QUOTED_VARS=YES)
 $ echo $BUSYBOX_DEPENDENCIES
 skeleton toolchain
```
