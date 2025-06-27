## 16.3 `genimage.cfg` 文件

`genimage.cfg` 文件包含 genimage 工具用于创建最终 .img 文件的输出镜像布局。

示例如下：

```
image efi-part.vfat {
        vfat {
                file EFI {
                        image = "efi-part/EFI"
                }

                file Image {
                        image = "Image"
                }
        }

        size = 32M
}

image sdimage.img {
        hdimage {
        }

        partition u-boot {
                image = "efi-part.vfat"
                offset = 8K
        }

        partition root {
                image = "rootfs.ext2"
                size = 512M
        }
}
```

- 每个 `section`（如 hdimage、vfat 等）、`partition` 必须使用一个制表符缩进。
- 每个 `file` 或其他 `subnode` 必须使用两个制表符缩进。
- 每个节点（`section`、`partition`、`file`、`subnode`）的左花括号必须与节点名在同一行，右花括号必须单独成行，且其后需换行（最后一个节点除外）。选项（如 `size`、`=`）同理。
- 每个 `option`（如 `image`、`offset`、`size`）的 `=` 号前后各有一个空格。
- 文件名必须至少以 genimage 前缀开头并以 .cfg 扩展名结尾，以便于识别。
- `offset` 和 `size` 选项允许的单位有：`G`、`M`、`K`（不支持小写 `k`）。如果无法用上述单位精确表示字节数，则使用十六进制 `0x` 前缀，或最后采用字节数。在注释中请使用 `GB`、`MB`、`KB`（不支持小写 `kb`）代替 `G`、`M`、`K`。
- 对于 GPT 分区，`partition-type-uuid` 的值必须为：EFI 系统分区用 `U`（*genimage* 会扩展为 `c12a7328-f81f-11d2-ba4b-00a0c93ec93b`），FAT 分区用 `F`（扩展为 `ebd0a0a2-b9e5-4433-87c0-68b6b72699c7`），根文件系统或其他文件系统用 `L`（扩展为 `0fc63daf-8483-4772-8e79-3d69d8477de4`）。虽然 `L` 是 *genimage* 的默认值，但我们建议在 `genimage.cfg` 文件中显式指定。最后，这些快捷方式应不加引号使用，例如 `partition-type-uuid = U`。如果指定了显式 GUID，字母应为小写。

`genimage.cfg` 文件是 Buildroot 用于生成最终镜像文件（如 sdcard.img）的 genimage 工具的输入文件。关于 *genimage* 语言的更多细节，请参阅 https://github.com/pengutronix/genimage/blob/master/README.rst。

## 16.4 文档

文档采用 [asciidoc](https://asciidoc-py.github.io/) 格式。

关于 asciidoc 语法的更多细节，请参阅 https://asciidoc-py.github.io/userguide.html。

## 16.5 支持脚本

`support/` 和 `utils/` 目录中的部分脚本采用 Python 编写，应遵循 [PEP8 Python 代码风格指南](https://www.python.org/dev/peps/pep-0008/)。

## 第17章 为特定开发板添加支持

Buildroot 为多种公开可用的硬件开发板提供了基础配置，方便这些开发板的用户轻松构建已知可用的系统。你也可以为 Buildroot 添加对其他开发板的支持。

为此，你需要创建一个普通的 Buildroot 配置，用于为硬件构建一个基础系统：包括（内部）工具链、内核、引导加载程序、文件系统和一个仅包含 BusyBox 的简单用户空间。不要选择特定的软件包：配置应尽可能精简，仅为目标平台构建一个可用的 BusyBox 基础系统。你当然可以为内部项目使用更复杂的配置，但 Buildroot 项目只会集成基础的开发板配置，因为软件包的选择高度依赖于具体应用。

一旦你有了已知可用的配置，运行 `make savedefconfig`。这将在 Buildroot 源码树根目录生成一个最小化的 `defconfig` 文件。将该文件移动到 `configs/` 目录，并重命名为 `<boardname>_defconfig`。

始终为各组件使用固定版本或提交哈希，而不是“最新”版本。例如，设置 `BR2_LINUX_KERNEL_CUSTOM_VERSION=y` 和 `BR2_LINUX_KERNEL_CUSTOM_VERSION_VALUE` 为你测试过的内核版本。如果你使用 Buildroot 工具链 `BR2_TOOLCHAIN_BUILDROOT`（默认选项），还需确保使用相同的内核头文件（`BR2_KERNEL_HEADERS_AS_KERNEL`，这也是默认值），并设置自定义内核头文件系列以匹配你的内核版本（`BR2_PACKAGE_HOST_LINUX_HEADERS_CUSTOM_*`）。

建议尽可能使用上游版本的 Linux 内核和引导加载程序，并尽量采用默认的内核和引导加载程序配置。如果这些配置对你的开发板不适用，或没有默认配置，欢迎你向相关上游项目提交修复补丁。

但在此期间，你可能希望存储针对目标平台的内核或引导加载程序配置或补丁。为此，请创建 `board/<manufacturer>` 目录和 `board/<manufacturer>/<boardname>` 子目录。你可以将补丁和配置存放在这些目录中，并在主 Buildroot 配置中引用。更多细节请参阅[第9章，*项目定制*](https://buildroot.org/downloads/manual/manual.html#customize)。

在提交新开发板补丁前，建议使用最新的 gitlab-CI docker 容器进行构建测试。为此可使用 `utils/docker-run` 脚本，并在其中执行如下命令：

```
 $ make <boardname>_defconfig
 $ make
```

默认情况下，Buildroot 开发者使用托管在 [gitlab.com registry](https://gitlab.com/buildroot.org/buildroot/container_registry/2395076) 的官方镜像，大多数场景下都很方便。如果你仍希望构建自己的 docker 镜像，可以在自己的 *Dockerfile* 的 `FROM` 指令中基于官方镜像：

```
FROM registry.gitlab.com/buildroot.org/buildroot/base:YYYYMMDD.HHMM
RUN ...
COPY ...
```

当前版本 *YYYYMMDD.HHMM* 可在 Buildroot 源码树顶层的 `.gitlab-ci.yml` 文件中找到；所有历史版本也可在上述 registry 中查阅。
