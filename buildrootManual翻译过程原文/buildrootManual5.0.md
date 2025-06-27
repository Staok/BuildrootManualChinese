## Chapter 9. Project-specific customization

Typical actions you may need to perform for a given project are:

- configuring Buildroot (including build options and toolchain, bootloader, kernel, package and filesystem image type selection)
- configuring other components, like the Linux kernel and BusyBox
- customizing the generated target filesystem
  - adding or overwriting files on the target filesystem (using `BR2_ROOTFS_OVERLAY`)
  - modifying or deleting files on the target filesystem (using `BR2_ROOTFS_POST_BUILD_SCRIPT`)
  - running arbitrary commands prior to generating the filesystem image (using `BR2_ROOTFS_POST_BUILD_SCRIPT`)
  - setting file permissions and ownership (using `BR2_ROOTFS_DEVICE_TABLE`)
  - adding custom devices nodes (using `BR2_ROOTFS_STATIC_DEVICE_TABLE`)
- adding custom user accounts (using `BR2_ROOTFS_USERS_TABLES`)
- running arbitrary commands after generating the filesystem image (using `BR2_ROOTFS_POST_IMAGE_SCRIPT`)
- adding project-specific patches to some packages (using `BR2_GLOBAL_PATCH_DIR`)
- adding project-specific packages

An important note regarding such *project-specific* customizations: please carefully consider which changes are indeed project-specific and which changes are also useful to developers outside your project. The Buildroot community highly recommends and encourages the upstreaming of improvements, packages and board support to the official Buildroot project. Of course, it is sometimes not possible or desirable to upstream because the changes are highly specific or proprietary.

This chapter describes how to make such project-specific customizations in Buildroot and how to store them in a way that you can build the same image in a reproducible way, even after running *make clean*. By following the recommended strategy, you can even use the same Buildroot tree to build multiple distinct projects!

## 9.1. Recommended directory structure

When customizing Buildroot for your project, you will be creating one or more project-specific files that need to be stored somewhere. While most of these files could be placed in *any* location as their path is to be specified in the Buildroot configuration, the Buildroot developers recommend a specific directory structure which is described in this section.

Orthogonal to this directory structure, you can choose *where* you place this structure itself: either inside the Buildroot tree, or outside of it using a br2-external tree. Both options are valid, the choice is up to you.

```
+-- board/
|   +-- <company>/
|       +-- <boardname>/
|           +-- linux.config
|           +-- busybox.config
|           +-- <other configuration files>
|           +-- post_build.sh
|           +-- post_image.sh
|           +-- rootfs_overlay/
|           |   +-- etc/
|           |   +-- <some files>
|           +-- patches/
|               +-- foo/
|               |   +-- <some patches>
|               +-- libbar/
|                   +-- <some other patches>
|
+-- configs/
|   +-- <boardname>_defconfig
|
+-- package/
|   +-- <company>/
|       +-- Config.in (if not using a br2-external tree)
|       +-- <company>.mk (if not using a br2-external tree)
|       +-- package1/
|       |    +-- Config.in
|       |    +-- package1.mk
|       +-- package2/
|           +-- Config.in
|           +-- package2.mk
|
+-- Config.in (if using a br2-external tree)
+-- Makefile (if using a custom top makefile)
+-- external.mk (if using a br2-external tree)
+-- external.desc (if using a br2-external tree)
```

Details on the files shown above are given further in this chapter.

Note: if you choose to place this structure outside of the Buildroot tree but in a br2-external tree, the <company> and possibly <boardname> components may be superfluous and can be left out.

### 9.1.1. Implementing layered customizations

It is quite common for a user to have several related projects that partly need the same customizations. Instead of duplicating these customizations for each project, it is recommended to use a layered customization approach, as explained in this section.

Almost all of the customization methods available in Buildroot, like post-build scripts and root filesystem overlays, accept a space-separated list of items. The specified items are always treated in order, from left to right. By creating more than one such item, one for the common customizations and another one for the really project-specific customizations, you can avoid unnecessary duplication. Each layer is typically embodied by a separate directory inside `board/<company>/`. Depending on your projects, you could even introduce more than two layers.

An example directory structure for where a user has two customization layers *common* and *fooboard* is:

```
+-- board/
    +-- <company>/
        +-- common/
        |   +-- post_build.sh
        |   +-- rootfs_overlay/
        |   |   +-- ...
        |   +-- patches/
        |       +-- ...
        |
        +-- fooboard/
            +-- linux.config
            +-- busybox.config
            +-- <other configuration files>
            +-- post_build.sh
            +-- rootfs_overlay/
            |   +-- ...
            +-- patches/
                +-- ...
```

For example, if the user has the `BR2_GLOBAL_PATCH_DIR` configuration option set as:

```
BR2_GLOBAL_PATCH_DIR="board/<company>/common/patches board/<company>/fooboard/patches"
```

then first the patches from the *common* layer would be applied, followed by the patches from the *fooboard* layer.

### 9.1.2. Custom top Makefile

You normally launch Buildroot from the buildroot source directory, pointing `BR2_EXTERNAL` and `O` to the right places for the build you want to make. You can simplify this by adding a Makefile to your br2-external that sets these variables and calls into buildroot.

You can add additional, custom rules to this Makefile for various tasks you need to perform, e.g. integrate multiple configurations into a single image, upload to a release server or to a test device, include multiple br2-external configurations, etc.

A basic Makefile looks like this. It assumes the buildroot source is available (e.g. as a git submodule) in the `buildroot` subdirectory. It makes sure that the download directory is shared between different builds, and it organizes the output directories in a structure under `outputs/`.

```
# SPDX-License-Identifier: GPL-2.0

# Avoid surprises by disabling default rules
MAKEFLAGS += --no-builtin-rules
.SUFFIXES:

THIS_EXTERNAL_PATH := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))

# Put downloads in this directory instead of in the Buildroot directory
ifeq ($(BR2_DL_DIR),)
BR2_DL_DIR = $(THIS_EXTERNAL_PATH)/dl
endif

OUTPUT_BASEDIR = $(THIS_EXTERNAL_PATH)/output
OUTPUT_DIR = $(OUTPUT_BASEDIR)/$(patsubst %_defconfig,%,$@)

MAKE_BUILDROOT = $(MAKE) -C $(THIS_EXTERNAL_PATH)/buildroot BR2_EXTERNAL=$(THIS_EXTERNAL_PATH)

%: $(THIS_EXTERNAL_PATH)/configs/%
        $(MAKE_BUILDROOT) O=$(OUTPUT_DIR) $@
        sed -i /^BR2_DL_DIR=.*/s%%BR2_DL_DIR=$(BR2_DL_DIR)% $(OUTPUT_DIR)/.config
```
