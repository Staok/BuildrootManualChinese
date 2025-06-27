## 18.6. 针对具有特定构建系统的软件包的基础设施

所谓“具有特定构建系统的软件包”，是指那些构建系统不是标准构建系统（如 autotools 或 CMake）的所有软件包。这通常包括基于手写 Makefile 或 shell 脚本的构建系统的软件包。

### 18.6.1. `generic-package` 教程

```
01: ################################################################################
02: #
03: # libfoo
04: #
05: ################################################################################
06:
07: LIBFOO_VERSION = 1.0
08: LIBFOO_SOURCE = libfoo-$(LIBFOO_VERSION).tar.gz
09: LIBFOO_SITE = http://www.foosoftware.org/download
10: LIBFOO_LICENSE = GPL-3.0+
11: LIBFOO_LICENSE_FILES = COPYING
12: LIBFOO_INSTALL_STAGING = YES
13: LIBFOO_CONFIG_SCRIPTS = libfoo-config
14: LIBFOO_DEPENDENCIES = host-libaaa libbbb
15:
16: define LIBFOO_BUILD_CMDS
17:     $(MAKE) $(TARGET_CONFIGURE_OPTS) -C $(@D) all
18: endef
19:
20: define LIBFOO_INSTALL_STAGING_CMDS
21:     $(INSTALL) -D -m 0755 $(@D)/libfoo.a $(STAGING_DIR)/usr/lib/libfoo.a
22:     $(INSTALL) -D -m 0644 $(@D)/foo.h $(STAGING_DIR)/usr/include/foo.h
23:     $(INSTALL) -D -m 0755 $(@D)/libfoo.so* $(STAGING_DIR)/usr/lib
24: endef
25:
26: define LIBFOO_INSTALL_TARGET_CMDS
27:     $(INSTALL) -D -m 0755 $(@D)/libfoo.so* $(TARGET_DIR)/usr/lib
28:     $(INSTALL) -d -m 0755 $(TARGET_DIR)/etc/foo.d
29: endef
30:
31: define LIBFOO_USERS
32:     foo -1 libfoo -1 * - - - LibFoo daemon
33: endef
34:
35: define LIBFOO_DEVICES
36:     /dev/foo c 666 0 0 42 0 - - -
37: endef
38:
39: define LIBFOO_PERMISSIONS
40:     /bin/foo f 4755 foo libfoo - - - - -
41: endef
42:
43: $(eval $(generic-package))
```

该 Makefile 在第 7 至 11 行给出了元数据信息：软件包的版本（`LIBFOO_VERSION`）、包含该软件包的 tarball 文件名（`LIBFOO_SOURCE`，推荐使用 xz 压缩的 tarball）、tarball 可下载的互联网地址（`LIBFOO_SITE`）、许可证（`LIBFOO_LICENSE`）以及包含许可证文本的文件（`LIBFOO_LICENSE_FILES`）。所有变量都必须以相同的前缀开头，这里是 `LIBFOO_`。该前缀始终是软件包名称的大写形式（见下文了解软件包名称的定义位置）。

第 12 行指定该软件包需要向 staging 空间安装内容。对于库（library）来说，这通常是必需的，因为它们必须在 staging 空间安装头文件和其他开发文件。这将确保 `LIBFOO_INSTALL_STAGING_CMDS` 变量中列出的命令会被执行。

第 13 行指定需要对某些在 `LIBFOO_INSTALL_STAGING_CMDS` 阶段安装的 *libfoo-config* 文件进行修正。这些 *-config 文件是可执行的 shell 脚本，位于 *$(STAGING_DIR)/usr/bin* 目录下，被其他第三方软件包调用，用于获取该软件包的位置和链接参数。

问题在于，这些 *-config 文件默认给出的链接参数是主机系统的，无法用于交叉编译。

例如：*-I/usr/include* 而不是 *-I$(STAGING_DIR)/usr/include*，或 *-L/usr/lib* 而不是 *-L$(STAGING_DIR)/usr/lib*

因此需要用 sed 对这些脚本做一些处理，使其给出正确的参数。`LIBFOO_CONFIG_SCRIPTS` 的参数是需要修正的 shell 脚本文件名，这些名称都相对于 *$(STAGING_DIR)/usr/bin*，如有多个文件名可用空格分隔。

此外，`LIBFOO_CONFIG_SCRIPTS` 中列出的脚本会从 `$(TARGET_DIR)/usr/bin` 中移除，因为它们在目标系统上不需要。

**示例 18.1. 配置脚本：*divine* 软件包**

软件包 divine 安装 shell 脚本 *$(STAGING_DIR)/usr/bin/divine-config*。

因此其修正方式如下：

```
DIVINE_CONFIG_SCRIPTS = divine-config
```

**示例 18.2. 配置脚本：*imagemagick* 软件包：**

软件包 imagemagick 安装以下脚本：*$(STAGING_DIR)/usr/bin/{Magick,Magick++,MagickCore,MagickWand,Wand}-config*

因此其修正方式如下：

```
IMAGEMAGICK_CONFIG_SCRIPTS = \
   Magick-config Magick++-config \
   MagickCore-config MagickWand-config Wand-config
```

第 14 行指定该软件包依赖的软件包列表。这些依赖以小写包名列出，可以是目标（target）包（不带 `host-` 前缀）或主机（host）包（带 `host-` 前缀）。Buildroot 会确保所有这些依赖包在当前包开始配置前已构建并安装。

Makefile 的其余部分（第 16~29 行）定义了软件包配置、编译和安装各阶段应执行的操作。`LIBFOO_BUILD_CMDS` 指定构建该软件包应执行的步骤。`LIBFOO_INSTALL_STAGING_CMDS` 指定安装到 staging 空间应执行的步骤。`LIBFOO_INSTALL_TARGET_CMDS` 指定安装到目标空间应执行的步骤。

所有这些步骤都依赖于 `$(@D)` 变量，该变量包含软件包源码解压后的目录。

第 31~33 行定义了该软件包使用的用户（如以非 root 身份运行守护进程）（`LIBFOO_USERS`）。

在第 35~37 行，我们定义了该软件包使用的设备节点文件（`LIBFOO_DEVICES`）。

在第 39~41 行，我们定义了该软件包安装的特定文件需要设置的权限（`LIBFOO_PERMISSIONS`）。

最后，在第 43 行，我们调用了 `generic-package` 函数。该函数会根据前面定义的变量，自动生成所有让你的软件包能够正常工作的 Makefile 代码。
