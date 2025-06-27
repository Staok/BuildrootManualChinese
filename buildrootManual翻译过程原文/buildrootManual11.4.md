- `LIBFOO_CPE_ID_*` variables is a set of variables that allows the package to define its [CPE identifier](https://nvd.nist.gov/products/cpe). The available variables are:

  - `LIBFOO_CPE_ID_VALID`, if set to `YES`, specifies that the default values for each of the following variables is appropriate, and generates a valid CPE ID.
  - `LIBFOO_CPE_ID_PREFIX`, specifies the prefix of the CPE identifier, i.e the first three fields. When not defined, the default value is `cpe:2.3:a`.
  - `LIBFOO_CPE_ID_VENDOR`, specifies the vendor part of the CPE identifier. When not defined, the default value is `<pkgname>_project`.
  - `LIBFOO_CPE_ID_PRODUCT`, specifies the product part of the CPE identifier. When not defined, the default value is `<pkgname>`.
  - `LIBFOO_CPE_ID_VERSION`, specifies the version part of the CPE identifier. When not defined the default value is `$(LIBFOO_VERSION)`.
  - `LIBFOO_CPE_ID_UPDATE` specifies the *update* part of the CPE identifier. When not defined the default value is `*`.

  If any of those variables is defined, then the generic package infrastructure assumes the package provides valid CPE information. In this case, the generic package infrastructure will define `LIBFOO_CPE_ID`.

  For a host package, if its `LIBFOO_CPE_ID_*` variables are not defined, it inherits the value of those variables from the corresponding target package.

The recommended way to define these variables is to use the following syntax:

```
LIBFOO_VERSION = 2.32
```

Now, the variables that define what should be performed at the different steps of the build process.

- `LIBFOO_EXTRACT_CMDS` lists the actions to be performed to extract the package. This is generally not needed as tarballs are automatically handled by Buildroot. However, if the package uses a non-standard archive format, such as a ZIP or RAR file, or has a tarball with a non-standard organization, this variable allows to override the package infrastructure default behavior.
- `LIBFOO_CONFIGURE_CMDS` lists the actions to be performed to configure the package before its compilation.
- `LIBFOO_BUILD_CMDS` lists the actions to be performed to compile the package.
- `HOST_LIBFOO_INSTALL_CMDS` lists the actions to be performed to install the package, when the package is a host package. The package must install its files to the directory given by `$(HOST_DIR)`. All files, including development files such as headers should be installed, since other packages might be compiled on top of this package.
- `LIBFOO_INSTALL_TARGET_CMDS` lists the actions to be performed to install the package to the target directory, when the package is a target package. The package must install its files to the directory given by `$(TARGET_DIR)`. Only the files required for *execution* of the package have to be installed. Header files, static libraries and documentation will be removed again when the target filesystem is finalized.
- `LIBFOO_INSTALL_STAGING_CMDS` lists the actions to be performed to install the package to the staging directory, when the package is a target package. The package must install its files to the directory given by `$(STAGING_DIR)`. All development files should be installed, since they might be needed to compile other packages.
- `LIBFOO_INSTALL_IMAGES_CMDS` lists the actions to be performed to install the package to the images directory, when the package is a target package. The package must install its files to the directory given by `$(BINARIES_DIR)`. Only files that are binary images (aka images) that do not belong in the `TARGET_DIR` but are necessary for booting the board should be placed here. For example, a package should utilize this step if it has binaries which would be similar to the kernel image, bootloader or root filesystem images.
- `LIBFOO_INSTALL_INIT_SYSV`, `LIBFOO_INSTALL_INIT_OPENRC` and `LIBFOO_INSTALL_INIT_SYSTEMD` list the actions to install init scripts either for the systemV-like init systems (busybox, sysvinit, etc.), openrc or for the systemd units. These commands will be run only when the relevant init system is installed (i.e. if systemd is selected as the init system in the configuration, only `LIBFOO_INSTALL_INIT_SYSTEMD` will be run). The only exception is when openrc is chosen as init system and `LIBFOO_INSTALL_INIT_OPENRC` has not been set, in such situation `LIBFOO_INSTALL_INIT_SYSV` will be called, since openrc supports sysv init scripts. When systemd is used as the init system, buildroot will automatically enable all services using the `systemctl preset-all` command in the final phase of image building. You can add preset files to prevent a particular unit from being automatically enabled by buildroot.
- `LIBFOO_HELP_CMDS` lists the actions to print the package help, which is included to the main `make help` output. These commands can print anything in any format. This is seldom used, as packages rarely have custom rules. ***\*Do not use this variable\****, unless you really know that you need to print help.
- `LIBFOO_LINUX_CONFIG_FIXUPS` lists the Linux kernel configuration options that are needed to build and use this package, and without which the package is fundamentally broken. This shall be a set of calls to one of the kconfig tweaking option: `KCONFIG_ENABLE_OPT`, `KCONFIG_DISABLE_OPT`, or `KCONFIG_SET_OPT`. This is seldom used, as package usually have no strict requirements on the kernel options.
- `LIBFOO_BUSYBOX_CONFIG_FIXUPS` lists the Busybox configuration options that are needed to use this package especially in some scripts, or at contrario the useless options. This shall be a set of calls to one of the kconfig tweaking option: `KCONFIG_ENABLE_OPT`, `KCONFIG_DISABLE_OPT`, or `KCONFIG_SET_OPT`.

The preferred way to define these variables is:

```
define LIBFOO_CONFIGURE_CMDS
        action 1
        action 2
        action 3
endef
```

In the action definitions, you can use the following variables:

- `$(LIBFOO_PKGDIR)` contains the path to the directory containing the `libfoo.mk` and `Config.in` files. This variable is useful when it is necessary to install a file bundled in Buildroot, like a runtime configuration file, a splashscreen image…
- `$(@D)`, which contains the directory in which the package source code has been uncompressed.
- `$(LIBFOO_DL_DIR)` contains the path to the directory where all the downloads made by Buildroot for `libfoo` are stored in.
- `$(TARGET_CC)`, `$(TARGET_LD)`, etc. to get the target cross-compilation utilities
- `$(TARGET_CROSS)` to get the cross-compilation toolchain prefix
- Of course the `$(HOST_DIR)`, `$(STAGING_DIR)` and `$(TARGET_DIR)` variables to install the packages properly. Those variables point to the global *host*, *staging* and *target* directories, unless *per-package directory* support is used, in which case they point to the current package *host*, *staging* and *target* directories. In both cases, it doesn’t make any difference from the package point of view: it should simply use `HOST_DIR`, `STAGING_DIR` and `TARGET_DIR`. See [Section 8.12, “Top-level parallel build”](https://buildroot.org/downloads/manual/manual.html#top-level-parallel-build) for more details about *per-package directory* support.

Finally, you can also use hooks. See [Section 18.23, “Hooks available in the various build steps”](https://buildroot.org/downloads/manual/manual.html#hooks) for more information.

