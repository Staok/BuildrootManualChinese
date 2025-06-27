### 18.2.4 对目标和工具链选项的依赖

许多软件包依赖于工具链的某些选项：如 C 库的选择、C++ 支持、线程支持、RPC 支持、wchar 支持或动态库支持。有些软件包只能在特定的目标架构上构建，或仅在处理器有 MMU 时可用。

这些依赖项必须在 Config.in 文件中用合适的 *depends on* 语句表达。此外，对于工具链选项的依赖，还应在未启用该选项时显示 `comment`，让用户知道为什么该软件包不可用。对于目标架构或 MMU 支持的依赖，则无需用注释显式提示：因为用户通常无法随意更换目标架构，所以没必要特别显示这些依赖。

`comment` 只应在 `config` 选项本身在工具链选项依赖满足时可见时才显示。这意味着，软件包的所有其他依赖（包括目标架构和 MMU 支持的依赖）都要在 `comment` 定义中重复。为保持清晰，建议将这些非工具链选项的 `depends on` 语句与工具链选项的 `depends on` 语句分开写。如果在同一文件中有对配置选项的依赖（通常是主软件包），建议使用全局 `if ... endif` 结构，而不是在注释和其他配置选项中重复写 `depends on`。

依赖注释的一般格式如下：

```
foo needs a toolchain w/ featA, featB, featC
```

例如：

```
mpd needs a toolchain w/ C++, threads, wchar
```

或

```
crda needs a toolchain w/ threads
```

注意，这些文本特意保持简短，以适应 80 字符终端。

本节其余部分列举了不同的目标和工具链选项、相应的配置符号以及注释文本：

- 目标架构（Target architecture）
  - 依赖符号：`BR2_powerpc`、`BR2_mips` 等（见 `arch/Config.in`）
  - 注释字符串：无需添加注释
- MMU 支持
  - 依赖符号：`BR2_USE_MMU`
  - 注释字符串：无需添加注释
- Gcc `*_sync**` 内建原子操作。不同架构支持不同字节数的原子操作，每种字节数有一个依赖符号：
  - 依赖符号：1字节用 `BR2_TOOLCHAIN_HAS_SYNC_1`，2字节用 `BR2_TOOLCHAIN_HAS_SYNC_2`，4字节用 `BR2_TOOLCHAIN_HAS_SYNC_4`，8字节用 `BR2_TOOLCHAIN_HAS_SYNC_8`
  - 注释字符串：无需添加注释
- Gcc `*_atomic**` 内建原子操作。
  - 依赖符号：`BR2_TOOLCHAIN_HAS_ATOMIC`
  - 注释字符串：无需添加注释
- 内核头文件（Kernel headers）
  - 依赖符号：`BR2_TOOLCHAIN_HEADERS_AT_LEAST_X_Y`（将 `X_Y` 替换为具体版本，见 `toolchain/Config.in`）
  - 注释字符串：`headers >= X.Y` 和/或 `headers <= X.Y`（将 `X.Y` 替换为具体版本）
- GCC 版本
  - 依赖符号：`BR2_TOOLCHAIN_GCC_AT_LEAST_X_Y`（将 `X_Y` 替换为具体版本，见 `toolchain/Config.in`）
  - 注释字符串：`gcc >= X.Y` 和/或 `gcc <= X.Y`（将 `X.Y` 替换为具体版本）
- 主机 GCC 版本
  - 依赖符号：`BR2_HOST_GCC_AT_LEAST_X_Y`（将 `X_Y` 替换为具体版本，见 `Config.in`）
  - 注释字符串：无需添加注释
  - 通常不是软件包本身要求主机 GCC 版本，而是其依赖的主机软件包有此要求。
- C 库（C library）
  - 依赖符号：`BR2_TOOLCHAIN_USES_GLIBC`、`BR2_TOOLCHAIN_USES_MUSL`、`BR2_TOOLCHAIN_USES_UCLIBC`
  - 注释字符串：对于 C 库，注释文本略有不同：`foo needs a glibc toolchain`，或 `foo needs a glibc toolchain w/ C++`
- C++ 支持
  - 依赖符号：`BR2_INSTALL_LIBSTDCPP`
  - 注释字符串：`C++`
- D 语言支持
  - 依赖符号：`BR2_TOOLCHAIN_HAS_DLANG`
  - 注释字符串：`Dlang`
- Fortran 支持
  - 依赖符号：`BR2_TOOLCHAIN_HAS_FORTRAN`
  - 注释字符串：`fortran`
- 线程支持
  - 依赖符号：`BR2_TOOLCHAIN_HAS_THREADS`
  - 注释字符串：`threads`（除非还需要 `BR2_TOOLCHAIN_HAS_THREADS_NPTL`，此时只需写 `NPTL`）
- NPTL 线程支持
  - 依赖符号：`BR2_TOOLCHAIN_HAS_THREADS_NPTL`
  - 注释字符串：`NPTL`
- RPC 支持
  - 依赖符号：`BR2_TOOLCHAIN_HAS_NATIVE_RPC`
  - 注释字符串：`RPC`
- wchar 支持
  - 依赖符号：`BR2_USE_WCHAR`
  - 注释字符串：`wchar`
- 动态库（dynamic library）
  - 依赖符号：`!BR2_STATIC_LIBS`
  - 注释字符串：`dynamic library`

### 18.2.5 对 Buildroot 构建的 Linux 内核的依赖

有些软件包需要 Buildroot 构建的 Linux 内核，通常是内核模块或固件。应在 Config.in 文件中添加注释表达此依赖，格式如下：

```
foo needs a Linux kernel to be built
```

如果同时依赖工具链选项和 Linux 内核，格式如下：

```
foo needs a toolchain w/ featA, featB, featC and a Linux kernel to be built
```

### 18.2.6 对 udev /dev 管理的依赖

如果软件包需要 udev /dev 管理，应依赖符号 `BR2_PACKAGE_HAS_UDEV`，并添加如下注释：

```
foo needs udev /dev management
```

如果同时依赖工具链选项和 udev /dev 管理，格式如下：

```
foo needs udev /dev management and a toolchain w/ featA, featB, featC
```

### 18.2.7 对虚拟软件包（virtual packages）特性依赖

有些特性可由多个软件包提供，如 openGL 库。

详见[18.12节“虚拟软件包基础设施”](https://buildroot.org/downloads/manual/manual.html#virtual-package-tutorial)。
