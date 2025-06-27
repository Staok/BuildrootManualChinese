### 8.13.4 下载包的位置

Buildroot 下载的各类 tar 包均存储在 `BR2_DL_DIR`，默认是 `dl` 目录。如需保留一份与 tar 包配套、可复现的 Buildroot 版本，可直接复制该目录。这样可确保工具链和目标文件系统可用完全相同的源码重建。

如维护多个 Buildroot 树，建议用共享下载目录。可将 `BR2_DL_DIR` 环境变量指向该目录，此时会覆盖 Buildroot 配置中的 `BR2_DL_DIR`。可在 `<~/.bashrc>` 添加：

```
 export BR2_DL_DIR=<shared download location>
```

下载目录也可在 `.config` 文件用 `BR2_DL_DIR` 选项设置，但该值会被环境变量覆盖。

### 8.13.5 包专用 make 目标

运行 `make <package>` 会构建并安装该包及其依赖。

依赖 Buildroot 基础设施的软件包有许多专用 make 目标，可独立调用：

```
make <package>-<target>
```

包构建目标（按执行顺序）：

| 命令/目标         | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| `source`          | 获取源码（下载 tar 包、克隆源码仓库等）                      |
| `depends`         | 构建并安装构建该包所需的所有依赖                              |
| `extract`         | 将源码放入包构建目录（解包、复制源码等）                      |
| `patch`           | 应用补丁（如有）                                             |
| `configure`       | 运行配置命令（如有）                                         |
| `build`           | 运行编译命令                                                 |
| `install-staging` | ***目标包：*** 安装到 staging 目录（如有必要）                |
| `install-target`  | ***目标包：*** 安装到 target 目录（如有必要）                 |
| `install`         | ***目标包：*** 运行前两步安装命令；***主机包：*** 安装到 host 目录 |

其他常用 make 目标：

| 命令/目标                  | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `show-depends`             | 显示构建该包所需的一阶依赖                                   |
| `show-recursive-depends`   | 递归显示构建该包所需的所有依赖                               |
| `show-rdepends`            | 显示直接依赖该包的包（一阶反向依赖）                         |
| `show-recursive-rdepends`  | 递归显示所有依赖该包的包（直接或间接）                       |
| `graph-depends`            | 生成该包的依赖关系图，详见[依赖关系图章节](https://buildroot.org/downloads/manual/manual.html#graph-depends) |
| `graph-rdepends`           | 生成该包的反向依赖关系图                                     |
| `graph-both-depends`       | 生成该包的正反向依赖关系图                                   |
| `dirclean`                 | 删除整个包的构建目录                                         |
| `reinstall`                | 重新运行安装命令                                             |
| `rebuild`                  | 重新运行编译命令（仅适用于 OVERRIDE_SRCDIR 或直接修改构建目录源码） |
| `reconfigure`              | 重新运行配置命令并重编译（仅适用于 OVERRIDE_SRCDIR 或直接修改构建目录源码） |

### 8.13.6 开发阶段使用 Buildroot

Buildroot 正常流程为：下载 tar 包、解包、配置、编译、安装。源码解包到 `output/build/<package>-<version>`，该目录为临时目录，`make clean` 时会被删除并在下次 `make` 时重建。即使源码为 Git 或 SVN 仓库，Buildroot 也会先打包再解包，流程一致。

此流程适合 Buildroot 作为集成工具使用。但如需开发某组件，直接在 `output/build/<package>-<version>` 修改源码并不合适，因为 `make clean` 会删除该目录。

为此，Buildroot 提供 `<pkg>_OVERRIDE_SRCDIR` 机制。Buildroot 会读取 override 文件，允许用户指定某些包的源码路径。

override 文件默认位置为 `$(CONFIG_DIR)/local.mk`，由 `BR2_PACKAGE_OVERRIDE_FILE` 配置。`$(CONFIG_DIR)` 为 Buildroot `.config` 所在目录，即：

- 树内构建（未用 O=）时为 Buildroot 源码主目录
- 树外构建（用 O=）时为输出目录

如需自定义 override 文件位置，可用 `BR2_PACKAGE_OVERRIDE_FILE` 配置。

override 文件内容格式：

```
<pkg1>_OVERRIDE_SRCDIR = /path/to/pkg1/sources
<pkg2>_OVERRIDE_SRCDIR = /path/to/pkg2/sources
```

示例：

```
LINUX_OVERRIDE_SRCDIR = /home/bob/linux/
BUSYBOX_OVERRIDE_SRCDIR = /home/bob/busybox/
```

如为某包定义了 `<pkg>_OVERRIDE_SRCDIR`，Buildroot 不再下载、解包、打补丁，而是直接用指定目录源码，`make clean` 也不会影响该目录。这样可将 Buildroot 指向你用 Git、SVN 等管理的源码目录。Buildroot 会用 *rsync* 将 `<pkg>_OVERRIDE_SRCDIR` 下源码同步到 `output/build/<package>-custom/`。

此机制建议结合 `make <pkg>-rebuild` 和 `make <pkg>-reconfigure` 使用。`make <pkg>-rebuild all` 会用 rsync 同步源码（仅同步变更文件），并重启该包的构建流程。

如上例，开发者可在 `/home/bob/linux` 修改源码后运行：

```
make linux-rebuild all
```

几秒后即可在 `output/images` 得到更新的内核镜像。同理，修改 `/home/bob/busybox` 后：

```
make busybox-rebuild all
```

`output/images` 下的根文件系统镜像即包含更新后的 BusyBox。
