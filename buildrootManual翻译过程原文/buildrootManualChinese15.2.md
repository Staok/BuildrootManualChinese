### 18.21.2. `asciidoc-document` 参考

可以在 `.mk` 文件中设置如下变量（假设文档名为 `foo`），用于提供元数据信息：

- `FOO_SOURCES`（必需）：定义文档的源文件。
- `FOO_RESOURCES`（可选）：可包含一个或多个资源目录（如CSS或图片）的路径，空格分隔。默认值为空。
- `FOO_DEPENDENCIES`（可选）：构建该文档前必须构建的软件包列表（多为host包）。
- `FOO_TOC_DEPTH`、`FOO_TOC_DEPTH_<FMT>`（可选）：文档目录的深度，可针对指定格式 `<FMT>`（见上文渲染格式列表，全部大写，连字符用下划线替换）单独设置。默认值为 `1`。

还可以设置额外的钩子（详见[第18.23节，“各构建步骤可用的钩子”](https://buildroot.org/downloads/manual/manual.html#hooks)）：

- `FOO_POST_RSYNC_HOOKS`：在Buildroot复制源文件后运行额外命令。例如可用于根据源码树信息生成手册部分内容。Buildroot自身用此钩子生成附录表格。
- `FOO_CHECK_DEPENDENCIES_HOOKS`：对生成文档所需组件进行额外测试。AsciiDoc支持调用过滤器（filter），即对AsciiDoc块进行解析和渲染的程序（如 [ditaa](http://ditaa.sourceforge.net/) 或 [aafigure](https://pythonhosted.org/aafigure/)）。
- `FOO_CHECK_DEPENDENCIES_<FMT>_HOOKS`：对指定格式 `<FMT>` 进行额外测试。

Buildroot会设置如下变量，可在上述定义中使用：

- `$(FOO_DOCDIR)`：类似于 `$(FOO_PKGDIR)`，包含 `foo.mk` 所在目录路径。可用于引用文档源文件，或在钩子中使用，尤其是post-rsync钩子需生成文档部分内容时。
- `$(@D)`：与传统包一样，包含文档被复制和构建的目录路径。

下面是一个使用了所有变量和钩子的完整示例：

```
01: ################################################################################
02: #
03: # foo-document
04: #
05: ################################################################################
06:
07: FOO_SOURCES = $(sort $(wildcard $(FOO_DOCDIR)/*))
08: FOO_RESOURCES = $(sort $(wildcard $(FOO_DOCDIR)/resources))
09:
10: FOO_TOC_DEPTH = 2
11: FOO_TOC_DEPTH_HTML = 1
12: FOO_TOC_DEPTH_SPLIT_HTML = 3
13:
14: define FOO_GEN_EXTRA_DOC
15:     /path/to/generate-script --outdir=$(@D)
16: endef
17: FOO_POST_RSYNC_HOOKS += FOO_GEN_EXTRA_DOC
18:
19: define FOO_CHECK_MY_PROG
20:     if ! which my-prog >/dev/null 2>&1; then \
21:         echo "You need my-prog to generate the foo document"; \
22:         exit 1; \
23:     fi
24: endef
25: FOO_CHECK_DEPENDENCIES_HOOKS += FOO_CHECK_MY_PROG
26:
27: define FOO_CHECK_MY_OTHER_PROG
28:     if ! which my-other-prog >/dev/null 2>&1; then \
29:         echo "You need my-other-prog to generate the foo document as PDF"; \
30:         exit 1; \
31:     fi
32: endef
33: FOO_CHECK_DEPENDENCIES_PDF_HOOKS += FOO_CHECK_MY_OTHER_PROG
34:
35: $(eval $(call asciidoc-document))
```

## 18.22. Linux内核包专用基础设施

Linux内核包可基于包钩子使用一些专用基础设施，用于构建Linux内核工具或扩展。

### 18.22.1. linux-kernel-tools

Buildroot提供了辅助基础设施，用于为目标系统构建Linux内核源码中的部分用户空间工具。由于这些工具的源码属于内核源码，因此有一个特殊包 `linux-tools`，复用目标系统运行的Linux内核源码。

以Linux工具 `foo` 为例。为新工具 `foo` 在现有 `package/linux-tools/Config.in` 中创建菜单项。该文件包含每个内核工具的选项描述，配置工具中会显示。大致如下：

```
01: config BR2_PACKAGE_LINUX_TOOLS_FOO
02:     bool "foo"
03:     select BR2_PACKAGE_LINUX_TOOLS
04:     help
05:       这里是对foo内核工具的说明注释。
06:
07:       http://foosoftware.org/foo/
```

选项名以 `BR2_PACKAGE_LINUX_TOOLS_` 为前缀，后接工具大写名（与包命名方式一致）。

**注意。** 与其他包不同，`linux-tools` 选项出现在 `linux` 内核菜单下的 `Linux Kernel Tools` 子菜单中，而不是 `Target packages` 主菜单。

然后为每个linux工具添加一个名为 `package/linux-tools/linux-tool-foo.mk.in` 的 `.mk.in` 文件，大致如下：

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: LINUX_TOOLS += foo
08:
09: FOO_DEPENDENCIES = libbbb
10:
11: define FOO_BUILD_CMDS
12:     $(TARGET_MAKE_ENV) $(MAKE) -C $(LINUX_DIR)/tools foo
13: endef
14:
15: define FOO_INSTALL_STAGING_CMDS
16:     $(TARGET_MAKE_ENV) $(MAKE) -C $(LINUX_DIR)/tools \
17:             DESTDIR=$(STAGING_DIR) \
18:             foo_install
19: endef
20:
21: define FOO_INSTALL_TARGET_CMDS
22:     $(TARGET_MAKE_ENV) $(MAKE) -C $(LINUX_DIR)/tools \
23:             DESTDIR=$(TARGET_DIR) \
24:             foo_install
25: endef
```

第7行，将Linux工具 `foo` 注册到可用工具列表。

第9行，指定该工具依赖的包列表。仅当选择了 `foo` 工具时，这些依赖才会被添加到Linux包依赖列表。

其余Makefile（第11-25行）定义了Linux工具构建流程各步骤的操作，类似于[通用包](https://buildroot.org/downloads/manual/manual.html#generic-package-tutorial)。只有选择了 `foo` 工具时才会实际使用。仅支持 `_BUILD_CMDS`、`_INSTALL_STAGING_CMDS` 和 `_INSTALL_TARGET_CMDS` 命令。

**注意。** 不能调用 `$(eval $(generic-package))` 或其他包基础设施！Linux工具不是独立包，而是 `linux-tools` 包的一部分。

### 18.22.2. linux-kernel-extensions

有些包提供新特性，需要修改Linux内核源码树。这可能是以补丁形式应用到内核树，也可能是以新文件形式添加到源码树。Buildroot的Linux内核扩展基础设施提供了简单方案，在内核源码解压后、补丁应用前自动完成。例如Xenomai、RTAI实时扩展和 `fbtft`（一组树外LCD驱动）等。

以添加新内核扩展 `foo` 为例：

首先，创建提供该扩展的包 `foo`，这是一个标准包，负责下载源码、校验哈希、定义许可证信息及构建用户空间工具（如有）。

然后创建*Linux扩展*本体：在现有 `linux/Config.ext.in` 中添加菜单项。该文件包含每个内核扩展的选项描述，配置工具中会显示。大致如下：

```
01: config BR2_LINUX_KERNEL_EXT_FOO
02:     bool "foo"
03:     help
04:       这里是对foo内核扩展的说明注释。
05:
06:       http://foosoftware.org/foo/
```

然后为每个linux扩展添加一个名为 `linux/linux-ext-foo.mk` 的 `.mk` 文件，内容大致如下：

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: LINUX_EXTENSIONS += foo
08:
09: define FOO_PREPARE_KERNEL
10:     $(FOO_DIR)/prepare-kernel-tree.sh --linux-dir=$(@D)
11: endef
```

第7行，将Linux扩展 `foo` 添加到可用扩展列表。

第9-11行，定义扩展对内核源码树的修改操作；具体内容由扩展决定，可用 `foo` 包定义的变量（如 `$(FOO_DIR)`、`$(FOO_VERSION)`）及所有Linux变量（如 `$(LINUX_VERSION)`、`$(LINUX_VERSION_PROBED)`、`$(KERNEL_ARCH)`）。详见[这些内核变量的定义](https://buildroot.org/downloads/manual/manual.html#kernel-variables)。
