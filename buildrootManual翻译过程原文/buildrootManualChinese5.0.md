## 第9章 项目专属定制

对于特定项目，你可能需要执行的典型操作包括：

- 配置 Buildroot（包括构建选项和工具链、引导加载程序、内核、软件包和文件系统镜像类型的选择）
- 配置其他组件，如 Linux 内核和 BusyBox（原文：BusyBox）
- 定制生成的目标文件系统
  - 在目标文件系统上添加或覆盖文件（使用 `BR2_ROOTFS_OVERLAY`）
  - 修改或删除目标文件系统上的文件（使用 `BR2_ROOTFS_POST_BUILD_SCRIPT`）
  - 在生成文件系统镜像前运行任意命令（使用 `BR2_ROOTFS_POST_BUILD_SCRIPT`）
  - 设置文件权限和所有权（使用 `BR2_ROOTFS_DEVICE_TABLE`）
  - 添加自定义设备节点（使用 `BR2_ROOTFS_STATIC_DEVICE_TABLE`）
- 添加自定义用户账户（使用 `BR2_ROOTFS_USERS_TABLES`）
- 在生成文件系统镜像后运行任意命令（使用 `BR2_ROOTFS_POST_IMAGE_SCRIPT`）
- 为某些软件包添加项目专属补丁（使用 `BR2_GLOBAL_PATCH_DIR`）
- 添加项目专属软件包

关于此类*项目专属*定制的重要说明：请仔细考虑哪些更改确实仅针对你的项目，哪些更改对项目外的开发者也有用。Buildroot 社区强烈建议并鼓励将改进、软件包和板级支持上游到官方 Buildroot 项目。当然，有时由于更改高度专用或专有，无法或不适合上游。

本章介绍如何在 Buildroot 中进行此类项目专属定制，以及如何以可复现的方式存储这些定制内容，即使在执行 *make clean* 后也能构建相同的镜像。按照推荐策略，你甚至可以使用同一个 Buildroot 树来构建多个不同的项目！

## 9.1 推荐的目录结构

在为你的项目定制 Buildroot 时，你将创建一个或多个项目专属文件，这些文件需要存放在某处。虽然大多数文件可以放在*任意*位置（因为其路径会在 Buildroot 配置中指定），但 Buildroot 开发者推荐采用本节描述的特定目录结构。

与该目录结构正交的是，你可以选择将其放置在 Buildroot 树内部，或通过 br2-external（原文：br2-external）树放在外部。这两种方式都可以，选择权在你。

```
+-- board/
|   +-- <company>/
|       +-- <boardname>/
|           +-- linux.config
|           +-- busybox.config
|           +-- <其他配置文件>
|           +-- post_build.sh
|           +-- post_image.sh
|           +-- rootfs_overlay/
|           |   +-- etc/
|           |   +-- <一些文件>
|           +-- patches/
|               +-- foo/
|               |   +-- <一些补丁>
|               +-- libbar/
|                   +-- <其他补丁>
|
+-- configs/
|   +-- <boardname>_defconfig
|
+-- package/
|   +-- <company>/
|       +-- Config.in（如果未使用 br2-external 树）
|       +-- <company>.mk（如果未使用 br2-external 树）
|       +-- package1/
|       |    +-- Config.in
|       |    +-- package1.mk
|       +-- package2/
|           +-- Config.in
|           +-- package2.mk
|
+-- Config.in（如果使用 br2-external 树）
+-- Makefile（如果使用自定义顶层 makefile）
+-- external.mk（如果使用 br2-external 树）
+-- external.desc（如果使用 br2-external 树）
```

上述文件的详细说明将在本章后续部分给出。

注意：如果你选择将该结构放在 Buildroot 树外部（即 br2-external 树），则 <company> 和可能的 <boardname> 组件可以省略。

### 9.1.1 分层定制的实现

用户经常会有多个相关项目，这些项目部分需要相同的定制。与其为每个项目重复这些定制，推荐采用分层定制方法，如本节所述。

Buildroot 提供的几乎所有定制方法（如 post-build 脚本和 root 文件系统覆盖）都支持以空格分隔的项目列表。指定的项目总是按顺序（从左到右）处理。通过创建多个项目（如一个用于通用定制，另一个用于项目专属定制），可以避免不必要的重复。每一层通常对应 `board/<company>/` 下的一个独立目录。根据你的项目情况，甚至可以引入多于两层。

例如，用户有两个定制层 *common* 和 *fooboard*，其目录结构如下：

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
            +-- <其他配置文件>
            +-- post_build.sh
            +-- rootfs_overlay/
            |   +-- ...
            +-- patches/
                +-- ...
```

例如，用户将 `BR2_GLOBAL_PATCH_DIR` 配置项设置为：

```
BR2_GLOBAL_PATCH_DIR="board/<company>/common/patches board/<company>/fooboard/patches"
```

则会先应用 *common* 层的补丁，再应用 *fooboard* 层的补丁。

### 9.1.2 自定义顶层 Makefile（原文：Makefile）

通常你会从 buildroot 源码目录启动 Buildroot，指定 `BR2_EXTERNAL` 和 `O` 到正确的位置以进行构建。你可以通过在 br2-external 中添加一个 Makefile 来简化此过程，该文件设置这些变量并调用 buildroot。

你还可以为各种任务在此 Makefile 中添加自定义规则，例如将多个配置集成到单一镜像、上传到发布服务器或测试设备、包含多个 br2-external 配置等。

一个基础的 Makefile 如下。假设 buildroot 源码位于 `buildroot` 子目录（如作为 git 子模块）。它确保下载目录在不同构建间共享，并将输出目录组织在 `outputs/` 下。

```
# SPDX-License-Identifier: GPL-2.0

# 通过禁用默认规则避免意外
MAKEFLAGS += --no-builtin-rules
.SUFFIXES:

THIS_EXTERNAL_PATH := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))

# 将下载内容放在此目录而不是 Buildroot 目录
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
