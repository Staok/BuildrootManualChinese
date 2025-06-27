### 9.8.2 提供额外哈希

Buildroot 自带一份[哈希列表](https://buildroot.org/downloads/manual/manual.html#adding-packages-hash)，用于校验下载归档文件或从 VCS 检出后本地生成的归档文件的完整性。但对于可自定义版本的软件包（如自定义版本号、远程 tarball URL 或 VCS 仓库及变更集），Buildroot 无法自带哈希。

如果你关心此类下载的完整性，可以为 Buildroot 提供一份额外哈希列表，用于校验任意下载文件。这些额外哈希的查找方式与额外补丁类似：对于 `BR2_GLOBAL_PATCH_DIR` 中的每个目录，Buildroot 会依次查找以下文件，找到第一个存在的文件用于校验：

- `<global-patch-dir>/<packagename>/<packageversion>/<packagename>.hash`
- `<global-patch-dir>/<packagename>/<packagename>.hash`

可以使用 `utils/add-custom-hashes` 脚本生成这些文件。

## 9.9 添加项目专属软件包

一般来说，任何新软件包都应直接添加到 `package` 目录并提交到 Buildroot 上游项目。如何向 Buildroot 添加软件包详见[第18章“向 Buildroot 添加新软件包”](https://buildroot.org/downloads/manual/manual.html#adding-packages)，此处不再赘述。但你的项目可能需要无法上游的专有软件包，本节介绍如何将此类项目专属软件包保存在项目专属目录。

如[9.1节“推荐的目录结构”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)所示，推荐将项目专属软件包放在 `package/<company>/`。如果使用 br2-external 树（见[9.2节“保持定制内容在 Buildroot 之外”](https://buildroot.org/downloads/manual/manual.html#outside-br-custom)），推荐放在 br2-external 树下的 `package/` 子目录。

但 Buildroot 默认不会识别这些位置的软件包，需要额外操作。正如[第18章“向 Buildroot 添加新软件包”](https://buildroot.org/downloads/manual/manual.html#adding-packages)所述，Buildroot 的软件包主要由两个文件组成：`.mk` 文件（描述如何构建软件包）和 `Config.in` 文件（描述该软件包的配置选项）。

Buildroot 会自动包含 `package` 目录一级子目录下的 `.mk` 文件（模式为 `package/*/*.mk`）。如果希望 Buildroot 包含更深层目录（如 `package/<company>/package1/`）下的 `.mk` 文件，只需在一级子目录下添加一个 `.mk` 文件，将这些 `.mk` 文件包含进来。例如，在 `package/<company>/<company>.mk` 文件中写入如下内容（假设只多一级目录）：

```
include $(sort $(wildcard package/<company>/*/*.mk))
```

对于 `Config.in` 文件，在 `package/<company>/Config.in` 中依次包含所有软件包的 `Config.in` 文件。由于 kconfig 的 source 命令不支持通配符，需手动列出所有文件。例如：

```
source "package/<company>/package1/Config.in"
source "package/<company>/package2/Config.in"
```

建议将该新建的 `package/<company>/Config.in` 文件以公司专属菜单的方式包含到 `package/Config.in`，以便后续 Buildroot 升级时便于合并。

如果使用 br2-external 树，相关文件的写法见[9.2节“保持定制内容在 Buildroot 之外”](https://buildroot.org/downloads/manual/manual.html#outside-br-custom)。

## 9.10 项目专属定制内容存储快速指南

本章前文介绍了多种项目专属定制方法，本节将以步骤形式总结如何存储项目专属定制内容。与项目无关的步骤可跳过。

1. `make menuconfig` 配置工具链、软件包和内核。
2. `make linux-menuconfig` 更新内核配置，其他配置（如 busybox、uclibc 等）同理。
3. `mkdir -p board/<manufacturer>/<boardname>`
4. 将以下选项设置为 `board/<manufacturer>/<boardname>/<package>.config`（如有需要）：
   - `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE`
   - `BR2_PACKAGE_BUSYBOX_CONFIG`
   - `BR2_UCLIBC_CONFIG`
   - `BR2_TARGET_AT91BOOTSTRAP3_CUSTOM_CONFIG_FILE`
   - `BR2_TARGET_BAREBOX_CUSTOM_CONFIG_FILE`
   - `BR2_TARGET_UBOOT_CUSTOM_CONFIG_FILE`
5. 写入配置文件：
   - `make linux-update-defconfig`
   - `make busybox-update-config`
   - `make uclibc-update-config`
   - `cp <output>/build/at91bootstrap3-*/.config board/<manufacturer>/<boardname>/at91bootstrap3.config`
   - `make barebox-update-defconfig`
   - `make uboot-update-defconfig`
6. 创建 `board/<manufacturer>/<boardname>/rootfs-overlay/`，将 rootfs 需要的额外文件放入，如 `board/<manufacturer>/<boardname>/rootfs-overlay/etc/inittab`。将 `BR2_ROOTFS_OVERLAY` 设为 `board/<manufacturer>/<boardname>/rootfs-overlay`。
7. 创建 post-build 脚本 `board/<manufacturer>/<boardname>/post_build.sh`，将 `BR2_ROOTFS_POST_BUILD_SCRIPT` 设为该脚本路径。
8. 如需设置 setuid 权限或创建设备节点，创建 `board/<manufacturer>/<boardname>/device_table.txt`，并将其路径加入 `BR2_ROOTFS_DEVICE_TABLE`。
9. 如需创建用户账户，创建 `board/<manufacturer>/<boardname>/users_table.txt`，并将其路径加入 `BR2_ROOTFS_USERS_TABLES`。
10. 若需为某些软件包添加自定义补丁，将 `BR2_GLOBAL_PATCH_DIR` 设为 `board/<manufacturer>/<boardname>/patches/`，并为每个软件包在以包名命名的子目录下添加补丁。每个补丁建议命名为 `<packagename>-<num>-<description>.patch`。
11. 针对 Linux 内核，也可用 `BR2_LINUX_KERNEL_PATCH` 选项（可从 URL 下载补丁），如无特殊需求，优先用 `BR2_GLOBAL_PATCH_DIR`。U-Boot、Barebox、at91bootstrap 和 at91bootstrap3 也有类似选项，但不如 `BR2_GLOBAL_PATCH_DIR` 统一，未来可能移除。
12. 如需添加项目专属软件包，创建 `package/<manufacturer>/`，将软件包放入该目录。创建 `<manufacturer>.mk` 文件包含所有 `.mk` 文件，创建 `Config.in` 文件依次 source 所有 `Config.in`，并将其包含到 Buildroot 的 `package/Config.in`。
13. `make savedefconfig` 保存 buildroot 配置。
14. `cp defconfig configs/<boardname>_defconfig`
