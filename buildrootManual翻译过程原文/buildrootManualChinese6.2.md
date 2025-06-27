### 9.5.1 设置文件权限和所有权及添加自定义设备节点

有时需要为文件或设备节点设置特定权限或所有权。例如，某些文件可能需要归 root 所有。由于 post-build 脚本不是以 root 身份运行，除非在脚本中显式使用 fakeroot，否则无法完成这些更改。

为此，Buildroot 提供了*权限表*（permission tables）功能。启用该功能需将 `BR2_ROOTFS_DEVICE_TABLE` 配置项设置为一个或多个权限表（空格分隔），这些表是遵循 [makedev 语法](https://buildroot.org/downloads/manual/manual.html#makedev-syntax)的普通文本文件。

如果你使用静态设备表（即未使用 `devtmpfs`、`mdev` 或 `(e)udev`），则可以用相同语法在*设备表*中添加设备节点。启用该功能需将 `BR2_ROOTFS_STATIC_DEVICE_TABLE` 配置项设置为一个或多个设备表（空格分隔）。

如[9.1节“推荐的目录结构”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)所述，推荐将这些文件放在 `board/<company>/<boardname>/`。

需要注意的是，如果特定权限或设备节点与某个应用相关，建议在该软件包的 `.mk` 文件中设置 `FOO_PERMISSIONS` 和 `FOO_DEVICES` 变量（见[18.6.2节“generic-package 参考”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)）。

## 9.6 添加自定义用户账户

有时需要在目标系统中添加特定用户。为此，Buildroot 提供了*用户表*（users tables）功能。启用该功能需将 `BR2_ROOTFS_USERS_TABLES` 配置项设置为一个或多个用户表（空格分隔），这些表是遵循 [makeusers 语法](https://buildroot.org/downloads/manual/manual.html#makeuser-syntax)的普通文本文件。

如[9.1节“推荐的目录结构”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)所述，推荐将这些文件放在 `board/<company>/<boardname>/`。

需要注意的是，如果自定义用户与某个应用相关，建议在该软件包的 `.mk` 文件中设置 `FOO_USERS` 变量（见[18.6.2节“generic-package 参考”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)）。

## 9.7 镜像生成后的定制

post-build 脚本（见[9.5节“定制生成的目标文件系统”](https://buildroot.org/downloads/manual/manual.html#rootfs-custom)）是在构建文件系统镜像、内核和引导加载程序*之前*运行的，***post-image 脚本***可用于在所有镜像生成*之后*执行特定操作。

post-image 脚本可用于自动将 root 文件系统 tar 包解压到 NFS 服务器导出的目录，或创建包含 root 文件系统和内核镜像的特殊固件镜像，或执行项目所需的其他自定义操作。

启用该功能需在“System configuration”菜单中将 `BR2_ROOTFS_POST_IMAGE_SCRIPT` 配置项设置为 post-image 脚本列表（空格分隔）。若指定相对路径，则相对于 Buildroot 根目录。

与 post-build 脚本类似，post-image 脚本运行时当前工作目录为 Buildroot 主目录，`images` 输出目录路径作为第一个参数传递。如果 `BR2_ROOTFS_POST_SCRIPT_ARGS` 不为空，这些参数也会传递给脚本。所有脚本接收相同参数，无法为每个脚本单独设置参数。

注意，`BR2_ROOTFS_POST_SCRIPT_ARGS` 的参数也会传递给 post-build 和 post-fakeroot 脚本。如果只想为 post-image 脚本使用特定参数，可用 `BR2_ROOTFS_POST_IMAGE_SCRIPT_ARGS`。

同样，post-image 脚本可访问环境变量 `BR2_CONFIG`、`HOST_DIR`、`STAGING_DIR`、`TARGET_DIR`、`BUILD_DIR`、`BINARIES_DIR`、`CONFIG_DIR`、`BASE_DIR` 和 `PARALLEL_JOBS`。

post-image 脚本会以执行 Buildroot 的用户身份运行，通常*不应*是 root 用户。因此，若脚本中有需要 root 权限的操作，需要特殊处理（如使用 fakeroot 或 sudo），具体由脚本开发者决定。

## 9.8 添加项目专属补丁和哈希

### 9.8.1 提供额外补丁

有时需要为软件包应用*额外*补丁（在 Buildroot 自带补丁基础上）。例如，为项目支持自定义功能，或开发新架构时。

`BR2_GLOBAL_PATCH_DIR` 配置项可用于指定一个或多个包含软件包补丁的目录（空格分隔）。

对于特定版本 `<packageversion>` 的特定软件包 `<packagename>`，补丁应用顺序如下：

1. 对于 `BR2_GLOBAL_PATCH_DIR` 中的每个目录 `<global-patch-dir>`，确定 `<package-patch-dir>`：
   - 若 `<global-patch-dir>/<packagename>/<packageversion>/` 存在，则使用该目录；
   - 否则，若 `<global-patch-dir>/<packagename>` 存在，则使用该目录。
2. 从 `<package-patch-dir>` 应用补丁：
   - 若目录下有 `series` 文件，则按 `series` 文件顺序应用补丁；
   - 否则，按字母顺序应用所有 `*.patch` 文件。为确保顺序，建议补丁文件命名为 `<number>-<description>.patch`，其中 `<number>` 表示应用顺序。

关于软件包补丁应用顺序，详见[19.2节“补丁应用顺序”](https://buildroot.org/downloads/manual/manual.html#patch-apply-order)。

`BR2_GLOBAL_PATCH_DIR` 是为软件包指定自定义补丁目录的首选方法。它可用于为 buildroot 中的任意软件包指定补丁目录，也应优先于 U-Boot、Barebox 等软件包的自定义补丁目录选项。这样用户可在顶层目录统一管理所有补丁。

唯一例外是 `BR2_LINUX_KERNEL_PATCH`，该选项用于指定可通过 URL 获取的内核补丁。***注意：*** `BR2_LINUX_KERNEL_PATCH` 指定的补丁会在 `BR2_GLOBAL_PATCH_DIR` 补丁之后应用（由 Linux 软件包的 post-patch hook 完成）。
