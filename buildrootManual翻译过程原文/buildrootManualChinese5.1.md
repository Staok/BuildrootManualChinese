## 9.2 保持定制内容在 Buildroot 之外

如[9.1节“推荐的目录结构”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure)中简要提到，你可以将项目专属定制内容放在两个位置：

- 直接放在 Buildroot 树内，通常通过版本控制系统的分支进行管理，这样升级到新版本 Buildroot 时会更方便。
- 放在 Buildroot 树外，使用 *br2-external*（原文：br2-external）机制。该机制允许将软件包配方、板级支持和配置文件放在 Buildroot 树外，同时又能很好地集成到构建逻辑中。我们称这个位置为 *br2-external 树*。本节将介绍如何使用 br2-external 机制以及 br2-external 树中需要提供哪些内容。

可以通过设置 `BR2_EXTERNAL` make 变量为 br2-external 树的路径（可以是一个或多个）来让 Buildroot 使用 br2-external 树。该变量可以在任何 Buildroot `make` 命令中传递。它会自动保存在输出目录下的隐藏文件 `.br2-external.mk` 中。因此，无需在每次 `make` 时都传递 `BR2_EXTERNAL`。当然，你可以随时通过传递新值来更改它，也可以通过传递空值来移除。

**注意**：br2-external 树的路径可以是绝对路径或相对路径。如果是相对路径，需要注意它是相对于 Buildroot 源码目录（***不是*** Buildroot 输出目录）来解释的。

**注意**：如果你使用的是 Buildroot 2016.11 之前的 br2-external 树，需要先进行转换，才能与 Buildroot 2016.11 及以后版本配合使用。转换方法见[27.2节“迁移到2016.11”](https://buildroot.org/downloads/manual/manual.html#br2-external-converting)。

一些示例：

```
buildroot/ $ make BR2_EXTERNAL=/path/to/foo menuconfig
```

从此以后，将会使用 `/path/to/foo` br2-external 树中的定义：

```
buildroot/ $ make
buildroot/ $ make legal-info
```

我们可以随时切换到另一个 br2-external 树：

```
buildroot/ $ make BR2_EXTERNAL=/where/we/have/bar xconfig
```

也可以同时使用多个 br2-external 树：

```
buildroot/ $ make BR2_EXTERNAL=/path/to/foo:/where/we/have/bar menuconfig
```

或者禁用所有 br2-external 树：

```
buildroot/ $ make BR2_EXTERNAL= xconfig
```

### 9.2.1 br2-external 树的布局

一个 br2-external 树至少要包含以下三个文件，后续章节会详细介绍：

- `external.desc`
- `external.mk`
- `Config.in`

除了这些必需文件外，br2-external 树中还可以有其他可选内容，如 `configs/` 或 `provides/` 目录。它们也会在后续章节中介绍。

后面还会有一个完整的 br2-external 树布局示例。

#### `external.desc` 文件

该文件描述 br2-external 树：*名称*和*描述*。

该文件为逐行格式，每行以关键字开头，后跟冒号和一个或多个空格，再跟要赋值的内容。目前识别两个关键字：

- `name`，必需，定义 br2-external 树的名称。该名称只能使用 ASCII 字符集 `[A-Za-z0-9_]`，其他字符均不允许。Buildroot 会设置变量 `BR2_EXTERNAL_$(NAME)_PATH` 为 br2-external 树的绝对路径，这样你可以用它来引用 br2-external 树。该变量在 Kconfig 和 Makefile 中都可用，因此你可以用它来引用 Kconfig 文件（见下文），也可以在 Makefile 中引用其他文件（如数据文件）。

  **注意**：由于可以同时使用多个 br2-external 树，这个名称会被 Buildroot 用来为每个树生成变量。该名称用于标识你的 br2-external 树，所以请尽量取一个有代表性的、相对唯一的名字，避免与其他 br2-external 树冲突，尤其是你打算与第三方共享或使用第三方 br2-external 树时。

- `desc`，可选，为 br2-external 树提供简短描述。应控制在一行内，内容较为自由（见下文），用于显示 br2-external 树相关信息（如在 defconfig 文件列表上方或 menuconfig 提示中）；因此应尽量简洁（建议不超过40个字符）。该描述会存储在变量 `BR2_EXTERNAL_$(NAME)_DESC` 中。

名称与变量示例：

- `FOO` → `BR2_EXTERNAL_FOO_PATH`
- `BAR_42` → `BR2_EXTERNAL_BAR_42_PATH`

以下示例假设名称为 `BAR_42`。

**注意**：`BR2_EXTERNAL_$(NAME)_PATH` 和 `BR2_EXTERNAL_$(NAME)_DESC` 在 Kconfig 和 Makefile 中都可用，也会导出到环境变量，因此在 post-build、post-image 和 in-fakeroot 脚本中也可用。

#### `Config.in` 和 `external.mk` 文件

这两个文件（可以为空）可用于定义软件包配方（即 `foo/Config.in` 和 `foo/foo.mk`，与 Buildroot 自带软件包类似）或其他自定义配置选项和 make 逻辑。

Buildroot 会自动包含每个 br2-external 树的 `Config.in`，使其出现在顶层配置菜单中，并包含每个 br2-external 树的 `external.mk`，与其他 makefile 逻辑一起处理。

主要用途是存放软件包配方。推荐写法如下：

`Config.in` 文件：

```
source "$BR2_EXTERNAL_BAR_42_PATH/package/package1/Config.in"
source "$BR2_EXTERNAL_BAR_42_PATH/package/package2/Config.in"
```

`external.mk` 文件：

```
include $(sort $(wildcard $(BR2_EXTERNAL_BAR_42_PATH)/package/*/*.mk))
```

然后在 `$(BR2_EXTERNAL_BAR_42_PATH)/package/package1` 和 `$(BR2_EXTERNAL_BAR_42_PATH)/package/package2` 下创建常规 Buildroot 软件包配方，具体方法见[第18章“向 Buildroot 添加新软件包”](https://buildroot.org/downloads/manual/manual.html#adding-packages)。如有需要，也可以按 <boardname> 分组。

你也可以在 `Config.in` 中定义自定义配置选项，在 `external.mk` 中定义自定义 make 逻辑。

#### `configs/` 目录

可以在 br2-external 树的 `configs` 子目录下存放 Buildroot defconfig 文件。Buildroot 会自动在 `make list-defconfigs` 输出中显示它们，并允许用常规 `make <name>_defconfig` 命令加载。它们会在 *make list-defconfigs* 输出中显示在 `External configs` 标签下，标签名为 br2-external 树的名称。

**注意**：如果同名 defconfig 文件在多个 br2-external 树中都存在，则以最后一个 br2-external 树中的为准。因此可以覆盖 Buildroot 自带或其他 br2-external 树中的 defconfig。
