## 9.5 定制生成的目标文件系统

除了通过 `make *config` 更改配置外，还有其他几种方式可以定制生成的目标文件系统。

推荐的两种方法（可共存）是 root 文件系统覆盖（overlay）和 post build 脚本。

- Root 文件系统覆盖（`BR2_ROOTFS_OVERLAY`）

  文件系统覆盖是一棵文件树，会在目标文件系统构建完成后直接拷贝到目标文件系统上。启用该功能需在“System configuration”菜单中设置 `BR2_ROOTFS_OVERLAY` 配置项为 overlay 根目录。你甚至可以指定多个 overlay，使用空格分隔。若指定相对路径，则相对于 Buildroot 根目录。版本控制系统的隐藏目录（如 `.git`、`.svn`、`.hg` 等）、名为 `.empty` 的文件和以 `~` 结尾的文件不会被拷贝。当启用 `BR2_ROOTFS_MERGED_USR` 时，overlay 不能包含 */bin*、*/lib* 或 */sbin* 目录，因为 Buildroot 会将它们创建为指向 */usr* 下相关文件夹的符号链接。此时，若 overlay 有程序或库，应放在 */usr/bin*、*/usr/sbin* 和 */usr/lib*。如[9.1节“推荐的目录结构”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)所示，推荐 overlay 路径为 `board/<company>/<boardname>/rootfs-overlay`。

- Post-build 脚本（`BR2_ROOTFS_POST_BUILD_SCRIPT`）

  Post-build 脚本是在 Buildroot 构建所有选定软件后、生成 rootfs 镜像前调用的 shell 脚本。启用该功能需在“System configuration”菜单中将 `BR2_ROOTFS_POST_BUILD_SCRIPT` 配置项设置为 post-build 脚本列表（空格分隔）。若指定相对路径，则相对于 Buildroot 根目录。通过 post-build 脚本可以删除或修改目标文件系统中的任意文件，但应谨慎使用。如果某个软件包生成了错误或不需要的文件，建议修复该软件包，而不是用 post-build 脚本清理。如[9.1节“推荐的目录结构”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)所示，推荐脚本路径为 `board/<company>/<boardname>/post_build.sh`。Post-build 脚本运行时，当前工作目录为 Buildroot 主目录，目标文件系统路径作为第一个参数传递。如果 `BR2_ROOTFS_POST_SCRIPT_ARGS` 不为空，这些参数也会传递给脚本。所有脚本接收相同参数，无法为每个脚本单独设置参数。`注意：+BR2_ROOTFS_POST_SCRIPT_ARGS+ 的参数也会传递给 post-image 和 post-fakeroot 脚本。如果只想为 post-build 脚本使用特定参数，可用 +BR2_ROOTFS_POST_BUILD_SCRIPT_ARGS+。`

此外，你还可以使用如下环境变量：

- `BR2_CONFIG`：Buildroot .config 文件路径
- `CONFIG_DIR`：包含 .config 文件的目录，即顶层 Buildroot Makefile 所在目录（适用于树内和树外构建）
- `HOST_DIR`、`STAGING_DIR`、`TARGET_DIR`：见[18.6.2节“generic-package 参考”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)
- `BUILD_DIR`：软件包解压和构建目录
- `BINARIES_DIR`：所有二进制文件（镜像）存放目录
- `BASE_DIR`：输出基础目录
- `PARALLEL_JOBS`：并行进程数

下述三种定制目标文件系统的方法不推荐使用：

- 直接修改目标文件系统

  临时修改时，可以直接修改 `output/target/` 下的目标文件系统，然后运行 `make` 重新生成镜像。此方法可对目标文件系统做任意更改，但如果执行 `make clean`，这些更改会丢失。某些情况下需要清理 Buildroot 树，详见[8.2节“何时需要完全重建”](https://buildroot.org/downloads/manual/manual.html#full-rebuild)。因此该方法仅适用于快速测试：*更改不会在 `make clean` 后保留*。验证更改后，应通过 overlay 或 post-build 脚本使其持久化。

- 自定义目标 skeleton（`BR2_ROOTFS_SKELETON_CUSTOM`）

  根文件系统镜像是从目标 skeleton 创建的，所有软件包都在其上安装文件。skeleton 会在构建和安装任何软件包前复制到 `output/target` 目录。默认 skeleton 提供标准 Unix 文件系统布局和基础 init 脚本及配置文件。如果默认 skeleton（位于 `system/skeleton`）不满足需求，通常用 overlay 或 post-build 脚本调整即可。但如果默认 skeleton 与需求完全不同，可以使用自定义 skeleton。启用该功能需设置 `BR2_ROOTFS_SKELETON_CUSTOM` 并将 `BR2_ROOTFS_SKELETON_CUSTOM_PATH` 设为自定义 skeleton 路径，均在“System configuration”菜单中。若指定相对路径，则相对于 Buildroot 根目录。自定义 skeleton 不需要包含 */bin*、*/lib* 或 */sbin* 目录，这些会在构建时自动创建。启用 `BR2_ROOTFS_MERGED_USR` 时，自定义 skeleton 也不能包含这些目录，此时程序或库应放在 */usr/bin*、*/usr/sbin* 和 */usr/lib*。该方法不推荐，因为会复制整个 skeleton，无法利用 Buildroot 后续版本对默认 skeleton 的修复和改进。

- Post-fakeroot 脚本（`BR2_ROOTFS_POST_FAKEROOT_SCRIPT`）

  在聚合最终镜像时，部分过程需要 root 权限（如在 `/dev` 创建设备节点、设置文件和目录权限/所有权等）。为避免实际 root 权限，Buildroot 使用 `fakeroot` 模拟 root 权限。虽然不完全等同于 root，但已满足 Buildroot 需求。Post-fakeroot 脚本是在 fakeroot 阶段*结束时*、文件系统镜像生成器调用*前*执行的 shell 脚本，因此在 fakeroot 环境下运行。若需以 root 用户权限修改文件系统，可用 post-fakeroot 脚本。**注意：** 推荐使用现有机制设置文件权限或在 `/dev` 创建节点（见[9.5.1节“设置文件权限和所有权及添加自定义设备节点”](https://buildroot.org/downloads/manual/manual.html#customize-device-permission)），或创建用户（见[9.6节“添加自定义用户账户”](https://buildroot.org/downloads/manual/manual.html#customize-users)）。**注意：** post-build 脚本和 fakeroot 脚本的区别在于，post-build 脚本不在 fakeroot 环境下执行。**注意：** `fakeroot` 仅模拟文件访问权限、类型（常规、块/字符设备等）和 uid/gid，这些都是内存中模拟的。
