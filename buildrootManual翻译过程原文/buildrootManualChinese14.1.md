## 18.16. 基于Meson的软件包基础设施

### 18.16.1. `meson-package` 教程

[Meson](http://mesonbuild.com/) 是一个开源构建系统，旨在极快且尽可能用户友好。它使用 [Ninja](https://ninja-build.org/) 作为实际构建操作的辅助工具。

下面通过一个例子，看看如何为基于Meson的软件包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.0
08: FOO_SOURCE = foo-$(FOO_VERSION).tar.gz
09: FOO_SITE = http://www.foosoftware.org/download
10: FOO_LICENSE = GPL-3.0+
11: FOO_LICENSE_FILES = COPYING
12: FOO_INSTALL_STAGING = YES
13:
14: FOO_DEPENDENCIES = host-pkgconf bar
15:
16: ifeq ($(BR2_PACKAGE_BAZ),y)
17: FOO_CONF_OPTS += -Dbaz=true
18: FOO_DEPENDENCIES += baz
19: else
20: FOO_CONF_OPTS += -Dbaz=false
21: endif
22:
23: $(eval $(meson-package))
```

Makefile从第7到11行定义了包声明的标准变量。

第23行，调用 `meson-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

在示例中，`host-pkgconf` 和 `bar` 在第14行被声明为依赖项，因为 `foo` 的Meson构建文件使用 `pkg-config` 来确定包 `bar` 的编译标志和库。

注意，不需要在包的 `FOO_DEPENDENCIES` 变量中添加 `host-meson`，因为Meson基础设施会根据需要自动添加该基本依赖。

如果选择了“baz”包，则通过在第17行向 `FOO_CONF_OPTS` 添加 `-Dbaz=true` 来激活对“baz”特性的支持，并在 `FOO_DEPENDENCIES` 中添加“baz”。如果未选择，则在第20行显式禁用对“baz”的支持。

总之，添加新的基于meson的软件包时，可以直接复制该Makefile示例，然后将所有 `FOO` 替换为新包的大写名，并更新标准变量的值。

### 18.16.2. `meson-package` 参考

Meson软件包基础设施的主要宏是 `meson-package`。它与 `generic-package` 宏类似。也可以通过 `host-meson-package` 宏支持目标和主机包。

与通用基础设施类似，Meson基础设施通过在调用 `meson-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在Meson基础设施中同样适用。

还可以定义一些Meson基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个。

- `FOO_SUBDIR` 可以包含包内包含主 meson.build 文件的子目录名称。如果主 meson.build 文件不在tarball解压后的根目录下，这会很有用。如果未指定 `HOST_FOO_SUBDIR`，则默认为 `FOO_SUBDIR`。
- `FOO_CONF_ENV`，用于指定配置步骤传递给 `meson` 的额外环境变量，默认空。
- `FOO_CONF_OPTS`，用于指定配置步骤传递给 `meson` 的额外选项，默认空。
- `FOO_CFLAGS`，用于指定添加到包专用 `cross-compile.conf` 文件 `c_args` 属性的编译器参数，默认值为 `TARGET_CFLAGS`。
- `FOO_CXXFLAGS`，用于指定添加到包专用 `cross-compile.conf` 文件 `cpp_args` 属性的编译器参数，默认值为 `TARGET_CXXFLAGS`。
- `FOO_LDFLAGS`，用于指定添加到包专用 `cross-compile.conf` 文件 `c_link_args` 和 `cpp_link_args` 属性的编译器参数，默认值为 `TARGET_LDFLAGS`。
- `FOO_MESON_EXTRA_BINARIES`，用于指定要添加到meson `cross-compilation.conf` 配置文件 `[binaries]` 部分的程序列表，格式为 `program-name='/path/to/program'`，默认空。Buildroot已为 `c`、`cpp`、`ar`、`strip` 和 `pkgconfig` 设置了正确的值。
- `FOO_MESON_EXTRA_PROPERTIES`，用于指定要添加到meson `cross-compilation.conf` 配置文件 `[properties]` 部分的属性列表，格式为 `property-name=<value>`，默认空。Buildroot已为 `needs_exe_wrapper`、`c_args`、`c_link_args`、`cpp_args`、`cpp_link_args`、`sys_root` 和 `pkg_config_libdir` 设置了值。
- `FOO_NINJA_ENV`，用于指定传递给ninja（meson的构建工具）的额外环境变量，默认空。
- `FOO_NINJA_OPTS`，用于指定要构建的目标列表，默认空（构建默认目标）。

## 18.17. 基于Cargo的软件包基础设施

Cargo是Rust编程语言的软件包管理器。它允许用户构建Rust程序或库，并自动下载和管理其依赖项，以确保可重复构建。Cargo包称为“crate”。

### 18.17.1. `cargo-package` 教程

Cargo包foo的 `Config.in` 文件应包含：

```
01: config BR2_PACKAGE_FOO
02:     bool "foo"
03:     depends on BR2_PACKAGE_HOST_RUSTC_TARGET_ARCH_SUPPORTS
04:     select BR2_PACKAGE_HOST_RUSTC
05:     help
06:       这里是对foo的说明注释。
07:
08:       http://foosoftware.org/foo/
```

该包的 `.mk` 文件应包含：

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.0
08: FOO_SOURCE = foo-$(FOO_VERSION).tar.gz
09: FOO_SITE = http://www.foosoftware.org/download
10: FOO_LICENSE = GPL-3.0+
11: FOO_LICENSE_FILES = COPYING
12:
13: $(eval $(cargo-package))
```

Makefile从第7到11行定义了包声明的标准变量。

第13行，基于 `cargo-package` 基础设施。Cargo会被该基础设施自动调用以构建和安装包。

仍可自定义构建命令或安装命令（即定义 FOO_BUILD_CMDS 和 FOO_INSTALL_TARGET_CMDS），这些会替换cargo基础设施的默认命令。

### 18.17.2. `cargo-package` 参考

Cargo软件包基础设施的主要宏为 `cargo-package`（目标包）和 `host-cargo-package`（主机包）。

与通用基础设施类似，Cargo基础设施通过在调用 `cargo-package` 或 `host-cargo-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在Cargo基础设施中同样适用。

还可以定义一些Cargo基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个。

- `FOO_SUBDIR` 可以包含包内包含Cargo.toml文件的子目录名称。如果Cargo.toml文件不在tarball解压后的根目录下，这会很有用。如果未指定 `HOST_FOO_SUBDIR`，则默认为 `FOO_SUBDIR`。
- `FOO_CARGO_ENV` 可用于在cargo调用时传递额外的环境变量，构建和安装时均适用。
- `FOO_CARGO_BUILD_OPTS` 可用于在构建时向cargo传递额外选项。
- `FOO_CARGO_INSTALL_OPTS` 可用于在安装时向cargo传递额外选项。

crate可以依赖于其Cargo.toml文件中列出的其他库（来自crates.io或git仓库）。Buildroot会自动在使用 `cargo-package` 基础设施的软件包的下载步骤中下载这些依赖项。这些依赖项会与包源码一起保存在Buildroot的 `DL_DIR` 缓存tarball中，因此包tarball的哈希不仅覆盖包本身的源码，也覆盖依赖项源码。这样，依赖项的任何更改都会被哈希检查发现。此外，该机制允许完全离线构建，因为cargo在构建时不会下载任何内容。该机制称为依赖项的vendor化（vendoring）。
