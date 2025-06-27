## 18.10. 基于LuaRocks的软件包基础设施

### 18.10.1. `luarocks-package` 教程

首先，让我们通过一个例子，看看如何为基于LuaRocks的软件包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # lua-foo
04: #
05: ################################################################################
06:
07: LUA_FOO_VERSION = 1.0.2-1
08: LUA_FOO_NAME_UPSTREAM = foo
09: LUA_FOO_DEPENDENCIES = bar
10:
11: LUA_FOO_BUILD_OPTS += BAR_INCDIR=$(STAGING_DIR)/usr/include
12: LUA_FOO_BUILD_OPTS += BAR_LIBDIR=$(STAGING_DIR)/usr/lib
13: LUA_FOO_LICENSE = luaFoo license
14: LUA_FOO_LICENSE_FILES = $(LUA_FOO_SUBDIR)/COPYING
15:
16: $(eval $(luarocks-package))
```

第7行，声明了软件包的版本（与rockspec中一致，由上游版本和rockspec修订号用连字符“-”拼接而成）。

第8行，声明该包在LuaRocks上的名称为“foo”。在Buildroot中，Lua相关包的名称以“lua”开头，因此Buildroot名称与上游名称不同。`LUA_FOO_NAME_UPSTREAM` 用于关联这两个名称。

第9行，声明了对本地库的依赖项，这样它们会在本软件包构建前被先行构建。

第11-12行，告诉Buildroot在构建包时向LuaRocks传递自定义选项。

第13-14行，指定了该包的许可条款。

最后，第16行，调用 `luarocks-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

大多数这些细节都可以从 `rock` 和 `rockspec` 文件中获取。因此，可以通过在Buildroot目录下运行 `luarocks buildroot foo lua-foo` 命令自动生成此文件和Config.in文件。该命令会运行LuaRocks的Buildroot插件，自动生成Buildroot包。生成结果仍需手动检查和可能的修改。

- 需要手动更新 `package/Config.in` 文件以包含生成的Config.in文件。

### 18.10.2. `luarocks-package` 参考

LuaRocks是一个用于Lua模块的部署和管理系统，支持多种 `build.type`：`builtin`、`make` 和 `cmake`。在Buildroot中，`luarocks-package` 基础设施仅支持 `builtin` 模式。使用 `make` 或 `cmake` 构建机制的LuaRocks包应分别使用Buildroot的 `generic-package` 和 `cmake-package` 基础设施进行打包。

LuaRocks软件包基础设施的主要宏是 `luarocks-package`：与 `generic-package` 类似，通过定义一系列变量提供包的元数据信息，然后调用 `luarocks-package` 宏。

与通用基础设施类似，LuaRocks基础设施通过在调用 `luarocks-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在LuaRocks基础设施中同样适用。

其中有两个变量由LuaRocks基础设施自动填充（用于 `download` 步骤）。如果你的包不托管在LuaRocks镜像 `$(BR2_LUAROCKS_MIRROR)` 上，可以覆盖它们：

- `LUA_FOO_SITE`，默认值为 `$(BR2_LUAROCKS_MIRROR)`
- `LUA_FOO_SOURCE`，默认值为 `$(lowercase LUA_FOO_NAME_UPSTREAM)-$(LUA_FOO_VERSION).src.rock`

还可以定义一些LuaRocks基础设施特有的变量，在特定情况下可被覆盖：

- `LUA_FOO_NAME_UPSTREAM`，默认值为 `lua-foo`，即Buildroot包名
- `LUA_FOO_ROCKSPEC`，默认值为 `$(lowercase LUA_FOO_NAME_UPSTREAM)-$(LUA_FOO_VERSION).rockspec`
- `LUA_FOO_SUBDIR`，默认值为 `$(LUA_FOO_NAME_UPSTREAM)-$(LUA_FOO_VERSION_WITHOUT_ROCKSPEC_REVISION)`
- `LUA_FOO_BUILD_OPTS`，包含传递给 `luarocks build` 调用的额外构建选项。

## 18.11. 基于Perl/CPAN的软件包基础设施

### 18.11.1. `perl-package` 教程

首先，让我们通过一个例子，看看如何为Perl/CPAN包编写 `.mk` 文件：

```
01: ################################################################################
02: #
03: # perl-foo-bar
04: #
05: ################################################################################
06:
07: PERL_FOO_BAR_VERSION = 0.02
08: PERL_FOO_BAR_SOURCE = Foo-Bar-$(PERL_FOO_BAR_VERSION).tar.gz
09: PERL_FOO_BAR_SITE = $(BR2_CPAN_MIRROR)/authors/id/M/MO/MONGER
10: PERL_FOO_BAR_DEPENDENCIES = perl-strictures
11: PERL_FOO_BAR_LICENSE = Artistic or GPL-1.0+
12: PERL_FOO_BAR_LICENSE_FILES = LICENSE
13: PERL_FOO_BAR_DISTNAME = Foo-Bar
14:
15: $(eval $(perl-package))
```

第7行，声明了软件包的版本。

第8和9行，声明了tarball的名称以及在CPAN服务器上的下载地址。Buildroot会自动从该地址下载tarball。

第10行，声明了依赖项，这样它们会在本软件包构建前被先行构建。

第11和12行，给出了该包的许可信息（第11行为许可证类型，第12行为包含许可证文本的文件）。

第13行，指定了分发名称，供 `utils/scancpan` 脚本（用于重新生成/升级这些包文件）使用。

最后，第15行，调用 `perl-package` 宏，生成实际允许软件包被构建的所有Makefile规则。

大多数数据可以从 https://metacpan.org/ 获取。因此，可以通过在Buildroot目录（或br2-external树）下运行 `utils/scancpan Foo-Bar` 脚本自动生成此文件和Config.in文件，并递归生成CPAN指定的所有依赖项。结果仍需手动编辑，特别需要检查以下内容：

- 如果Perl模块链接了由其他（非Perl）包提供的共享库，该依赖不会自动添加，需手动添加到 `PERL_FOO_BAR_DEPENDENCIES`。
- 需要手动更新 `package/Config.in` 文件以包含生成的Config.in文件。作为提示，`scancpan` 脚本会按字母顺序打印出所需的 `source "…"` 语句。

### 18.11.2. `perl-package` 参考

作为政策，提供Perl/CPAN模块的软件包在Buildroot中应全部命名为 `perl-<something>`。

该基础设施支持多种Perl构建系统：`ExtUtils-MakeMaker`（EUMM）、`Module-Build`（MB）和 `Module-Build-Tiny`。当包同时提供 `Makefile.PL` 和 `Build.PL` 时，默认优先使用 `Build.PL`。

Perl/CPAN软件包基础设施的主要宏是 `perl-package`。它与 `generic-package` 宏类似。也可以通过 `host-perl-package` 宏支持目标和主机软件包。

与通用基础设施类似，Perl/CPAN基础设施通过在调用 `perl-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在Perl/CPAN基础设施中同样适用。

注意，将 `PERL_FOO_INSTALL_STAGING` 设置为 `YES` 没有效果，除非定义了 `PERL_FOO_INSTALL_STAGING_CMDS` 变量。Perl基础设施不会定义这些命令，因为Perl模块通常不需要安装到 `staging` 目录。

还可以定义一些Perl/CPAN基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个。

- `PERL_FOO_PREFER_INSTALLER`/`HOST_PERL_FOO_PREFER_INSTALLER`，指定首选的安装方式。可选值为 `EUMM`（基于 `Makefile.PL` 的ExtUtils-MakeMaker安装）和 `MB`（基于 `Build.PL` 的Module-Build安装）。仅当包同时提供两种安装方式时才使用此变量。
- `PERL_FOO_CONF_ENV`/`HOST_PERL_FOO_CONF_ENV`，用于指定传递给 `perl Makefile.PL` 或 `perl Build.PL` 的额外环境变量。默认值为空。
- `PERL_FOO_CONF_OPTS`/`HOST_PERL_FOO_CONF_OPTS`，用于指定传递给 `perl Makefile.PL` 或 `perl Build.PL` 的额外配置选项。默认值为空。
- `PERL_FOO_BUILD_OPTS`/`HOST_PERL_FOO_BUILD_OPTS`，用于指定在构建步骤中传递给 `make pure_all` 或 `perl Build build` 的额外选项。默认值为空。
- `PERL_FOO_INSTALL_TARGET_OPTS`，用于指定在安装步骤中传递给 `make pure_install` 或 `perl Build install` 的额外选项。默认值为空。
- `HOST_PERL_FOO_INSTALL_OPTS`，用于指定在安装步骤中传递给 `make pure_install` 或 `perl Build install` 的额外选项。默认值为空。
