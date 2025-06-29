## 9.3. Storing the Buildroot configuration

The Buildroot configuration can be stored using the command `make savedefconfig`.

This strips the Buildroot configuration down by removing configuration options that are at their default value. The result is stored in a file called `defconfig`. If you want to save it in another place, change the `BR2_DEFCONFIG` option in the Buildroot configuration itself, or call make with `make savedefconfig BR2_DEFCONFIG=<path-to-defconfig>`.

The recommended place to store this defconfig is `configs/<boardname>_defconfig`. If you follow this recommendation, the configuration will be listed in `make list-defconfigs` and can be set again by running `make <boardname>_defconfig`.

Alternatively, you can copy the file to any other place and rebuild with `make defconfig BR2_DEFCONFIG=<path-to-defconfig-file>`.

## 9.4. Storing the configuration of other components

The configuration files for BusyBox, the Linux kernel, Barebox, U-Boot and uClibc should be stored as well if changed. For each of these components, a Buildroot configuration option exists to point to an input configuration file, e.g. `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE`. To store their configuration, set these configuration options to a path where you want to save the configuration files, and then use the helper targets described below to actually store the configuration.

As explained in [Section 9.1, “Recommended directory structure”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure), the recommended path to store these configuration files is `board/<company>/<boardname>/foo.config`.

Make sure that you create a configuration file *before* changing the `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE` etc. options. Otherwise, Buildroot will try to access this config file, which doesn’t exist yet, and will fail. You can create the configuration file by running `make linux-menuconfig` etc.

Buildroot provides a few helper targets to make the saving of configuration files easier.

- `make linux-update-defconfig` saves the linux configuration to the path specified by `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE`. It simplifies the config file by removing default values. However, this only works with kernels starting from 2.6.33. For earlier kernels, use `make linux-update-config`.
- `make busybox-update-config` saves the busybox configuration to the path specified by `BR2_PACKAGE_BUSYBOX_CONFIG`.
- `make uclibc-update-config` saves the uClibc configuration to the path specified by `BR2_UCLIBC_CONFIG`.
- `make barebox-update-defconfig` saves the barebox configuration to the path specified by `BR2_TARGET_BAREBOX_CUSTOM_CONFIG_FILE`.
- `make uboot-update-defconfig` saves the U-Boot configuration to the path specified by `BR2_TARGET_UBOOT_CUSTOM_CONFIG_FILE`.
- For at91bootstrap3, no helper exists so you have to copy the config file manually to `BR2_TARGET_AT91BOOTSTRAP3_CUSTOM_CONFIG_FILE`.

