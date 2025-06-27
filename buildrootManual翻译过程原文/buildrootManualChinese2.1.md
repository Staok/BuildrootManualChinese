### 6.1.3 使用 Buildroot 构建外部工具链

Buildroot 的内部工具链选项可用于创建外部工具链。以下是构建内部工具链并将其打包以供 Buildroot（或其他项目）复用的步骤：

新建 Buildroot 配置，主要设置如下：

- 在 ***目标选项（Target options）*** 中选择合适的目标 CPU 架构；
- 在 ***工具链（Toolchain）*** 菜单中，保持 ***工具链类型（Toolchain type）*** 为默认的 ***Buildroot toolchain***，并按需配置工具链；
- 在 ***系统配置（System configuration）*** 菜单中，将 ***Init system*** 设为 ***None***，将 *** /bin/sh *** 设为 ***none***；
- 在 ***目标软件包（Target packages）*** 菜单中，禁用 ***BusyBox***；
- 在 ***文件系统镜像（Filesystem images）*** 菜单中，禁用 ***tar the root filesystem***。

然后，触发构建并让 Buildroot 生成 SDK，这会为你生成一个包含工具链的 tar 包：

```
make sdk
```

SDK tar 包会生成在 `$(O)/images` 目录下，文件名类似于 `arm-buildroot-linux-uclibcgnueabi_sdk-buildroot.tar.gz`。保存该 tar 包，即可在其他 Buildroot 项目中作为外部工具链复用。

在其他 Buildroot 项目中，在 ***工具链（Toolchain）*** 菜单：

- 设置 ***工具链类型（Toolchain type）*** 为 ***External toolchain***；
- 设置 ***Toolchain*** 为 ***Custom toolchain***；
- 设置 ***Toolchain origin*** 为 ***Toolchain to be downloaded and installed***；
- 设置 ***Toolchain URL*** 为 `file:///path/to/your/sdk/tarball.tar.gz`。

#### 外部工具链包装器（External toolchain wrapper）

使用外部工具链时，Buildroot 会生成一个包装程序（wrapper），自动为外部工具链程序传递合适参数（根据配置）。如需调试该包装器以检查实际传递的参数，可设置环境变量 `BR2_DEBUG_WRAPPER`，可选值如下：

- `0`、空或未设置：无调试输出；
- `1`：所有参数输出在一行；
- `2`：每行输出一个参数。

## 6.2 /dev 管理

在 Linux 系统中，`/dev` 目录包含特殊文件，称为*设备文件（device files）*，用于让用户空间应用访问由内核管理的硬件设备。没有这些*设备文件*，即使内核已识别硬件，用户空间应用也无法使用。

在 `System configuration` 的 `/dev management` 下，Buildroot 提供四种 `/dev` 目录管理方案：

- 第一种方案是 ***静态设备表（Static using device table）***。这是 Linux 早期管理设备文件的传统方式。设备文件持久存储在根文件系统中（重启后依然存在），不会因硬件增减自动创建或删除。Buildroot 会用*设备表（device table）*创建一组标准设备文件，默认表位于 Buildroot 源码的 `system/device_table_dev.txt`。该文件在生成最终根文件系统镜像时处理，因此*设备文件*不会出现在 `output/target` 目录。`BR2_ROOTFS_STATIC_DEVICE_TABLE` 选项可更改默认设备表或添加额外设备表，以便在构建时创建更多*设备文件*。如需添加设备文件，可新建 `board/<yourcompany>/<yourproject>/device_table_dev.txt`，并将 `BR2_ROOTFS_STATIC_DEVICE_TABLE` 设为 `system/device_table_dev.txt board/<yourcompany>/<yourproject>/device_table_dev.txt`。设备表格式详见[第 25 章 Makedev 语法文档](https://buildroot.org/downloads/manual/manual.html#makedev-syntax)。
- 第二种方案是 ***仅用 devtmpfs 动态管理（Dynamic using devtmpfs only）***。*devtmpfs* 是内核 2.6.32 引入的虚拟文件系统（如用旧内核无法用此选项）。挂载到 `/dev` 后，内核会根据硬件增减自动动态创建和删除*设备文件*，但不会持久化。需启用内核配置 `CONFIG_DEVTMPFS` 和 `CONFIG_DEVTMPFS_MOUNT`。如 Buildroot 负责构建内核，会自动启用这两个选项；如在 Buildroot 外构建内核，需自行确保启用，否则 Buildroot 系统无法启动。
- 第三种方案是 ***devtmpfs + mdev 动态管理（Dynamic using devtmpfs + mdev）***。此法同样依赖上述 *devtmpfs*（同样需启用 `CONFIG_DEVTMPFS` 和 `CONFIG_DEVTMPFS_MOUNT`），但在其基础上增加了 BusyBox 的 `mdev` 用户空间工具。每当设备增减，内核会调用 `mdev`，其行为可通过 `/etc/mdev.conf` 配置，如设置设备文件权限、归属、调用脚本等。这样，*用户空间*可对设备事件做出反应。`mdev` 还可用于设备出现时自动加载内核模块，或为需固件的设备推送固件。`mdev` 是 `udev` 的轻量实现。详见 http://git.busybox.net/busybox/tree/docs/mdev.txt。
- 第四种方案是 ***devtmpfs + eudev 动态管理（Dynamic using devtmpfs + eudev）***。同样依赖 *devtmpfs*，但在其上运行 `eudev` 守护进程。`eudev` 是后台运行的守护进程，设备增减时由内核调用。比 `mdev` 更重，但更灵活。`eudev` 是 `udev` 的独立版本，`udev` 原为大多数桌面 Linux 发行版的用户空间守护进程，现已并入 Systemd。详见 http://en.wikipedia.org/wiki/Udev。

Buildroot 开发者建议优先使用 ***仅用 devtmpfs 动态管理（Dynamic using devtmpfs only）***，如需用户空间响应设备事件或需固件支持，则推荐 ***devtmpfs + mdev***。

注意：如选择 `systemd` 作为 init system，则 /dev 管理由 `systemd` 提供的 `udev` 完成。

## 6.3 init 系统

*init* 程序是内核启动的第一个用户空间程序（PID 为 1），负责启动用户空间服务和程序（如 Web 服务器、图形应用、网络服务等）。

Buildroot 支持三种 init 系统，可在 `System configuration` 的 `Init system` 选择：

- 第一种方案是 ***BusyBox***。BusyBox 除众多工具外，还实现了基本的 `init` 程序，足以满足大多数嵌入式系统。启用 `BR2_INIT_BUSYBOX` 后，BusyBox 会构建并安装其 `init` 程序（Buildroot 默认方案）。BusyBox 的 `init` 程序启动时读取 `/etc/inittab` 文件。该文件语法见 http://git.busybox.net/busybox/tree/examples/inittab（注意 BusyBox 的 inittab 语法特殊，请勿参考网络上其他 inittab 文档）。Buildroot 默认的 inittab 位于 `package/busybox/inittab`。除挂载重要文件系统外，默认 inittab 主要启动 `/etc/init.d/rcS` 脚本和 `getty`（登录提示）。
- 第二种方案是 ***systemV***。采用传统的 *sysvinit* 程序，Buildroot 打包于 `package/sysvinit`。该方案曾为大多数桌面 Linux 发行版所用，后被 Upstart、Systemd 等替代。`sysvinit` 也用 inittab 文件（语法与 BusyBox 略有不同），默认 inittab 位于 `package/sysvinit/inittab`。
- 第三种方案是 ***systemd***。`systemd` 是新一代 Linux init 系统，功能远超传统 *init*：支持激进的并行启动、基于 socket 和 D-Bus 的服务激活、按需启动守护进程、用 Linux 控制组跟踪进程、支持快照和恢复等。`systemd` 适用于较复杂的嵌入式系统（如需 D-Bus 及服务间通信）。需注意，`systemd` 依赖较多（如 `dbus`、`udev` 等）。详见 http://www.freedesktop.org/wiki/Software/systemd。

Buildroot 开发者推荐使用 ***BusyBox init***，足以满足大多数嵌入式系统。如需更复杂功能可选用 ***systemd***。

------

[3] 该术语与 GNU configure 不同，后者将 host 定义为应用实际运行的机器（通常即 target）。
