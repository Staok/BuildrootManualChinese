#### `provides/` 目录

对于某些软件包，Buildroot 提供了两个（或更多）API 兼容实现的选择。例如，可以选择 libjpeg 或 jpeg-turbo；可以选择 openssl 或 libressl；也可以选择已知的、预配置的工具链……

br2-external（原文：br2-external）可以通过提供一组定义这些替代项的文件来扩展这些选择：

- `provides/toolchains.in` 定义预配置工具链，将会在工具链选择中列出；
- `provides/jpeg.in` 定义 libjpeg 的替代实现；
- `provides/openssl.in` 定义 openssl 的替代实现；
- `provides/skeleton.in` 定义 skeleton（原文：skeleton）的替代实现；
- `provides/init.in` 定义 init 系统的替代实现，可用于为 init 选择默认 skeleton。

#### 自由格式内容

你可以在这里存放所有板级专属配置文件，如内核配置、root 文件系统覆盖或 Buildroot 允许设置位置的其他配置文件（通过 `BR2_EXTERNAL_$(NAME)_PATH` 变量）。例如，可以如下设置全局补丁目录、rootfs 覆盖和内核配置文件的路径（如通过 `make menuconfig` 填写这些选项）：

```
BR2_GLOBAL_PATCH_DIR=$(BR2_EXTERNAL_BAR_42_PATH)/patches/
BR2_ROOTFS_OVERLAY=$(BR2_EXTERNAL_BAR_42_PATH)/board/<boardname>/overlay/
BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE=$(BR2_EXTERNAL_BAR_42_PATH)/board/<boardname>/kernel.config
```

#### 额外的 Linux 内核扩展

可以通过在 br2-external 树根目录下的 `linux/` 目录中存放扩展，来添加额外的 Linux 内核扩展（详见[18.22.2节“linux-kernel-extensions”](https://buildroot.org/downloads/manual/manual.html#linux-kernel-ext)）。

#### 示例布局

以下是一个使用了 br2-external 所有特性的示例布局（为便于说明，相关文件的示例内容会在其上方展示，内容均为虚构）：

```
/path/to/br2-ext-tree/
  |- external.desc
  |     |name: BAR_42
  |     |desc: Example br2-external tree
  |     `----
  |
  |- Config.in
  |     |source "$BR2_EXTERNAL_BAR_42_PATH/toolchain/toolchain-external-mine/Config.in.options"
  |     |source "$BR2_EXTERNAL_BAR_42_PATH/package/pkg-1/Config.in"
  |     |source "$BR2_EXTERNAL_BAR_42_PATH/package/pkg-2/Config.in"
  |     |source "$BR2_EXTERNAL_BAR_42_PATH/package/my-jpeg/Config.in"
  |     |
  |     |config BAR_42_FLASH_ADDR
  |     |    hex "my-board flash address"
  |     |    default 0x10AD
  |     `----
  |
  |- external.mk
  |     |include $(sort $(wildcard $(BR2_EXTERNAL_BAR_42_PATH)/package/*/*.mk))
  |     |include $(sort $(wildcard $(BR2_EXTERNAL_BAR_42_PATH)/toolchain/*/*.mk))
  |     |
  |     |flash-my-board:
  |     |    $(BR2_EXTERNAL_BAR_42_PATH)/board/my-board/flash-image \
  |     |        --image $(BINARIES_DIR)/image.bin \
  |     |        --address $(BAR_42_FLASH_ADDR)
  |     `----
  |
  |- package/pkg-1/Config.in
  |     |config BR2_PACKAGE_PKG_1
  |     |    bool "pkg-1"
  |     |    help
  |     |      Some help about pkg-1
  |     `----
  |- package/pkg-1/pkg-1.hash
  |- package/pkg-1/pkg-1.mk
  |     |PKG_1_VERSION = 1.2.3
  |     |PKG_1_SITE = /some/where/to/get/pkg-1
  |     |PKG_1_LICENSE = blabla
  |     |
  |     |define PKG_1_INSTALL_INIT_SYSV
  |     |    $(INSTALL) -D -m 0755 $(PKG_1_PKGDIR)/S99my-daemon \
  |     |                          $(TARGET_DIR)/etc/init.d/S99my-daemon
  |     |endef
  |     |
  |     |$(eval $(autotools-package))
  |     `----
  |- package/pkg-1/S99my-daemon
  |
  |- package/pkg-2/Config.in
  |- package/pkg-2/pkg-2.hash
  |- package/pkg-2/pkg-2.mk
  |
  |- provides/jpeg.in
  |     |config BR2_PACKAGE_MY_JPEG
  |     |    bool "my-jpeg"
  |     `----
  |- package/my-jpeg/Config.in
  |     |config BR2_PACKAGE_PROVIDES_JPEG
  |     |    default "my-jpeg" if BR2_PACKAGE_MY_JPEG
  |     `----
  |- package/my-jpeg/my-jpeg.mk
  |     |# 这是一个常规 package .mk 文件
  |     |MY_JPEG_VERSION = 1.2.3
  |     |MY_JPEG_SITE = https://example.net/some/place
  |     |MY_JPEG_PROVIDES = jpeg
  |     |$(eval $(autotools-package))
  |     `----
  |
  |- provides/init.in
  |     |config BR2_INIT_MINE
  |     |    bool "my custom init"
  |     |    select BR2_PACKAGE_MY_INIT
  |     |    select BR2_PACKAGE_SKELETON_INIT_MINE if BR2_ROOTFS_SKELETON_DEFAULT
  |     `----
  |
  |- provides/skeleton.in
  |     |config BR2_ROOTFS_SKELETON_MINE
  |     |    bool "my custom skeleton"
  |     |    select BR2_PACKAGE_SKELETON_MINE
  |     `----
  |- package/skeleton-mine/Config.in
  |     |config BR2_PACKAGE_SKELETON_MINE
  |     |    bool
  |     |    select BR2_PACKAGE_HAS_SKELETON
  |     |
  |     |config BR2_PACKAGE_PROVIDES_SKELETON
  |     |    default "skeleton-mine" if BR2_PACKAGE_SKELETON_MINE
  |     `----
  |- package/skeleton-mine/skeleton-mine.mk
  |     |SKELETON_MINE_ADD_TOOLCHAIN_DEPENDENCY = NO
  |     |SKELETON_MINE_ADD_SKELETON_DEPENDENCY = NO
  |     |SKELETON_MINE_PROVIDES = skeleton
  |     |SKELETON_MINE_INSTALL_STAGING = YES
  |     |$(eval $(generic-package))
  |     `----
  |
  |- provides/toolchains.in
  |     |config BR2_TOOLCHAIN_EXTERNAL_MINE
  |     |    bool "my custom toolchain"
  |     |    depends on BR2_some_arch
  |     |    select BR2_INSTALL_LIBSTDCPP
  |     `----
  |- toolchain/toolchain-external-mine/Config.in.options
  |     |if BR2_TOOLCHAIN_EXTERNAL_MINE
  |     |config BR2_TOOLCHAIN_EXTERNAL_PREFIX
  |     |    default "arch-mine-linux-gnu"
  |     |config BR2_PACKAGE_PROVIDES_TOOLCHAIN_EXTERNAL
  |     |    default "toolchain-external-mine"
  |     |endif
  |     `----
  |- toolchain/toolchain-external-mine/toolchain-external-mine.mk
  |     |TOOLCHAIN_EXTERNAL_MINE_SITE = https://example.net/some/place
  |     |TOOLCHAIN_EXTERNAL_MINE_SOURCE = my-toolchain.tar.gz
  |     |$(eval $(toolchain-external-package))
  |     `----
  |
  |- linux/Config.ext.in
  |     |config BR2_LINUX_KERNEL_EXT_EXAMPLE_DRIVER
  |     |    bool "example-external-driver"
  |     |    help
  |     |      Example external driver
  |     |---
  |- linux/linux-ext-example-driver.mk
  |
  |- configs/my-board_defconfig
  |     |BR2_GLOBAL_PATCH_DIR="$(BR2_EXTERNAL_BAR_42_PATH)/patches/"
  |     |BR2_ROOTFS_OVERLAY="$(BR2_EXTERNAL_BAR_42_PATH)/board/my-board/overlay/"
  |     |BR2_ROOTFS_POST_IMAGE_SCRIPT="$(BR2_EXTERNAL_BAR_42_PATH)/board/my-board/post-image.sh"
  |     |BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE="$(BR2_EXTERNAL_BAR_42_PATH)/board/my-board/kernel.config"
  |     `----
  |
  |- patches/linux/0001-some-change.patch
  |- patches/linux/0002-some-other-change.patch
  |- patches/busybox/0001-fix-something.patch
  |
  |- board/my-board/kernel.config
  |- board/my-board/overlay/var/www/index.html
  |- board/my-board/overlay/var/www/my.css
  |- board/my-board/flash-image
  `- board/my-board/post-image.sh
        |#!/bin/sh
        |generate-my-binary-image \
        |    --root ${BINARIES_DIR}/rootfs.tar \
        |    --kernel ${BINARIES_DIR}/zImage \
        |    --dtb ${BINARIES_DIR}/my-board.dtb \
        |    --output ${BINARIES_DIR}/image.bin
        `----
```

br2-external 树会在 menuconfig（原文：menuconfig）中显示（布局已展开）：

```
External options  --->
    *** Example br2-external tree (in /path/to/br2-ext-tree/)
    [ ] pkg-1
    [ ] pkg-2
    (0x10AD) my-board flash address
```

如果你使用了多个 br2-external 树，效果如下（布局已展开，第二个名称为 `FOO_27`，但 `external.desc` 没有 `desc:` 字段）：

```
External options  --->
    Example br2-external tree  --->
        *** Example br2-external tree (in /path/to/br2-ext-tree)
        [ ] pkg-1
        [ ] pkg-2
        (0x10AD) my-board flash address
    FOO_27  --->
        *** FOO_27 (in /path/to/another-br2-ext)
        [ ] foo
        [ ] bar
```

此外，jpeg 提供者会在 jpeg 选择项中显示：

```
Target packages  --->
    Libraries  --->
        Graphics  --->
            [*] jpeg support
                jpeg variant ()  --->
                    ( ) jpeg
                    ( ) jpeg-turbo
                        *** jpeg from: Example br2-external tree ***
                    (X) my-jpeg
                        *** jpeg from: FOO_27 ***
                    ( ) another-jpeg
```

工具链同理：

```
Toolchain  --->
    Toolchain ()  --->
        ( ) Custom toolchain
            *** Toolchains from: Example br2-external tree ***
        (X) my custom toolchain
```

**注意**：`toolchain/toolchain-external-mine/Config.in.options` 中的工具链选项不会出现在 `Toolchain` 菜单中。它们必须在 br2-external 顶层 `Config.in` 中显式包含，因此会出现在 `External options` 菜单中。
