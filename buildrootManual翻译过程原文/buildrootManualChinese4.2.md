## 8.13 高级用法

### 8.13.1 在 Buildroot 外部使用生成的工具链

你可能希望为目标设备编译 Buildroot 未打包的软件或自有程序。此时可直接使用 Buildroot 生成的工具链。

Buildroot 生成的工具链默认位于 `output/host/`。最简单的用法是将 `output/host/bin/` 加入 PATH，然后使用 `ARCH-linux-gcc`、`ARCH-linux-objdump`、`ARCH-linux-ld` 等。

另外，Buildroot 也可通过 `make sdk` 导出工具链及所有已选包的开发文件，生成 SDK。该命令会将 `output/host/` 内容打包为 `<TARGET-TUPLE>_sdk-buildroot.tar.gz`（可通过环境变量 `BR2_SDK_PREFIX` 覆盖），存放于 `output/images/`。

该 tar 包可分发给应用开发者，用于开发尚未打包为 Buildroot 包的应用。

解压 SDK tar 包后，用户需运行 SDK 顶层目录下的 `relocate-sdk.sh` 脚本，以更新所有路径。

如只需准备 SDK 而不打包（如仅移动 `host` 目录或自行打包），可用 `make prepare-sdk`。

如启用 `BR2_PACKAGE_HOST_ENVIRONMENT_SETUP`，`output/host/` 及 SDK 中会安装 `environment-setup` 脚本。可用 `. your/sdk/path/environment-setup` 导出一系列环境变量，便于用 Buildroot SDK 交叉编译项目：`PATH` 会包含 SDK 二进制，标准 autotools 变量会自动设置，`CONFIGURE_FLAGS` 包含 autotools 项目交叉编译的基本 `./configure` 选项，还提供部分实用命令。注意：sourcing 该脚本后，环境仅适用于交叉编译，不再适用于本地编译。

### 8.13.2 在 Buildroot 中使用 `gdb`

Buildroot 支持交叉调试：调试器运行在构建机，通过 `gdbserver` 与目标设备通信，控制程序执行。

操作步骤：

- 如用*内部工具链*（Buildroot 构建），需启用 `BR2_PACKAGE_HOST_GDB`、`BR2_PACKAGE_GDB` 和 `BR2_PACKAGE_GDB_SERVER`，确保交叉 gdb 和 gdbserver 均被构建，gdbserver 安装到目标设备。
- 如用*外部工具链*，应启用 `BR2_TOOLCHAIN_EXTERNAL_GDB_SERVER_COPY`，将外部工具链自带的 gdbserver 拷贝到目标。如外部工具链无交叉 gdb 或 gdbserver，也可让 Buildroot 构建，方法同内部工具链。

调试名为 `foo` 的程序时，目标设备运行：

```
gdbserver :2345 foo
```

此时 `gdbserver` 监听 2345 端口，等待交叉 gdb 连接。

主机端启动交叉 gdb：

```
<buildroot>/output/host/bin/<tuple>-gdb -ix <buildroot>/output/staging/usr/share/buildroot/gdbinit foo
```

注意：`foo` 需带调试符号，建议在其构建目录下运行该命令（`output/target/` 下的二进制已剥离）。

`<buildroot>/output/staging/usr/share/buildroot/gdbinit` 文件会告知交叉 gdb 目标库文件位置。

最后，在交叉 gdb 中连接目标：

```
(gdb) target remote <target ip address>:2345
```

### 8.13.3 在 Buildroot 中使用 `ccache`

[ccache](http://ccache.samba.org/) 是编译器缓存工具。它缓存每次编译生成的目标文件，后续相同源码（编译器及参数一致）可直接复用缓存，加快多次相似全量构建。

Buildroot 集成了 `ccache` 支持。只需在 `Build options` 启用“Enable compiler cache”，即可自动构建并为所有主机和目标编译启用 `ccache`。

缓存目录由 `BR2_CCACHE_DIR` 配置项指定，默认 `$HOME/.buildroot-ccache`。该目录位于 Buildroot 输出目录之外，便于多个 Buildroot 构建共享。如需清理缓存，直接删除该目录即可。

可用 `make ccache-stats` 查看缓存统计（大小、命中/未命中等）。

`ccache-options` make 目标和 `CCACHE_OPTIONS` 变量可用于通用 ccache 操作。例如：

```
# 设置缓存大小上限
make CCACHE_OPTIONS="--max-size=5G" ccache-options

# 清零统计计数器
make CCACHE_OPTIONS="--zero-stats" ccache-options
```

`ccache` 会对源码和编译参数做哈希。若编译参数不同，则不会复用缓存。许多编译参数包含绝对路径（如 staging 目录），因此更换输出目录会导致大量缓存未命中。

为避免此问题，Buildroot 提供“Use relative paths”选项（`BR2_CCACHE_USE_BASEDIR`），会将输出目录内的绝对路径重写为相对路径，从而更换输出目录时不影响缓存命中。

缺点是目标文件中的路径也变为相对路径，调试时需先切换到输出目录。

详见 [ccache 手册“在不同目录编译”章节](https://ccache.samba.org/manual.html#_compiling_in_different_directories)。

启用 `BR2_CCACHE=y` 后：

- Buildroot 构建过程会用 `ccache`
- 在 Buildroot 外部（如直接调用交叉编译器或用 SDK）不会用 `ccache`

可用 `BR2_USE_CCACHE` 环境变量覆盖此行为：设为 `1` 时启用（Buildroot 构建默认），未设或非 `1` 时禁用。
