# 第三部分：开发者指南

## 第15章 Buildroot 的工作原理

如前所述，Buildroot 本质上是一组 Makefile，用于以正确的选项下载、配置和编译软件。它还包含了针对各种软件包的补丁，主要是交叉编译工具链（`gcc`、`binutils` 和 `uClibc`）相关的软件包。

基本上，每个软件包都有一个对应的 Makefile，文件扩展名为 `.mk`。Makefile 被划分为多个不同的部分。

- `toolchain/` 目录包含与交叉编译工具链相关的所有软件的 Makefile 及相关文件：`binutils`、`gcc`、`gdb`、`kernel-headers` 和 `uClibc`。
- `arch/` 目录包含 Buildroot 支持的所有处理器架构的定义。
- `package/` 目录包含 Buildroot 能够编译并添加到目标根文件系统中的所有用户空间工具和库的 Makefile 及相关文件。每个软件包有一个子目录。
- `linux/` 目录包含 Linux 内核的 Makefile 及相关文件。
- `boot/` 目录包含 Buildroot 支持的引导加载程序的 Makefile 及相关文件。
- `system/` 目录包含系统集成支持，例如目标文件系统 skeleton 和初始化系统的选择。
- `fs/` 目录包含与生成目标根文件系统镜像相关的软件的 Makefile 及相关文件。

每个目录至少包含两个文件：

- `something.mk` 是用于下载、配置、编译和安装 `something` 软件包的 Makefile。
- `Config.in` 是配置工具描述文件的一部分。它描述了与该软件包相关的选项。

主 Makefile 执行以下步骤（配置完成后）：

- 在输出目录（默认为 `output/`，也可通过 `O=` 指定其他值）中创建所有输出目录：`staging`、`target`、`build` 等。
- 生成工具链目标。当使用内部工具链时，这意味着生成交叉编译工具链。当使用外部工具链时，这意味着检查外部工具链的特性并将其导入 Buildroot 环境。
- 生成 `TARGETS` 变量中列出的所有目标。该变量由所有单独组件的 Makefile 填充。生成这些目标会触发用户空间软件包（库、程序）、内核、引导加载程序的编译，以及根文件系统镜像的生成，具体取决于配置。

## 第16章 代码风格

总体来说，这些代码风格规则旨在帮助你为 Buildroot 添加新文件或重构现有文件。

如果你只是对现有文件做了轻微修改，重要的是保持整个文件的风格一致，因此你可以：

- 遵循该文件中可能已过时的代码风格，
- 或者彻底重写，使其符合这些规则。

## 16.1 `Config.in` 文件

`Config.in` 文件包含 Buildroot 中几乎所有可配置项的条目。

一个条目的模式如下：

```
config BR2_PACKAGE_LIBFOO
        bool "libfoo"
        depends on BR2_PACKAGE_LIBBAZ
        select BR2_PACKAGE_LIBBAR
        help
          This is a comment that explains what libfoo is. The help text
          should be wrapped.

          http://foosoftware.org/libfoo/
```

- `bool`、`depends on`、`select` 和 `help` 行使用一个制表符缩进。
- 帮助文本本身应使用一个制表符和两个空格缩进。
- 帮助文本应换行以适应 72 列，其中制表符算作 8 个字符，因此文本本身为 62 个字符。

`Config.in` 文件是 Buildroot 所用配置工具的输入文件，该工具为常规 *Kconfig*。关于 *Kconfig* 语言的更多细节，请参阅 http://kernel.org/doc/Documentation/kbuild/kconfig-language.txt。

## 16.2 `.mk` 文件

- 头部：文件以头部注释开始。内容为模块名，建议小写，位于 80 个 # 号组成的分隔符之间。头部后必须有一个空行：

  ```
  ################################################################################
  #
  # libfoo
  #
  ################################################################################
  ```

- 赋值：使用 `=`，前后各有一个空格：

  ```
  LIBFOO_VERSION = 1.0
  LIBFOO_CONF_OPTS += --without-python-support
  ```

  不要对齐 `=` 号。

- 缩进：只使用制表符：

  ```
  define LIBFOO_REMOVE_DOC
          $(RM) -r $(TARGET_DIR)/usr/share/libfoo/doc \
                  $(TARGET_DIR)/usr/share/man/man3/libfoo*
  endef
  ```

  注意，`define` 块中的命令应始终以制表符开头，这样 *make* 才能识别为命令。

- 可选依赖：

  - 推荐多行语法。

    推荐：

    ```
    ifeq ($(BR2_PACKAGE_PYTHON3),y)
    LIBFOO_CONF_OPTS += --with-python-support
    LIBFOO_DEPENDENCIES += python3
    else
    LIBFOO_CONF_OPTS += --without-python-support
    endif
    ```

    不推荐：

    ```
    LIBFOO_CONF_OPTS += --with$(if $(BR2_PACKAGE_PYTHON3),,out)-python-support
    LIBFOO_DEPENDENCIES += $(if $(BR2_PACKAGE_PYTHON3),python3,)
    ```

  - 保持配置选项和依赖项靠近。

- 可选钩子：将钩子定义和赋值放在同一个 if 块中。

  推荐：

  ```
  ifneq ($(BR2_LIBFOO_INSTALL_DATA),y)
  define LIBFOO_REMOVE_DATA
          $(RM) -r $(TARGET_DIR)/usr/share/libfoo/data
  endef
  LIBFOO_POST_INSTALL_TARGET_HOOKS += LIBFOO_REMOVE_DATA
  endif
  ```

  不推荐：

  ```
  define LIBFOO_REMOVE_DATA
          $(RM) -r $(TARGET_DIR)/usr/share/libfoo/data
  endef

  ifneq ($(BR2_LIBFOO_INSTALL_DATA),y)
  LIBFOO_POST_INSTALL_TARGET_HOOKS += LIBFOO_REMOVE_DATA
  endif
  ```
