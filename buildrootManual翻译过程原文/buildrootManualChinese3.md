## 第 7 章 其他组件的配置

在尝试修改下述任何组件前，请确保已配置好 Buildroot 本身，并已启用相应软件包。

- BusyBox

  如果已有 BusyBox 配置文件，可在 Buildroot 配置中通过 `BR2_PACKAGE_BUSYBOX_CONFIG` 直接指定该文件。否则，Buildroot 会从默认 BusyBox 配置文件开始。如需后续修改配置，可用 `make busybox-menuconfig` 打开 BusyBox 配置编辑器。也可通过环境变量指定 BusyBox 配置文件，但不推荐。详见[第 8.6 节 环境变量](https://buildroot.org/downloads/manual/manual.html#env-vars)。

- uClibc

  uClibc 的配置方式与 BusyBox 类似。可用配置变量 `BR2_UCLIBC_CONFIG` 指定已有配置文件，后续修改可用 `make uclibc-menuconfig`。

- Linux 内核

  如已有内核配置文件，可在 Buildroot 配置中通过 `BR2_LINUX_KERNEL_USE_CUSTOM_CONFIG` 直接指定。如无配置文件，可在 Buildroot 配置中用 `BR2_LINUX_KERNEL_USE_DEFCONFIG` 指定 defconfig，或新建空文件并指定为自定义配置文件（`BR2_LINUX_KERNEL_USE_CUSTOM_CONFIG`）。后续修改可用 `make linux-menuconfig`。

- Barebox

  Barebox 的配置方式与 Linux 内核类似。相关配置变量为 `BR2_TARGET_BAREBOX_USE_CUSTOM_CONFIG` 和 `BR2_TARGET_BAREBOX_USE_DEFCONFIG`。配置编辑器命令为 `make barebox-menuconfig`。

- U-Boot

  U-Boot（2015.04 及以上版本）配置方式同 Linux 内核。相关配置变量为 `BR2_TARGET_UBOOT_USE_CUSTOM_CONFIG` 和 `BR2_TARGET_UBOOT_USE_DEFCONFIG`。配置编辑器命令为 `make uboot-menuconfig`。

## 第 8 章 Buildroot 常用用法

### 8.1 *make* 使用技巧

以下是帮助你高效使用 Buildroot 的技巧。

**显示 make 执行的所有命令：**

```
 $ make V=1 <target>
```

**显示带 defconfig 的所有开发板列表：**

```
 $ make list-defconfigs
```

**显示所有可用目标：**

```
 $ make help
```

并非所有目标始终可用，`.config` 文件中的部分设置会隐藏某些目标：

- 仅启用 busybox 时可用 `busybox-menuconfig`；
- 仅启用 linux 时可用 `linux-menuconfig` 和 `linux-savedefconfig`；
- 仅选择内部工具链为 uClibc 时可用 `uclibc-menuconfig`；
- 仅启用 barebox 启动加载器时可用 `barebox-menuconfig` 和 `barebox-savedefconfig`；
- 仅启用 U-Boot 启动加载器且构建系统为 Kconfig 时可用 `uboot-menuconfig` 和 `uboot-savedefconfig`。

**清理：** 如更改体系结构或工具链配置选项，需显式清理。

删除所有构建产物（包括构建目录、host、staging 和 target 树、镜像和工具链）：

```
 $ make clean
```

**生成手册：** 当前手册源码位于 *docs/manual* 目录。生成手册命令：

```
 $ make manual-clean
 $ make manual
```

手册输出位于 *output/docs/manual*。

**注意**

- 构建文档需安装部分工具（见[第 2.2 节 可选软件包](https://buildroot.org/downloads/manual/manual.html#requirement-optional)）。

**为新目标重置 Buildroot：** 删除所有构建产物及配置：

```
 $ make distclean
```

**注意：** 如启用 `ccache`，运行 `make clean` 或 `distclean` 不会清空 Buildroot 使用的编译器缓存。清理方法见[第 8.13.3 节 Buildroot 中使用 ccache](https://buildroot.org/downloads/manual/manual.html#ccache)。

**导出 make 内部变量：** 可导出 make 已知变量及其值：

```
 $ make -s printvars VARS='VARIABLE1 VARIABLE2'
 VARIABLE1=value_of_variable
 VARIABLE2=value_of_variable
```

可通过变量调整输出：
