Buildroot 2025.05 手册，生成于 2025-06-09 20:23:58 UTC，基于 git 修订版 fcde5363aa

Buildroot 手册由 Buildroot 开发者编写。其授权协议为 GNU 通用公共许可证第 2 版（GNU General Public License, version 2）。完整协议内容请参见 Buildroot 源码中的 [COPYING](http://git.buildroot.org/buildroot/tree/COPYING?id=fcde5363aa35220a1f201159a05de652ec6f811f) 文件。

版权所有 © Buildroot 开发者 <[buildroot@buildroot.org](mailto:buildroot@buildroot.org)>

# 第一部分 入门

## 第 1 章 关于 Buildroot

Buildroot 是一个简化并自动化为嵌入式系统构建完整 Linux 系统流程的工具，采用交叉编译（cross-compilation）方式。

为实现这一目标，Buildroot 能够为目标设备生成交叉编译工具链（cross-compilation toolchain）、根文件系统（root filesystem）、Linux 内核镜像（Linux kernel image）以及引导加载程序（bootloader）。Buildroot 可用于上述任意组合（例如，你可以使用已有的交叉编译工具链，仅用 Buildroot 构建根文件系统）。

Buildroot 主要适用于嵌入式系统开发人员。嵌入式系统通常使用的处理器并非大家在 PC 上常见的 x86 处理器，而是 PowerPC 处理器（PowerPC processors）、MIPS 处理器（MIPS processors）、ARM 处理器（ARM processors）等。

Buildroot 支持众多处理器及其变种，并为多款现成开发板（off-the-shelf boards）提供了默认配置。此外，许多第三方项目基于 Buildroot 开发其 BSP（Board Support Package）或 SDK（Software Development Kit）。

------

[1] BSP：板级支持包（Board Support Package）

[2] SDK：软件开发工具包（Software Development Kit）

## 第 2 章 系统要求

Buildroot 设计用于 Linux 系统。

虽然 Buildroot 会自动构建大部分所需的主机端（host）软件包，但仍要求主机系统预先安装部分标准 Linux 工具。下表列出了必需和可选软件包（注意，不同发行版的软件包名称可能有所不同）。

### 2.1 必需软件包

- 构建工具（Build tools）：
  - `which`
  - `sed`
  - `make`（版本 3.81 或更高）
  - `binutils`
  - `build-essential`（仅限基于 Debian 的系统）
  - `diffutils`
  - `gcc`（版本 4.8 或更高）
  - `g++`（版本 4.8 或更高）
  - `bash`
  - `patch`
  - `gzip`
  - `bzip2`
  - `perl`（版本 5.8.7 或更高）
  - `tar`
  - `cpio`
  - `unzip`
  - `rsync`
  - `file`（必须位于 `/usr/bin/file`）
  - `bc`
  - `findutils`
  - `awk`
- 源码获取工具（Source fetching tools）：
  - `wget`

### 2.2 可选软件包

- 推荐依赖（Recommended dependencies）：

  Buildroot 的部分功能或工具（如 legal-info、图形生成工具等）需要额外依赖。虽然这些依赖对于简单构建并非强制，但仍强烈推荐安装：

  - `python`（版本 2.7 或更高）

- 配置界面依赖（Configuration interface dependencies）：

  对于这些库，你需要同时安装运行时和开发包（runtime and development data），在许多发行版中开发包通常以 *-dev* 或 *-devel* 结尾。

  - `ncurses5`（用于 *menuconfig* 界面）
  - `qt5`（用于 *xconfig* 界面）
  - `glib2`、`gtk2` 和 `glade2`（用于 *gconfig* 界面）

- 源码获取工具（Source fetching tools）：

  在官方源码树中，大多数软件包源码通过 `wget` 从 *ftp*、*http* 或 *https* 位置获取。部分软件包仅能通过版本控制系统（version control system）获取。此外，Buildroot 还支持通过 `git` 或 `scp` 等工具下载源码（详见[第 20 章 下载基础设施](https://buildroot.org/downloads/manual/manual.html#download-infra)）。如启用这些方式获取的软件包，需在主机系统安装相应工具：

  - `bazaar`
  - `curl`
  - `cvs`
  - `git`
  - `mercurial`
  - `scp`
  - `sftp`
  - `subversion`

- Java 相关软件包（Java-related packages），如需为目标系统构建 Java Classpath：

  - `javac` 编译器（compiler）
  - `jar` 工具（tool）

- 文档生成工具（Documentation generation tools）：

  - `asciidoc`，版本 8.6.3 或更高
  - `w3m`
  - `python`，需带有 `argparse` 模块（2.7+ 和 3.2+ 默认自带）
  - `dblatex`（仅生成 pdf 手册时需要）

- 图形生成工具（Graph generation tools）：
  - `graphviz`（用于 *graph-depends* 和 *<pkg>-graph-depends*）
  - `python-matplotlib`（用于 *graph-build*）

- 软件包统计工具（Package statistics tools，*pkg-stats*）：
  - `python-aiohttp`

## 第 3 章 获取 Buildroot

Buildroot 每 3 个月发布一次新版本，发布时间为 2 月、5 月、8 月和 11 月。版本号格式为 YYYY.MM，例如 2013.02、2014.08。

发布版压缩包可在 http://buildroot.org/downloads/ 获取。

为方便用户，Buildroot 源码树的 `support/misc/Vagrantfile` 提供了一个 [Vagrantfile](https://www.vagrantup.com/)，可快速搭建包含所需依赖的虚拟机环境。

如需在 Linux 或 Mac OS X 上搭建隔离的 buildroot 环境，请在终端粘贴以下命令：

```
curl -O https://buildroot.org/downloads/Vagrantfile; vagrant up
```

如在 Windows 上，请在 powershell 粘贴以下命令：

```
(new-object System.Net.WebClient).DownloadFile(
"https://buildroot.org/downloads/Vagrantfile","Vagrantfile");
vagrant up
```

如需跟进开发版，可使用每日快照（daily snapshots）或克隆 Git 仓库。更多详情请参见 Buildroot 官网的 [下载页面](http://buildroot.org/download)。
