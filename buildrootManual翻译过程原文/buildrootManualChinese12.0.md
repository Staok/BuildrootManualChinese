## 18.7. 针对 autotools 构建系统的软件包基础设施

### 18.7.1. `autotools-package` 教程

首先，让我们通过一个例子，了解如何为基于 autotools 的软件包编写 `.mk` 文件：

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
10: LIBFOO_INSTALL_STAGING = YES
11: LIBFOO_INSTALL_TARGET = NO
12: LIBFOO_CONF_OPTS = --disable-shared
13: LIBFOO_DEPENDENCIES = libglib2 host-pkgconf
14:
15: $(eval $(autotools-package))
```

第 7 行声明了软件包的版本。

第 8、9 行声明了 tarball 的名称（推荐使用 xz 压缩的 tarball）及其在 Web 上的位置。Buildroot 会自动从该位置下载 tarball。

第 10 行告知 Buildroot 将软件包安装到 staging 目录。staging 目录位于 `output/staging/`，用于安装所有软件包及其开发文件等。默认情况下，软件包不会安装到 staging 目录，只有库（library）通常需要这样做，因为它们的开发文件会被其他依赖它们的库或应用编译时使用。启用 staging 安装后，默认通过 `make install` 命令安装。

第 11 行告知 Buildroot 不将该软件包安装到目标目录。目标目录包含将运行在目标上的根文件系统。对于纯静态库，无需安装到目标目录，因为运行时不会用到。默认情况下，目标安装是启用的；将此变量设为 NO 的情况极少。默认也是通过 `make install` 安装。

第 12 行指定了自定义的 configure 选项，会在配置和构建软件包前传递给 `./configure` 脚本。

第 13 行声明了依赖项，确保它们会在本包构建前被编译。

最后，第 15 行调用 `autotools-package` 宏，生成实际允许该包被构建的所有 Makefile 规则。

### 18.7.2. `autotools-package` 参考

autotools 包基础设施的主要宏是 `autotools-package`，其用法类似于 `generic-package` 宏。也支持目标包和主机包，主机包可用 `host-autotools-package` 宏。

与 generic 基础设施一样，autotools 基础设施通过在调用 `autotools-package` 宏前定义一系列变量来工作。

所有 [generic 包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) 中存在的软件包元数据变量，在 autotools 基础设施中同样适用。

此外，还可以定义一些 autotools 基础设施特有的变量。许多变量仅在特定场景下有用，典型软件包只需用到其中少数几个。

- `LIBFOO_SUBDIR` 可指定包含 configure 脚本的子目录名称。如果主 configure 脚本不在 tarball 解压根目录时很有用。若未指定 `HOST_LIBFOO_SUBDIR`，则默认与 `LIBFOO_SUBDIR` 相同。
- `LIBFOO_CONF_ENV`，指定传递给 configure 脚本的额外环境变量。默认空。
- `LIBFOO_CONF_OPTS`，指定传递给 configure 脚本的额外参数。默认空。
- `LIBFOO_MAKE`，指定替代的 `make` 命令。通常用于配置启用了并行 make（`BR2_JLEVEL`）但该包不支持并行构建时。默认值为 `$(MAKE)`。如包不支持并行构建，应设为 `LIBFOO_MAKE=$(MAKE1)`。
- `LIBFOO_MAKE_ENV`，指定构建步骤中传递给 make 的额外环境变量，位于 make 命令前。默认空。
- `LIBFOO_MAKE_OPTS`，指定构建步骤中传递给 make 的额外变量，位于 make 命令后。默认空。
- `LIBFOO_AUTORECONF`，指定是否需要自动重新配置（即是否需重新运行 autoconf、automake、libtool 等生成 configure 脚本和 Makefile.in 文件）。可选值为 `YES` 或 `NO`，默认 `NO`。
- `LIBFOO_AUTORECONF_ENV`，如 `LIBFOO_AUTORECONF=YES`，则指定传递给 *autoreconf* 程序的额外环境变量。默认空。
- `LIBFOO_AUTORECONF_OPTS`，如 `LIBFOO_AUTORECONF=YES`，则指定传递给 *autoreconf* 程序的额外参数。默认空。
- `LIBFOO_AUTOPOINT`，指定是否需要 autopoint（即包是否需要 I18N 基础设施）。仅在 `LIBFOO_AUTORECONF=YES` 时有效。可选值为 `YES` 或 `NO`，默认 `NO`。
- `LIBFOO_LIBTOOL_PATCH`，指定是否应用 Buildroot 修复 libtool 交叉编译问题的补丁。可选值为 `YES` 或 `NO`，默认 `YES`。
- `LIBFOO_INSTALL_STAGING_OPTS`，指定安装到 staging 目录时的 make 选项。默认值为 `DESTDIR=$(STAGING_DIR) install`，适用于大多数 autotools 包。如需可覆盖。
- `LIBFOO_INSTALL_TARGET_OPTS`，指定安装到目标目录时的 make 选项。默认值为 `DESTDIR=$(TARGET_DIR) install`，适用于大多数 autotools 包。如需可覆盖。

对于 autotools 基础设施，构建和安装包所需的所有步骤都已定义，且对大多数 autotools 包都适用。但如有需要，仍可自定义任意步骤：

- 可添加后置操作钩子（extract、patch、configure、build 或 install 之后）。详见[各构建阶段可用钩子](https://buildroot.org/downloads/manual/manual.html#hooks)。
- 可覆盖某个步骤。例如，即使使用 autotools 基础设施，若包的 `.mk` 文件自定义了 `LIBFOO_CONFIGURE_CMDS` 变量，则会用自定义内容替换默认 autotools 行为。但此方法仅限特殊场景，一般不建议使用。
