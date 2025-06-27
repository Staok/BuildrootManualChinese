## 18.23. 各构建步骤可用的钩子（Hooks）

通用基础架构（generic infrastructure）（以及由其派生的 autotools 和 cmake 基础架构）允许软件包指定钩子。这些钩子定义了在现有步骤之后要执行的进一步操作。大多数钩子对于通用软件包来说并不真正有用，因为 `.mk` 文件已经可以完全控制软件包构建过程中每个步骤的操作。

可用的钩子点如下：

- `LIBFOO_PRE_DOWNLOAD_HOOKS`
- `LIBFOO_POST_DOWNLOAD_HOOKS`
- `LIBFOO_PRE_EXTRACT_HOOKS`
- `LIBFOO_POST_EXTRACT_HOOKS`
- `LIBFOO_PRE_RSYNC_HOOKS`
- `LIBFOO_POST_RSYNC_HOOKS`
- `LIBFOO_PRE_PATCH_HOOKS`
- `LIBFOO_POST_PATCH_HOOKS`
- `LIBFOO_PRE_CONFIGURE_HOOKS`
- `LIBFOO_POST_CONFIGURE_HOOKS`
- `LIBFOO_PRE_BUILD_HOOKS`
- `LIBFOO_POST_BUILD_HOOKS`
- `LIBFOO_PRE_INSTALL_HOOKS`（仅限主机包）
- `LIBFOO_POST_INSTALL_HOOKS`（仅限主机包）
- `LIBFOO_PRE_INSTALL_STAGING_HOOKS`（仅限目标包）
- `LIBFOO_POST_INSTALL_STAGING_HOOKS`（仅限目标包）
- `LIBFOO_PRE_INSTALL_TARGET_HOOKS`（仅限目标包）
- `LIBFOO_POST_INSTALL_TARGET_HOOKS`（仅限目标包）
- `LIBFOO_PRE_INSTALL_IMAGES_HOOKS`
- `LIBFOO_POST_INSTALL_IMAGES_HOOKS`
- `LIBFOO_PRE_LEGAL_INFO_HOOKS`
- `LIBFOO_POST_LEGAL_INFO_HOOKS`
- `LIBFOO_TARGET_FINALIZE_HOOKS`

这些变量是*变量名的列表*，包含在该钩子点要执行的操作。这允许在给定的钩子点注册多个钩子。示例如下：

```
define LIBFOO_POST_PATCH_FIXUP
        action1
        action2
endef

LIBFOO_POST_PATCH_HOOKS += LIBFOO_POST_PATCH_FIXUP
```

### 18.23.1. 使用 `POST_RSYNC` 钩子

`POST_RSYNC` 钩子仅对使用本地源的包运行，无论是通过 `local` 站点方法还是 `OVERRIDE_SRCDIR` 机制。在这种情况下，包源代码会通过 `rsync` 从本地位置复制到 buildroot 构建目录。`rsync` 命令并不会复制源目录中的所有文件。例如，属于版本控制系统的文件，如 `.git`、`.hg` 等目录不会被复制。对于大多数包来说，这已经足够，但特定包可以通过 `POST_RSYNC` 钩子执行额外操作。

原则上，钩子中可以包含你想要的任何命令。一个具体的用例是有意通过 `rsync` 复制版本控制目录。你在钩子中使用的 `rsync` 命令可以使用如下变量：

- `$(SRCDIR)`：被覆盖的源目录路径
- `$(@D)`：构建目录路径

### 18.23.2. Target-finalize 钩子

包还可以在 `LIBFOO_TARGET_FINALIZE_HOOKS` 中注册钩子。这些钩子会在所有包构建完成后、生成文件系统镜像之前运行。它们很少被使用，你的包大概率不需要它们。

## 18.24. Gettext 集成与包的交互

许多支持国际化的包会使用 gettext（gettext）库。该库的依赖关系相当复杂，因此需要一些说明。

*glibc* C 库集成了完整的 *gettext* 实现，支持翻译。因此，*glibc* 内置了本地语言支持（Native Language Support，NLS）。

而 *uClibc* 和 *musl* C 库只提供了 gettext 功能的存根实现（stub implementation），这允许使用 gettext 函数编译库和程序，但不提供完整 gettext 实现的翻译能力。对于这类 C 库，如果需要真正的本地语言支持，可以通过 `gettext` 包的 `libintl`（libintl）库来提供。

因此，为了确保本地语言支持被正确处理，Buildroot 中可以使用 NLS 支持的包应当：

1. 当 `BR2_SYSTEM_ENABLE_NLS=y` 时确保启用 NLS 支持。对于 *autotools* 包会自动完成，因此只需对使用其他包基础架构的包进行处理。
2. 在包的 `<pkg>_DEPENDENCIES` 变量中无条件添加 `$(TARGET_NLS_DEPENDENCIES)`。该变量的值会被核心基础架构自动调整为相关包的列表。如果 NLS 支持被禁用，该变量为空；如果启用，则包含 `host-gettext`，以便主机上可用编译翻译文件所需的工具。此外，如果使用 *uClibc* 或 *musl*，该变量还会包含 `gettext`，以获得完整的 *gettext* 实现。
3. 如有需要，在链接参数中添加 `$(TARGET_NLS_LIBS)`，以便包能与 `libintl` 链接。对于 *autotools* 包通常不需要这样做，因为它们通常能自动检测是否需要链接 `libintl`。但使用其他构建系统的包或有问题的 autotools 包可能需要这样做。`$(TARGET_NLS_LIBS)` 应无条件添加到链接参数中，核心会根据配置自动将其设为空或定义为 `-lintl`。

无需对 `Config.in` 文件做任何更改以支持 NLS。

最后，某些包需要在目标系统上提供一些 gettext 工具，例如 `gettext` 程序本身（允许从命令行检索翻译字符串）。在这种情况下，包应当：

- 在其 `Config.in` 文件中使用 `select BR2_PACKAGE_GETTEXT`，并在上方注释说明这是仅运行时依赖。
- 不要在 `.mk` 文件的 `DEPENDENCIES` 变量中添加任何 `gettext` 依赖。

## 18.25. 技巧与提示

### 18.25.1. 包名、配置项名与 Makefile 变量的关系

在 Buildroot 中，以下三者之间存在一定的关系：

- *包名*，即包目录名（也是 `*.mk` 文件名）；
- 在 `Config.in` 文件中声明的配置项名；
- Makefile 变量前缀。

必须保持这些元素之间的一致性，规则如下：

- 包目录和 `*.mk` 文件名即为*包名*本身（如：`package/foo-bar_boo/foo-bar_boo.mk`）；
- *make* 目标名即为包名本身（如：`foo-bar_boo`）；
- 配置项名为大写包名，将 `.` 和 `-` 替换为 `_`，并加前缀 `BR2_PACKAGE_`（如：`BR2_PACKAGE_FOO_BAR_BOO`）；
- `*.mk` 文件变量前缀为大写包名，将 `.` 和 `-` 替换为 `_`（如：`FOO_BAR_BOO_VERSION`）。

### 18.25.2. 如何检查代码风格

Buildroot 在 `utils/check-package` 提供了一个脚本，用于检查新建或修改的文件的代码风格。它不是完整的语言校验器，但能捕捉到许多常见错误。建议在你创建或修改文件后、提交补丁前运行该脚本。

该脚本可用于包、文件系统 Makefile、Config.in 文件等。它不会检查定义包基础架构的文件及其他包含类似通用代码的文件。

使用方法如下，指定你新建或修改的文件：

```
$ ./utils/check-package package/new-package/*
```

如果你已将 `utils` 目录加入 PATH，也可以这样运行：

```
$ cd package/new-package/
$ check-package *
```

该工具也可用于 br2-external 里的包：

```
$ check-package -b /path/to/br2-ext-tree/package/my-package/*
```

`check-package` 脚本需要你安装 `shellcheck` 以及 Python PyPi 包 `flake8` 和 `python-magic`。Buildroot 代码库当前针对 ShellCheck 0.7.1 版本进行测试。如果你使用不同版本的 ShellCheck，可能会看到额外的、未修复的警告。

如果你有 Docker 或 Podman，可以无需安装依赖直接运行 `check-package`：

```
$ ./utils/docker-run ./utils/check-package
```
