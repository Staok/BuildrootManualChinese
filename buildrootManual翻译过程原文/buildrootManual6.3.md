### 9.8.2. Providing extra hashes

Buildroot bundles a [list of hashes](https://buildroot.org/downloads/manual/manual.html#adding-packages-hash) against which it checks the integrity of the downloaded archives, or of those it generates locally from VCS checkouts. However, it can only do so for the known versions; for packages where it is possible to specify a custom version (e.g. a custom version string, a remote tarball URL, or a VCS repository location and changeset), Buildroot can’t carry hashes for those.

For users concerned with the integrity of such downloads, it is possible to provide a list of hashes that Buildroot can use to check arbitrary downloaded files. Those extra hashes are looked up similarly to the extra patches (above); for each directory in `BR2_GLOBAL_PATCH_DIR`, the first file to exist is used to check a package download:

- `<global-patch-dir>/<packagename>/<packageversion>/<packagename>.hash`
- `<global-patch-dir>/<packagename>/<packagename>.hash`

The `utils/add-custom-hashes` script can be used to generate these files.

## 9.9. Adding project-specific packages

In general, any new package should be added directly in the `package` directory and submitted to the Buildroot upstream project. How to add packages to Buildroot in general is explained in full detail in [Chapter 18, *Adding new packages to Buildroot*](https://buildroot.org/downloads/manual/manual.html#adding-packages) and will not be repeated here. However, your project may need some proprietary packages that cannot be upstreamed. This section will explain how you can keep such project-specific packages in a project-specific directory.

As shown in [Section 9.1, “Recommended directory structure”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure), the recommended location for project-specific packages is `package/<company>/`. If you are using the br2-external tree feature (see [Section 9.2, “Keeping customizations outside of Buildroot”](https://buildroot.org/downloads/manual/manual.html#outside-br-custom)) the recommended location is to put them in a sub-directory named `package/` in your br2-external tree.

However, Buildroot will not be aware of the packages in this location, unless we perform some additional steps. As explained in [Chapter 18, *Adding new packages to Buildroot*](https://buildroot.org/downloads/manual/manual.html#adding-packages), a package in Buildroot basically consists of two files: a `.mk` file (describing how to build the package) and a `Config.in` file (describing the configuration options for this package).

Buildroot will automatically include the `.mk` files in first-level subdirectories of the `package` directory (using the pattern `package/*/*.mk`). If we want Buildroot to include `.mk` files from deeper subdirectories (like `package/<company>/package1/`) then we simply have to add a `.mk` file in a first-level subdirectory that includes these additional `.mk` files. Therefore, create a file `package/<company>/<company>.mk` with following contents (assuming you have only one extra directory level below `package/<company>/`):

```
include $(sort $(wildcard package/<company>/*/*.mk))
```

For the `Config.in` files, create a file `package/<company>/Config.in` that includes the `Config.in` files of all your packages. An exhaustive list has to be provided since wildcards are not supported in the source command of kconfig. For example:

```
source "package/<company>/package1/Config.in"
source "package/<company>/package2/Config.in"
```

Include this new file `package/<company>/Config.in` from `package/Config.in`, preferably in a company-specific menu to make merges with future Buildroot versions easier.

If using a br2-external tree, refer to [Section 9.2, “Keeping customizations outside of Buildroot”](https://buildroot.org/downloads/manual/manual.html#outside-br-custom) for how to fill in those files.

## 9.10. Quick guide to storing your project-specific customizations

Earlier in this chapter, the different methods for making project-specific customizations have been described. This section will now summarize all this by providing step-by-step instructions to storing your project-specific customizations. Clearly, the steps that are not relevant to your project can be skipped.

1. `make menuconfig` to configure toolchain, packages and kernel.
2. `make linux-menuconfig` to update the kernel config, similar for other configuration like busybox, uclibc, …
3. `mkdir -p board/<manufacturer>/<boardname>`
4. Set the following options to `board/<manufacturer>/<boardname>/<package>.config` (as far as they are relevant):
   - `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE`
   - `BR2_PACKAGE_BUSYBOX_CONFIG`
   - `BR2_UCLIBC_CONFIG`
   - `BR2_TARGET_AT91BOOTSTRAP3_CUSTOM_CONFIG_FILE`
   - `BR2_TARGET_BAREBOX_CUSTOM_CONFIG_FILE`
   - `BR2_TARGET_UBOOT_CUSTOM_CONFIG_FILE`
5. Write the configuration files:
   - `make linux-update-defconfig`
   - `make busybox-update-config`
   - `make uclibc-update-config`
   - `cp <output>/build/at91bootstrap3-*/.config board/<manufacturer>/<boardname>/at91bootstrap3.config`
   - `make barebox-update-defconfig`
   - `make uboot-update-defconfig`
6. Create `board/<manufacturer>/<boardname>/rootfs-overlay/` and fill it with additional files you need on your rootfs, e.g. `board/<manufacturer>/<boardname>/rootfs-overlay/etc/inittab`. Set `BR2_ROOTFS_OVERLAY` to `board/<manufacturer>/<boardname>/rootfs-overlay`.
7. Create a post-build script `board/<manufacturer>/<boardname>/post_build.sh`. Set `BR2_ROOTFS_POST_BUILD_SCRIPT` to `board/<manufacturer>/<boardname>/post_build.sh`
8. If additional setuid permissions have to be set or device nodes have to be created, create `board/<manufacturer>/<boardname>/device_table.txt` and add that path to `BR2_ROOTFS_DEVICE_TABLE`.
9. If additional user accounts have to be created, create `board/<manufacturer>/<boardname>/users_table.txt` and add that path to `BR2_ROOTFS_USERS_TABLES`.
10. To add custom patches to certain packages, set `BR2_GLOBAL_PATCH_DIR` to `board/<manufacturer>/<boardname>/patches/` and add your patches for each package in a subdirectory named after the package. Each patch should be called `<packagename>-<num>-<description>.patch`.
11. Specifically for the Linux kernel, there also exists the option `BR2_LINUX_KERNEL_PATCH` with as main advantage that it can also download patches from a URL. If you do not need this, `BR2_GLOBAL_PATCH_DIR` is preferred. U-Boot, Barebox, at91bootstrap and at91bootstrap3 also have separate options, but these do not provide any advantage over `BR2_GLOBAL_PATCH_DIR` and will likely be removed in the future.
12. If you need to add project-specific packages, create `package/<manufacturer>/` and place your packages in that directory. Create an overall `<manufacturer>.mk` file that includes the `.mk` files of all your packages. Create an overall `Config.in` file that sources the `Config.in` files of all your packages. Include this `Config.in` file from Buildroot’s `package/Config.in` file.
13. `make savedefconfig` to save the buildroot configuration.
14. `cp defconfig configs/<boardname>_defconfig`

