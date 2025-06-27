## 18.14. 基于rebar的软件包基础设施

### 18.14.1. `rebar-package` 教程

首先，让我们通过一个例子，看看如何为基于rebar的软件包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # erlang-foobar
04: #
05: ################################################################################
06:
07: ERLANG_FOOBAR_VERSION = 1.0
08: ERLANG_FOOBAR_SOURCE = erlang-foobar-$(ERLANG_FOOBAR_VERSION).tar.xz
09: ERLANG_FOOBAR_SITE = http://www.foosoftware.org/download
10: ERLANG_FOOBAR_DEPENDENCIES = host-libaaa libbbb
11:
12: $(eval $(rebar-package))
```

第7行，声明了软件包的版本。

第8和9行，声明了tarball（建议使用xz压缩包）的名称以及在Web上的下载地址。Buildroot会自动从该地址下载tarball。

第10行，声明了依赖项，这样它们会在本软件包构建前被先行构建。

最后，第12行，调用 `rebar-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

### 18.14.2. `rebar-package` 参考

`rebar` 软件包基础设施的主要宏是 `rebar-package`。它与 `generic-package` 宏类似。也可以通过 `host-rebar-package` 宏支持主机包。

与通用基础设施类似，`rebar` 基础设施通过在调用 `rebar-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在 `rebar` 基础设施中同样适用。

还可以定义一些 `rebar` 基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个。

- `ERLANG_FOOBAR_USE_AUTOCONF`，指定包在配置步骤中是否使用 *autoconf*。当该变量设为 `YES` 时，将使用 `autotools` 基础设施。

  **注意。** 你也可以使用 `autotools` 基础设施中的一些变量：`ERLANG_FOOBAR_CONF_ENV`、`ERLANG_FOOBAR_CONF_OPTS`、`ERLANG_FOOBAR_AUTORECONF`、`ERLANG_FOOBAR_AUTORECONF_ENV` 和 `ERLANG_FOOBAR_AUTORECONF_OPTS`。

- `ERLANG_FOOBAR_USE_BUNDLED_REBAR`，指定包是否自带 *rebar* 并且必须使用自带的。有效值为 `YES` 或 `NO`（默认）。

  **注意。** 如果包自带 *rebar* 工具，但可以使用Buildroot提供的通用版本，只需设为 `NO`（即不指定此变量）。只有在必须使用包自带 *rebar* 工具时才设置。

- `ERLANG_FOOBAR_REBAR_ENV`，用于指定传递给 *rebar* 工具的额外环境变量。

- `ERLANG_FOOBAR_KEEP_DEPENDENCIES`，指定是否保留rebar.config文件中描述的依赖项。有效值为 `YES` 或 `NO`（默认）。除非设为 `YES`，否则rebar基础设施会在post-patch钩子中移除这些依赖，以确保rebar不会下载或编译它们。

使用rebar基础设施，构建和安装软件包所需的所有步骤都已定义好，并且对于大多数基于rebar的软件包来说都能很好地工作。然而，在需要时，仍然可以自定义某个特定步骤的行为：

- 通过添加后操作钩子（在提取、打补丁、配置、构建或安装之后）。详见[第18.23节，“各构建步骤可用的钩子”](https://buildroot.org/downloads/manual/manual.html#hooks)。
- 通过重写某个步骤。例如，即使使用了rebar基础设施，如果包的 `.mk` 文件定义了自己的 `ERLANG_FOOBAR_BUILD_CMDS` 变量，则会使用自定义的而不是默认的rebar命令。但这种方法应仅限于非常特殊的情况。一般情况下不要使用。

## 18.15. 基于Waf的软件包基础设施

### 18.15.1. `waf-package` 教程

首先，让我们通过一个例子，看看如何为基于Waf的软件包编写 `.mk` 文件：

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
10: LIBFOO_CONF_OPTS = --enable-bar --disable-baz
11: LIBFOO_DEPENDENCIES = bar
12:
13: $(eval $(waf-package))
```

第7行，声明了软件包的版本。

第8和9行，声明了tarball（建议使用xz压缩包）的名称以及在Web上的下载地址。Buildroot会自动从该地址下载tarball。

第10行，告诉Buildroot为libfoo启用哪些选项。

第11行，声明了libfoo的依赖项。

最后，第13行，调用 `waf-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

### 18.15.2. `waf-package` 参考

Waf软件包基础设施的主要宏是 `waf-package`。它与 `generic-package` 宏类似。

与通用基础设施类似，Waf基础设施通过在调用 `waf-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在Waf基础设施中同样适用。

还可以定义一些Waf基础设施特有的变量：

- `LIBFOO_SUBDIR` 可以包含包内包含主wscript文件的子目录名称。如果主wscript文件不在tarball解压后的根目录下，这会很有用。如果未指定 `HOST_LIBFOO_SUBDIR`，则默认为 `LIBFOO_SUBDIR`。
- `LIBFOO_NEEDS_EXTERNAL_WAF` 可设为 `YES` 或 `NO`，用于指定Buildroot是否使用自带的waf可执行文件。若设为 `NO`（默认），Buildroot将使用包源码树中的waf；若设为 `YES`，Buildroot会下载并安装waf作为host工具并用于构建包。
- `LIBFOO_WAF_OPTS`，用于指定在包构建过程的每一步（配置、构建、安装）传递给waf脚本的额外选项，默认空。
- `LIBFOO_CONF_OPTS`，用于指定配置步骤传递给waf脚本的额外选项，默认空。
- `LIBFOO_BUILD_OPTS`，用于指定构建步骤传递给waf脚本的额外选项，默认空。
- `LIBFOO_INSTALL_STAGING_OPTS`，用于指定staging安装步骤传递给waf脚本的额外选项，默认空。
- `LIBFOO_INSTALL_TARGET_OPTS`，用于指定target安装步骤传递给waf脚本的额外选项，默认空。
