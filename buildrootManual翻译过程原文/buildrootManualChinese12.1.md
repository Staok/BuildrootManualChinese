## 18.8. 基于CMake的软件包基础设施

### 18.8.1. `cmake-package` 教程

首先，让我们通过一个例子，看看如何为基于CMake的软件包编写 `.mk` 文件：

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
12: LIBFOO_CONF_OPTS = -DBUILD_DEMOS=ON
13: LIBFOO_DEPENDENCIES = libglib2 host-pkgconf
14:
15: $(eval $(cmake-package))
```

第7行，声明了软件包的版本。

第8和9行，声明了tarball（建议使用xz压缩包）的名称以及在Web上的下载地址。Buildroot会自动从该地址下载tarball。

第10行，告诉Buildroot将该软件包安装到staging目录。staging目录位于 `output/staging/`，是所有软件包（包括开发文件等）被安装的目录。默认情况下，软件包不会被安装到staging目录，因为通常只有库（library）才需要安装到staging目录：它们的开发文件用于编译依赖它们的其他库或应用程序。默认情况下，当启用staging安装时，软件包会通过 `make install` 命令安装到该位置。

第11行，告诉Buildroot不要将该软件包安装到target目录。该目录包含将来在目标设备上运行的根文件系统内容。对于纯静态库来说，没有必要安装到target目录，因为它们在运行时不会被使用。默认情况下，target安装是启用的；将该变量设置为NO几乎不需要。默认情况下，软件包会通过 `make install` 命令安装到该位置。

第12行，告诉Buildroot在配置软件包时传递自定义的CMake选项。

第13行，声明了依赖项，这样它们会在本软件包构建前被先行构建。

最后，第15行，调用 `cmake-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

### 18.8.2. `cmake-package` 参考

CMake软件包基础设施的主要宏是 `cmake-package`。它与 `generic-package` 宏类似。也可以通过 `host-cmake-package` 宏支持目标（target）和主机（host）软件包。

与通用基础设施类似，CMake基础设施通过在调用 `cmake-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在CMake基础设施中同样适用。

此外，还可以定义一些CMake基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个。

- `LIBFOO_SUBDIR` 可以包含包内包含主CMakeLists.txt文件的子目录名称。如果主CMakeLists.txt文件不在tarball解压后的根目录下，这会很有用。如果未指定 `HOST_LIBFOO_SUBDIR`，则默认为 `LIBFOO_SUBDIR`。
- `LIBFOO_CMAKE_BACKEND` 指定要使用的cmake后端，可以是 `make`（使用GNU Makefiles生成器，默认）或 `ninja`（使用Ninja生成器）。
- `LIBFOO_CONF_ENV`，用于指定传递给CMake的额外环境变量。默认值为空。
- `LIBFOO_CONF_OPTS`，用于指定传递给CMake的额外配置选项。默认值为空。CMake基础设施会设置许多常用CMake选项，因此通常不需要在包的 `*.mk` 文件中设置，除非需要覆盖它们：
  - `CMAKE_BUILD_TYPE` 由 `BR2_ENABLE_RUNTIME_DEBUG` 控制；
  - `CMAKE_INSTALL_PREFIX`；
  - `BUILD_SHARED_LIBS` 由 `BR2_STATIC_LIBS` 控制；
  - `BUILD_DOC`、`BUILD_DOCS` 被禁用；
  - `BUILD_EXAMPLE`、`BUILD_EXAMPLES` 被禁用；
  - `BUILD_TEST`、`BUILD_TESTS`、`BUILD_TESTING` 被禁用。
- `LIBFOO_BUILD_ENV` 和 `LIBFOO_BUILD_OPTS`，用于指定在构建时传递给后端的额外环境变量或命令行选项。
- `LIBFOO_SUPPORTS_IN_SOURCE_BUILD = NO`，当软件包不能在源码树内构建而需要单独的构建目录时应设置此项。
- `LIBFOO_MAKE`，用于指定备用的 `make` 命令。当配置中启用了并行make（使用 `BR2_JLEVEL`）但由于某些原因需要为该包禁用此功能时，这通常很有用。默认值为 `$(MAKE)`。如果该包不支持并行构建，则应设置为 `LIBFOO_MAKE=$(MAKE1)`。
- `LIBFOO_MAKE_ENV`，用于指定在构建步骤中传递给make的额外环境变量。这些变量会在 `make` 命令前传递。默认值为空。
- `LIBFOO_MAKE_OPTS`，用于指定在构建步骤中传递给make的额外变量。这些变量会在 `make` 命令后传递。默认值为空。
- `LIBFOO_INSTALL_OPTS`，包含用于将软件包安装到host目录的make选项。默认值为 `install`，对大多数CMake包来说是正确的，但仍可覆盖。
- `LIBFOO_INSTALL_STAGING_OPTS`，包含用于将软件包安装到staging目录的make选项。默认值为 `DESTDIR=$(STAGING_DIR) install/fast`，对大多数CMake包来说是正确的，但仍可覆盖。
- `LIBFOO_INSTALL_TARGET_OPTS`，包含用于将软件包安装到target目录的make选项。默认值为 `DESTDIR=$(TARGET_DIR) install/fast`。对大多数CMake包来说是正确的，但如有需要仍可覆盖。

使用CMake基础设施，构建和安装软件包所需的所有步骤都已定义好，并且对于大多数基于CMake的软件包来说都能很好地工作。然而，在需要时，仍然可以自定义某个特定步骤的行为：

- 通过添加后操作钩子（在提取、打补丁、配置、构建或安装之后）。详见[第18.23节，“各构建步骤可用的钩子”](https://buildroot.org/downloads/manual/manual.html#hooks)。
- 通过重写某个步骤。例如，即使使用了CMake基础设施，如果包的 `.mk` 文件定义了自己的 `LIBFOO_CONFIGURE_CMDS` 变量，则会使用自定义的而不是默认的CMake命令。但这种方法应仅限于非常特殊的情况。一般情况下不要使用。

## 18.9. 基于Python的软件包基础设施

该基础设施适用于使用标准Python setuptools、pep517、flit或maturin机制作为构建系统的Python软件包，通常可以通过 `setup.py` 脚本或 `pyproject.toml` 文件识别。

### 18.9.1. `python-package` 教程

首先，让我们通过一个例子，看看如何为Python软件包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # python-foo
04: #
05: ################################################################################
06:
07: PYTHON_FOO_VERSION = 1.0
08: PYTHON_FOO_SOURCE = python-foo-$(PYTHON_FOO_VERSION).tar.xz
09: PYTHON_FOO_SITE = http://www.foosoftware.org/download
10: PYTHON_FOO_LICENSE = BSD-3-Clause
11: PYTHON_FOO_LICENSE_FILES = LICENSE
12: PYTHON_FOO_ENV = SOME_VAR=1
13: PYTHON_FOO_DEPENDENCIES = libmad
14: PYTHON_FOO_SETUP_TYPE = setuptools
15:
16: $(eval $(python-package))
```

第7行，声明了软件包的版本。

第8和9行，声明了tarball（建议使用xz压缩包）的名称以及在Web上的下载地址。Buildroot会自动从该地址下载tarball。

第10和11行，给出了该软件包的许可信息（第10行为许可证类型，第11行为包含许可证文本的文件）。

第12行，告诉Buildroot在配置软件包时传递自定义选项给Python的 `setup.py` 脚本。

第13行，声明了依赖项，这样它们会在本软件包构建前被先行构建。

第14行，声明了所使用的Python构建系统类型。本例中使用的是 `setuptools`。目前支持的有 `flit`、`hatch`、`pep517`、`poetry`、`setuptools`、`setuptools-rust` 和 `maturin`。

最后，第16行，调用 `python-package` 宏，生成实际允许软件包被构建的所有Makefile规则。
