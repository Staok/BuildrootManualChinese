## 8.5 树外构建（out-of-tree build）

默认情况下，Buildroot 构建的所有内容都存储在 Buildroot 树下的 `output` 目录。

Buildroot 也支持类似 Linux 内核的树外构建。用法是在 make 命令行加 `O=<目录>`：

```
 $ make O=/tmp/build menuconfig
```

或：

```
 $ cd /tmp/build; make O=$PWD -C path/to/buildroot menuconfig
```

所有输出文件都位于 `/tmp/build`。如 `O` 路径不存在，Buildroot 会自动创建。

***注意：*** `O` 路径可为绝对或相对路径，但如为相对路径，则相对于 Buildroot 源码主目录（而非当前工作目录）。

使用树外构建时，Buildroot 的 `.config` 和临时文件也存储在输出目录。这样可在同一源码树下并行运行多个构建，只需保证输出目录唯一。

为方便使用，Buildroot 会在输出目录生成 Makefile 包装器——首次运行后，无需再传递 `O=<…>` 和 `-C <…>`，只需在输出目录下直接运行：

```
 $ make <target>
```

## 8.6 环境变量

Buildroot 支持部分环境变量，可在调用 `make` 时传递或在环境中设置：

- `HOSTCXX`：主机 C++ 编译器
- `HOSTCC`：主机 C 编译器
- `UCLIBC_CONFIG_FILE=<path/to/.config>`：uClibc 配置文件路径（如构建内部工具链）。uClibc 配置文件也可在 Buildroot 配置界面设置，推荐用 `.config` 文件方式。
- `BUSYBOX_CONFIG_FILE=<path/to/.config>`：BusyBox 配置文件路径。也可在 Buildroot 配置界面设置，推荐用 `.config` 文件方式。
- `BR2_CCACHE_DIR`：覆盖 Buildroot 使用 ccache 时缓存文件的目录。
- `BR2_DL_DIR`：覆盖 Buildroot 下载/检索文件的目录。下载目录也可在 Buildroot 配置界面设置，推荐用 `.config` 文件方式。详见[第 8.13.4 节 下载包位置](https://buildroot.org/downloads/manual/manual.html#download-location)。
- `BR2_GRAPH_ALT`：如设置且非空，构建时图表使用备用配色方案。
- `BR2_GRAPH_OUT`：设置生成图表的文件类型，`pdf`（默认）或 `png`。
- `BR2_GRAPH_DEPS_OPTS`：为依赖关系图传递额外选项，详见[第 8.9 节 包依赖关系图](https://buildroot.org/downloads/manual/manual.html#graph-depends)。
- `BR2_GRAPH_DOT_OPTS`：原样传递给 `dot` 工具以绘制依赖关系图。
- `BR2_GRAPH_SIZE_OPTS`：为大小图传递额外选项，详见[第 8.11 节 文件系统大小图](https://buildroot.org/downloads/manual/manual.html#graph-size)。

示例：配置文件分别位于顶层目录和 $HOME：

```
 $ make UCLIBC_CONFIG_FILE=uClibc.config BUSYBOX_CONFIG_FILE=$HOME/bb.config
```

如需为主机端辅助二进制选择非默认 `gcc` 或 `g++` 编译器：

```
 $ make HOSTCXX=g++-4.3-HEAD HOSTCC=gcc-4.3-HEAD
```

## 8.7 高效处理文件系统镜像

文件系统镜像体积可能很大，取决于文件系统类型、软件包数量、预留空间等。但镜像中部分区域可能只是*空*（如长串零），这种文件称为*稀疏文件（sparse file）*。

大多数工具可高效处理稀疏文件，仅存储/写入非零部分。

例如：

- `tar` 支持 `-S` 选项，仅存储稀疏文件的非零块：
  - `tar cf archive.tar -S [files…]` 可高效打包稀疏文件
  - `tar xf archive.tar -S` 可高效解包稀疏文件
- `cp` 支持 `--sparse=WHEN` 选项（`WHEN` 可为 `auto`、`never` 或 `always`）：
  - `cp --sparse=always source.file dest.file` 如源文件有长零区，目标文件也会为稀疏文件

其他工具也有类似选项，详见各自 man 手册。

如需存储、传输文件系统镜像（如跨机传输或发给 QA），可用稀疏文件。

但注意：用 `dd` 的稀疏模式刷写镜像到设备可能导致文件系统损坏（如 ext2 的块位图损坏，或稀疏文件部分读回时非全零）。稀疏文件仅适用于构建机本地操作，不应用于实际设备刷写。
