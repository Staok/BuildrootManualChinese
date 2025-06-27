## 第18章 向 Buildroot 添加新软件包

本节介绍如何将新的软件包（用户空间库或应用程序）集成到 Buildroot 中。它还展示了现有软件包是如何集成的，这对于修复问题或调整其配置也很有帮助。

当你添加新软件包时，请务必在各种条件下进行测试（参见[18.25.3节“如何测试你的软件包”](https://buildroot.org/downloads/manual/manual.html#testing-package)），并检查其代码风格（参见[18.25.2节“如何检查代码风格”](https://buildroot.org/downloads/manual/manual.html#check-package)）。

## 18.1 软件包目录（Package Directory）

首先，在 `package` 目录下为你的软件创建一个目录，例如 `libfoo`。

有些软件包已经按主题分组在子目录下：如 `x11r7`、`qt5` 和 `gstreamer`。如果你的软件包属于这些类别之一，请在相应目录下创建你的软件包目录。但不建议新建其他子目录。

## 18.2 配置文件（Config Files）

为了让软件包在配置工具中显示，你需要在软件包目录下创建配置文件。主要有两种类型：`Config.in` 和 `Config.in.host`。

### 18.2.1 `Config.in` 文件

对于目标（target）上使用的软件包，创建名为 `Config.in` 的文件。该文件包含与我们的 `libfoo` 软件相关的选项描述，会在配置工具中显示。基本内容如下：

```
config BR2_PACKAGE_LIBFOO
        bool "libfoo"
        help
          This is a comment that explains what libfoo is. The help text
          should be wrapped.

          http://foosoftware.org/libfoo/
```

`bool` 行、`help` 行以及其他关于配置选项的元数据信息都必须用一个制表符（Tab）缩进。帮助文本本身应使用一个制表符和两个空格缩进，且每行应换行以适应 72 列（Tab 算作 8 个字符，因此文本本身为 62 个字符）。帮助文本最后一行应在空行后注明该项目的上游网址。

Buildroot 的约定是，属性的顺序如下：

1. 选项类型：如 `bool`、`string` 等及其提示
2. 如有需要，`default` 默认值
3. 针对目标的依赖项，使用 `depends on`
4. 针对工具链的依赖项，使用 `depends on`
5. 针对其他软件包的依赖项，使用 `depends on`
6. 选择型依赖项，使用 `select`
7. `help` 关键字和帮助文本

你可以在 `if BR2_PACKAGE_LIBFOO ... endif` 语句中添加其他子选项，以配置你的软件的特定内容。可以参考其他软件包的例子。`Config.in` 文件的语法与内核 Kconfig 文件相同，相关文档见 http://kernel.org/doc/Documentation/kbuild/kconfig-language.txt。

最后，你需要将新的 `libfoo/Config.in` 添加到 `package/Config.in`（如果你的软件包放在某个类别子目录下，则添加到该类别的 Config.in）。这些文件在每个类别下按字母顺序排序，且*只*包含软件包的*裸*名称。

```
source "package/libfoo/Config.in"
```

### 18.2.2 `Config.in.host` 文件

有些软件包还需要为主机（host）系统构建。这里有两种情况：

- 主机软件包仅用于满足一个或多个目标软件包的构建时依赖。在这种情况下，将 `host-foo` 添加到目标软件包的 `BAR_DEPENDENCIES` 变量中。无需创建 `Config.in.host` 文件。

- 主机软件包需要用户在配置菜单中显式选择。在这种情况下，为该主机软件包创建 `Config.in.host` 文件：

  ```
  config BR2_PACKAGE_HOST_FOO
          bool "host foo"
          help
            This is a comment that explains what foo for the host is.

            http://foosoftware.org/foo/
  ```

  其代码风格和选项与 `Config.in` 文件相同。

  最后，你需要将新的 `libfoo/Config.in.host` 添加到 `package/Config.in.host`。这些文件按字母顺序排序，且*只*包含软件包的*裸*名称。

  ```
  source "package/foo/Config.in.host"
  ```

  这样主机软件包就会出现在 `Host utilities` 菜单中。

### 18.2.3 选择 `depends on` 还是 `select`

你的软件包的 `Config.in` 文件还必须确保依赖项被启用。通常，Buildroot 遵循以下规则：

- 对于库类依赖，使用 `select`。这些依赖通常不明显，因此让 kconfig 系统自动选择依赖项更合理。例如，*libgtk2* 软件包使用 `select BR2_PACKAGE_LIBGLIB2` 来确保该库也被启用。`select` 关键字表达的是反向依赖关系。
- 当用户需要明确知晓依赖关系时，使用 `depends on`。通常用于目标架构、MMU 支持和工具链选项的依赖（参见[18.2.4节“对目标和工具链选项的依赖”](https://buildroot.org/downloads/manual/manual.html#dependencies-target-toolchain-options)），或对“大型”依赖（如 X.org 系统）。`depends on` 关键字表达的是正向依赖关系。

**注意：** 目前 *kconfig* 语言的一个问题是，这两种依赖语义在内部并不关联。因此，可能会出现选择了某个软件包，但其某个依赖/要求未被满足的情况。

下面的例子展示了 `select` 和 `depends on` 的用法：

```
config BR2_PACKAGE_RRDTOOL
        bool "rrdtool"
        depends on BR2_USE_WCHAR
        select BR2_PACKAGE_FREETYPE
        select BR2_PACKAGE_LIBART
        select BR2_PACKAGE_LIBPNG
        select BR2_PACKAGE_ZLIB
        help
          RRDtool is the OpenSource industry standard, high performance
          data logging and graphing system for time series data.

          http://oss.oetiker.ch/rrdtool/

comment "rrdtool needs a toolchain w/ wchar"
        depends on !BR2_USE_WCHAR
```

注意，这两种依赖类型只在同类依赖之间具有传递性。

也就是说，在如下例子中：

```
config BR2_PACKAGE_A
        bool "Package A"

config BR2_PACKAGE_B
        bool "Package B"
        depends on BR2_PACKAGE_A

config BR2_PACKAGE_C
        bool "Package C"
        depends on BR2_PACKAGE_B

config BR2_PACKAGE_D
        bool "Package D"
        select BR2_PACKAGE_B

config BR2_PACKAGE_E
        bool "Package E"
        select BR2_PACKAGE_D
```

- 选择 `Package C` 时，只有在 `Package B` 被选中时才可见，而 `Package B` 只有在 `Package A` 被选中时才可见。
- 选择 `Package E` 时，会自动选择 `Package D`，进而选择 `Package B`，但不会检查 `Package B` 的依赖，因此不会选择 `Package A`。
- 由于 `Package B` 被选中但 `Package A` 没有，这就违反了 `Package B` 对 `Package A` 的依赖。因此，在这种情况下，必须显式添加传递依赖：

```
config BR2_PACKAGE_D
        bool "Package D"
        depends on BR2_PACKAGE_A
        select BR2_PACKAGE_B

config BR2_PACKAGE_E
        bool "Package E"
        depends on BR2_PACKAGE_A
        select BR2_PACKAGE_D
```

总体来说，对于库类依赖，建议优先使用 `select`。

注意，这样的依赖关系只会确保依赖项被启用，但不一定会在你的软件包之前构建。为此，还需要在软件包的 `.mk` 文件中表达该依赖。

更多格式细节请参见[代码风格](https://buildroot.org/downloads/manual/manual.html#writing-rules-config-in)。
