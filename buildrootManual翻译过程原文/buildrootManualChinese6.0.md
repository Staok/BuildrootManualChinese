## 9.3 保存 Buildroot 配置

可以使用命令 `make savedefconfig` 保存 Buildroot 配置。

该命令会去除所有为默认值的配置选项，仅保留实际更改的内容，结果存储在名为 `defconfig` 的文件中。如需保存到其他位置，可在 Buildroot 配置中更改 `BR2_DEFCONFIG` 选项，或用 `make savedefconfig BR2_DEFCONFIG=<path-to-defconfig>` 指定路径。

推荐将 defconfig 文件存放在 `configs/<boardname>_defconfig`。这样配置会被 `make list-defconfigs` 列出，并可通过 `make <boardname>_defconfig` 恢复。

当然，也可以将该文件复制到任意位置，并用 `make defconfig BR2_DEFCONFIG=<path-to-defconfig-file>` 重新构建。

## 9.4 保存其他组件的配置

如对 BusyBox、Linux 内核、Barebox、U-Boot 和 uClibc 做了配置更改，也应保存其配置文件。Buildroot 为每个组件都提供了配置项用于指定输入配置文件路径，例如 `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE`。要保存这些配置文件，先将配置项指向你希望保存的位置，然后用下述辅助目标实际保存配置。

如[第 9.1 节 推荐目录结构](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)所述，推荐将这些配置文件存放在 `board/<company>/<boardname>/foo.config`。

注意：请在更改 `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE` 等选项*之前*先创建配置文件，否则 Buildroot 会尝试访问尚不存在的配置文件并报错。可通过 `make linux-menuconfig` 等命令创建配置文件。

Buildroot 提供了若干辅助目标，便于保存配置文件：

- `make linux-update-defconfig`：将 Linux 配置保存到 `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE` 指定路径，并去除默认值（仅适用于 2.6.33 及以上内核，早期内核用 `make linux-update-config`）。
- `make busybox-update-config`：将 BusyBox 配置保存到 `BR2_PACKAGE_BUSYBOX_CONFIG` 指定路径。
- `make uclibc-update-config`：将 uClibc 配置保存到 `BR2_UCLIBC_CONFIG` 指定路径。
- `make barebox-update-defconfig`：将 Barebox 配置保存到 `BR2_TARGET_BAREBOX_CUSTOM_CONFIG_FILE` 指定路径。
- `make uboot-update-defconfig`：将 U-Boot 配置保存到 `BR2_TARGET_UBOOT_CUSTOM_CONFIG_FILE` 指定路径。
- 对于 at91bootstrap3，没有辅助命令，需手动将配置文件复制到 `BR2_TARGET_AT91BOOTSTRAP3_CUSTOM_CONFIG_FILE`。
