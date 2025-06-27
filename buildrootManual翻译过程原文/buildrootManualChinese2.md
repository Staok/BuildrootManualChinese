# 第二部分 用户指南

## 第 6 章 Buildroot 配置

所有 `make *config` 的配置选项都带有帮助文本，详细说明该选项的作用。

`make *config` 命令还提供了搜索工具。具体用法请参见各前端菜单中的帮助信息：

- 在 *menuconfig* 中，按 `/` 调出搜索工具；
- 在 *xconfig* 中，按 `Ctrl` + `f` 调出搜索工具。

搜索结果会显示匹配项的帮助信息。在 *menuconfig* 中，左侧的数字为快捷方式，输入该数字可直接跳转到对应条目，若因依赖未满足该条目不可选，则跳转到包含该条目的菜单。

虽然菜单结构和帮助文本已较为自解释，但部分主题需要额外说明，详见下文。

## 6.1 交叉编译工具链（Cross-compilation toolchain）

编译工具链（compilation toolchain）是一组用于为你的系统编译代码的工具。它包括编译器（compiler，通常为 `gcc`）、二进制工具（binary utils，如汇编器 assembler 和链接器 linker，通常为 `binutils`）以及 C 标准库（C standard library，例如 [GNU Libc](http://www.gnu.org/software/libc/libc.html)、[uClibc-ng](http://www.uclibc-ng.org/)）。

你的开发主机系统通常已自带一套编译工具链，可用于编译在本机运行的应用程序。如果你使用 PC，则该工具链在 x86 处理器上运行并生成 x86 代码。在大多数 Linux 系统中，工具链使用 GNU libc（glibc）作为 C 标准库。此类工具链称为“主机编译工具链”（host compilation toolchain），其运行的机器称为“主机系统”（host system）[3]。

主机编译工具链由发行版提供，Buildroot 仅使用它来构建交叉编译工具链及其他主机端工具。

如上所述，主机自带的工具链只能为主机处理器生成代码。嵌入式系统通常使用不同的处理器，因此需要交叉编译工具链（cross-compilation toolchain）：即在*主机系统*上运行、为*目标系统*（target system）和目标处理器生成代码的工具链。例如，主机为 x86，目标为 ARM，则主机工具链在 x86 上运行并生成 x86 代码，而交叉编译工具链在 x86 上运行并生成 ARM 代码。

Buildroot 提供两种交叉编译工具链方案：

- ***内部工具链后端（internal toolchain backend）***，在配置界面中称为 `Buildroot toolchain`；
- ***外部工具链后端（external toolchain backend）***，在配置界面中称为 `External toolchain`。

可在 `Toolchain` 菜单的 `Toolchain Type` 选项中选择方案。选择后会出现相应配置选项，详见下文。

### 6.1.1 内部工具链后端（Internal toolchain backend）

*内部工具链后端* 指 Buildroot 自行构建交叉编译工具链，然后再为目标嵌入式系统构建用户空间应用和库。

该后端支持多种 C 库：[uClibc-ng](http://www.uclibc-ng.org/)、[glibc](http://www.gnu.org/software/libc/libc.html) 和 [musl](http://www.musl-libc.org/)。

选择该后端后，可配置以下重要选项：

- 更改用于构建工具链的 Linux 内核头文件（kernel headers）版本。此项需特别说明：在构建交叉编译工具链过程中，需要构建 C 库。C 库为用户空间应用与 Linux 内核的接口。为实现此接口，C 库需访问 *Linux 内核头文件*（即内核的 `.h` 文件），这些头文件定义了用户空间与内核的接口（系统调用、数据结构等）。由于该接口向后兼容，构建工具链时使用的内核头文件版本无需与目标设备实际运行的内核版本*完全一致*，只需不高于目标内核版本即可。如果头文件版本高于目标内核，则 C 库可能会使用目标内核不支持的接口。
- 更改 GCC 编译器、binutils 及 C 库的版本。
- 选择多项工具链选项（仅 uClibc）：如是否支持 RPC（主要用于 NFS）、宽字符（wide-char）、本地化（locale，国际化）、C++ 支持、线程支持等。不同选项会影响 Buildroot 菜单中可见的用户空间应用和库：许多应用和库依赖特定工具链选项。大多数软件包在需要特定工具链选项时会有提示。如需进一步自定义 uClibc 配置，可运行 `make uclibc-menuconfig`。但需注意，Buildroot 仅测试其自带的 uClibc 默认配置：如自行裁剪 uClibc 功能，部分软件包可能无法编译。

注意：每当修改上述选项之一时，需重新构建整个工具链和系统。详见[第 8.2 节 “何时需要全量重构”](https://buildroot.org/downloads/manual/manual.html#full-rebuild)。

该后端优点：

- 与 Buildroot 集成良好
- 构建速度快，仅构建必要部分

缺点：

- 执行 `make clean` 时需重建工具链，耗时较长。如需缩短构建时间，可考虑*外部工具链后端*。

### 6.1.2 外部工具链后端（External toolchain backend）

*外部工具链后端* 允许使用已有的预编译交叉编译工具链。Buildroot 已内置多款知名交叉编译工具链（如 [Linaro](http://www.linaro.org/) 针对 ARM，[Sourcery CodeBench](http://www.mentor.com/embedded-software/sourcery-tools/sourcery-codebench/editions/lite-edition/) 针对 ARM、x86-64、PowerPC 和 MIPS），可自动下载，或指定自定义工具链（本地或可下载）。

使用外部工具链有三种方式：

- 使用预定义外部工具链配置文件，由 Buildroot 自动下载、解压和安装工具链。Buildroot 已内置部分 CodeSourcery 和 Linaro 工具链配置，只需在 `Toolchain` 菜单中选择即可。这是最简单的方式。
- 使用预定义外部工具链配置文件，但不让 Buildroot 下载和解压，而是手动指定已安装的工具链路径。在 `Toolchain` 菜单中选择配置，取消勾选 `Download toolchain automatically`，并在 `Toolchain path` 填写交叉编译工具链路径。
- 使用完全自定义的外部工具链。适用于用 crosstool-NG 或 Buildroot 自行生成的工具链。选择 `Custom toolchain`，填写 `Toolchain path`、`Toolchain prefix` 和 `External toolchain C library`。然后需告知 Buildroot 该工具链的支持特性：如为 *glibc*，只需说明是否支持 C++ 及内置 RPC；如为 *uClibc*，则需说明是否支持 RPC、宽字符、本地化、程序调用、线程和 C++。Buildroot 启动时会校验所选项与工具链配置是否匹配。

Buildroot 的外部工具链支持已在 CodeSourcery、Linaro、crosstool-NG 及 Buildroot 自身生成的工具链上测试。一般来说，支持 *sysroot* 特性的工具链均可用。如遇问题请联系开发者。

不支持 OpenEmbedded 或 Yocto 生成的工具链或 SDK，因为这些并非纯工具链（即仅包含编译器、binutils、C/C++ 库），而是包含大量预编译库和程序，Buildroot 无法导入其 *sysroot*，否则会包含数百兆预编译库，这些库本应由 Buildroot 构建。

也不支持将发行版自带工具链（即发行版安装的 gcc/binutils/C 库）作为目标软件编译工具链。因为发行版工具链并非“纯”工具链，无法正确导入 Buildroot 构建环境。即使目标为 x86 或 x86_64，也需用 Buildroot 或 crosstool-NG 生成交叉编译工具链。

如需为项目生成可用作 Buildroot 外部工具链的自定义工具链，建议用 Buildroot（见[第 6.1.3 节 “用 Buildroot 构建外部工具链”](https://buildroot.org/downloads/manual/manual.html#build-toolchain-with-buildroot)）或 [crosstool-NG](http://crosstool-ng.org/) 构建。

该后端优点：

- 可用知名、成熟的交叉编译工具链
- 避免了交叉编译工具链的构建时间，通常在嵌入式 Linux 系统整体构建时间中占比很大

缺点：

- 如预编译外部工具链有 bug，除非自行用 Buildroot 或 Crosstool-NG 构建，否则难以获得修复

[3] 主机系统（host system）：即你工作的开发主机。
