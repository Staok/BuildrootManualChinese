## 18.18. 基于Go的软件包基础设施

该基础设施适用于使用标准构建系统和自带依赖的Go包。

### 18.18.1. `golang-package` 教程

首先，让我们通过一个例子，看看如何为Go包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.0
08: FOO_SITE = $(call github,bar,foo,$(FOO_VERSION))
09: FOO_LICENSE = BSD-3-Clause
10: FOO_LICENSE_FILES = LICENSE
11:
12: $(eval $(golang-package))
```

第7行，声明了软件包的版本。

第8行，声明了包的上游位置，这里从Github获取，因为大量Go包托管在Github上。

第9和10行，给出了该包的许可信息。

最后，第12行，调用 `golang-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

### 18.18.2. `golang-package` 参考

在 `Config.in` 文件中，使用 `golang-package` 基础设施的软件包应依赖 `BR2_PACKAGE_HOST_GO_TARGET_ARCH_SUPPORTS`，因为Buildroot会自动为这些包添加对 `host-go` 的依赖。如果你的包需要CGO支持，必须添加对 `BR2_PACKAGE_HOST_GO_TARGET_CGO_LINKING_SUPPORTS` 的依赖；对于主机包，添加对 `BR2_PACKAGE_HOST_GO_HOST_CGO_LINKING_SUPPORTS` 的依赖。

Go软件包基础设施的主要宏是 `golang-package`，它与 `generic-package` 宏类似。也可以通过 `host-golang-package` 宏支持主机包。主机包应依赖 `BR2_PACKAGE_HOST_GO_HOST_ARCH_SUPPORTS`。

与通用基础设施类似，Go基础设施通过在调用 `golang-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在Go基础设施中同样适用。

注意，不需要在包的 `FOO_DEPENDENCIES` 变量中添加 `host-go`，因为Go基础设施会根据需要自动添加该基本依赖。

还可以根据包的需要选择性地定义一些Go基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个，甚至一个都不用。

- 包必须在 `FOO_GOMOD` 变量中指定其Go模块名。如果未指定，默认值为 `URL-domain/1st-part-of-URL/2nd-part-of-URL`，例如对于 `FOO_SITE = $(call github,bar,foo,$(FOO_VERSION))`，`FOO_GOMOD` 默认为 `github.com/bar/foo`。如果包源码树中没有go.mod文件，Go基础设施会自动生成一个最小的go.mod文件。
- `FOO_LDFLAGS`、`FOO_EXTLDFLAGS` 和 `FOO_TAGS` 可分别用于传递go的 `LDFLAGS`（通过 `-ldflags` 命令行参数）、外部链接器标志 `EXTLDFLAGS`（通过 `-extldflags` 命令行参数）或 `TAGS` 给 `go` 构建命令。
- `FOO_BUILD_TARGETS` 可用于指定应构建的目标列表。如果未指定，默认为 `.`。有两种情况：
  - `FOO_BUILD_TARGETS` 为 `.`，假定只生成一个二进制文件，默认以包名命名。若不合适，可用 `FOO_BIN_NAME` 覆盖。
  - `FOO_BUILD_TARGETS` 非 `.`，则遍历每个目标并生成对应的二进制文件，文件名为目标的非目录部分。例如 `FOO_BUILD_TARGETS = cmd/docker cmd/dockerd`，则生成 `docker` 和 `dockerd`。
- `FOO_INSTALL_BINS` 可用于指定应安装到目标 `/usr/bin` 的二进制文件列表。如果未指定，默认为包名的小写形式。

使用Go基础设施，构建和安装包所需的所有步骤都已定义好，并且对于大多数基于Go的软件包来说都能很好地工作。然而，在需要时，仍然可以自定义某个特定步骤的行为：

- 通过添加后操作钩子（在提取、打补丁、配置、构建或安装之后）。详见[第18.23节，“各构建步骤可用的钩子”](https://buildroot.org/downloads/manual/manual.html#hooks)。
- 通过重写某个步骤。例如，即使使用了Go基础设施，如果包的 `.mk` 文件定义了自己的 `FOO_BUILD_CMDS` 变量，则会使用自定义的而不是默认的Go命令。但这种方法应仅限于非常特殊的情况。一般情况下不要使用。

Go包可以依赖于其go.mod文件中列出的其他Go模块。Buildroot会自动在使用 `golang-package` 基础设施的软件包的下载步骤中下载这些依赖项。这些依赖项会与包源码一起保存在Buildroot的 `DL_DIR` 缓存tarball中，因此包tarball的哈希也包含这些依赖。

该机制确保依赖项的任何更改都会被检测到，并允许完全离线构建。

## 18.19. 基于QMake的软件包基础设施

### 18.19.1. `qmake-package` 教程

首先，让我们通过一个例子，看看如何为基于QMake的软件包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # libfoo
04: #
05: ################################################################################
06:
07: LIBFOO_VERSION = 1.0
08: LIBFOO_SOURCE = libfoo-$(LIBFOO_VERSION).tar.gz
09: LIBFOO_SITE = http://www.foosoftware.org/download
10: LIBFOO_CONF_OPTS = QT_CONFIG+=bar QT_CONFIG-=baz
11: LIBFOO_DEPENDENCIES = bar
12:
13: $(eval $(qmake-package))
```

第7行，声明了软件包的版本。

第8和9行，声明了tarball（建议使用xz压缩包）的名称以及在Web上的下载地址。Buildroot会自动从该地址下载tarball。

第10行，告诉Buildroot为libfoo启用哪些选项。

第11行，声明了libfoo的依赖项。

最后，第13行，调用 `qmake-package` 宏，生成实际允许软件包被构建的所有Makefile规则。
