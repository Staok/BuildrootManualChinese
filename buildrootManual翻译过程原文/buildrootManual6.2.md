### 9.5.1. Setting file permissions and ownership and adding custom devices nodes

Sometimes it is needed to set specific permissions or ownership on files or device nodes. For example, certain files may need to be owned by root. Since the post-build scripts are not run as root, you cannot do such changes from there unless you use an explicit fakeroot from the post-build script.

Instead, Buildroot provides support for so-called *permission tables*. To use this feature, set config option `BR2_ROOTFS_DEVICE_TABLE` to a space-separated list of permission tables, regular text files following the [makedev syntax](https://buildroot.org/downloads/manual/manual.html#makedev-syntax).

If you are using a static device table (i.e. not using `devtmpfs`, `mdev`, or `(e)udev`) then you can add device nodes using the same syntax, in so-called *device tables*. To use this feature, set config option `BR2_ROOTFS_STATIC_DEVICE_TABLE` to a space-separated list of device tables.

As shown in [Section 9.1, “Recommended directory structure”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure), the recommended location for such files is `board/<company>/<boardname>/`.

It should be noted that if the specific permissions or device nodes are related to a specific application, you should set variables `FOO_PERMISSIONS` and `FOO_DEVICES` in the package’s `.mk` file instead (see [Section 18.6.2, “`generic-package` reference”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)).

## 9.6. Adding custom user accounts

Sometimes it is needed to add specific users in the target system. To cover this requirement, Buildroot provides support for so-called *users tables*. To use this feature, set config option `BR2_ROOTFS_USERS_TABLES` to a space-separated list of users tables, regular text files following the [makeusers syntax](https://buildroot.org/downloads/manual/manual.html#makeuser-syntax).

As shown in [Section 9.1, “Recommended directory structure”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure), the recommended location for such files is `board/<company>/<boardname>/`.

It should be noted that if the custom users are related to a specific application, you should set variable `FOO_USERS` in the package’s `.mk` file instead (see [Section 18.6.2, “`generic-package` reference”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)).

## 9.7. Customization *after* the images have been created

While post-build scripts ([Section 9.5, “Customizing the generated target filesystem”](https://buildroot.org/downloads/manual/manual.html#rootfs-custom)) are run *before* building the filesystem image, kernel and bootloader, ***\*post-image scripts\**** can be used to perform some specific actions *after* all images have been created.

Post-image scripts can for example be used to automatically extract your root filesystem tarball in a location exported by your NFS server, or to create a special firmware image that bundles your root filesystem and kernel image, or any other custom action required for your project.

To enable this feature, specify a space-separated list of post-image scripts in config option `BR2_ROOTFS_POST_IMAGE_SCRIPT` (in the `System configuration` menu). If you specify a relative path, it will be relative to the root of the Buildroot tree.

Just like post-build scripts, post-image scripts are run with the main Buildroot tree as current working directory. The path to the `images` output directory is passed as the first argument to each script. If the config option `BR2_ROOTFS_POST_SCRIPT_ARGS` is not empty, these arguments will be passed to the script too. All the scripts will be passed the exact same set of arguments, it is not possible to pass different sets of arguments to each script.

Note that the arguments from `BR2_ROOTFS_POST_SCRIPT_ARGS` will also be passed to post-build and post-fakeroot scripts. If you want to use arguments that are only used for the post-image scripts you can use `BR2_ROOTFS_POST_IMAGE_SCRIPT_ARGS`.

Again just like for the post-build scripts, the scripts have access to the environment variables `BR2_CONFIG`, `HOST_DIR`, `STAGING_DIR`, `TARGET_DIR`, `BUILD_DIR`, `BINARIES_DIR`, `CONFIG_DIR`, `BASE_DIR`, and `PARALLEL_JOBS`.

The post-image scripts will be executed as the user that executes Buildroot, which should normally *not* be the root user. Therefore, any action requiring root permissions in one of these scripts will require special handling (usage of fakeroot or sudo), which is left to the script developer.

## 9.8. Adding project-specific patches and hashes

### 9.8.1. Providing extra patches

It is sometimes useful to apply *extra* patches to packages - on top of those provided in Buildroot. This might be used to support custom features in a project, for example, or when working on a new architecture.

The `BR2_GLOBAL_PATCH_DIR` configuration option can be used to specify a space separated list of one or more directories containing package patches.

For a specific version `<packageversion>` of a specific package `<packagename>`, patches are applied from `BR2_GLOBAL_PATCH_DIR` as follows:

1. For every directory - `<global-patch-dir>` - that exists in `BR2_GLOBAL_PATCH_DIR`, a `<package-patch-dir>` will be determined as follows:
   - `<global-patch-dir>/<packagename>/<packageversion>/` if the directory exists.
   - Otherwise, `<global-patch-dir>/<packagename>` if the directory exists.
2. Patches will then be applied from a `<package-patch-dir>` as follows:
   - If a `series` file exists in the package directory, then patches are applied according to the `series` file;
   - Otherwise, patch files matching `*.patch` are applied in alphabetical order. So, to ensure they are applied in the right order, it is highly recommended to name the patch files like this: `<number>-<description>.patch`, where `<number>` refers to the *apply order*.

For information about how patches are applied for a package, see [Section 19.2, “How patches are applied”](https://buildroot.org/downloads/manual/manual.html#patch-apply-order)

The `BR2_GLOBAL_PATCH_DIR` option is the preferred method for specifying a custom patch directory for packages. It can be used to specify a patch directory for any package in buildroot. It should also be used in place of the custom patch directory options that are available for packages such as U-Boot and Barebox. By doing this, it will allow a user to manage their patches from one top-level directory.

The exception to `BR2_GLOBAL_PATCH_DIR` being the preferred method for specifying custom patches is `BR2_LINUX_KERNEL_PATCH`. `BR2_LINUX_KERNEL_PATCH` should be used to specify kernel patches that are available at a URL. ***\*Note:\**** `BR2_LINUX_KERNEL_PATCH` specifies kernel patches that are applied after patches available in `BR2_GLOBAL_PATCH_DIR`, as it is done from a post-patch hook of the Linux package.

