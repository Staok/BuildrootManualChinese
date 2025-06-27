## 第19章. 修补（Patching）软件包

在集成新软件包或更新已有软件包时，可能需要修补（patch）软件源代码，以便在 Buildroot 中进行交叉编译。

Buildroot 提供了自动处理补丁的基础架构。它支持三种应用补丁集的方式：下载补丁、Buildroot 内部自带补丁，以及用户自定义的全局补丁目录。

## 19.1. 提供补丁

### 19.1.1. 下载方式

如果需要应用可下载的补丁，则将其添加到 `<packagename>_PATCH` 变量中。如果条目包含 `://`，Buildroot 会认为它是完整的 URL，并从该位置下载补丁。否则，Buildroot 会认为补丁应从 `<packagename>_SITE` 下载。可以是单个补丁，也可以是包含补丁集的 tarball。

与所有下载一样，应在 `<packagename>.hash` 文件中添加哈希值。

这种方式通常用于 Debian 的软件包。

### 19.1.2. Buildroot 内部补丁

大多数补丁直接放在 Buildroot 的包目录下，这些补丁通常用于修复交叉编译、libc 支持或其他类似问题。

补丁文件应命名为 `<number>-<description>.patch`。

**注意事项**

- Buildroot 自带的补丁文件名不应包含包版本信息。
- 补丁文件名中的 `<number>` 表示*应用顺序*，应从 1 开始，建议用零填充至 4 位，如 *git-format-patch* 所为。例如：`0001-foobar-the-buz.patch`
- 补丁邮件主题前缀不应编号。补丁应通过 `git format-patch -N` 生成，因为该命令会自动为系列补丁添加编号。例如，补丁主题应为 `Subject: [PATCH] foobar the buz`，而不是 `Subject: [PATCH n/m] foobar the buz`。
- 以前要求补丁文件名前缀加包名，如 `<package>-<number>-<description>.patch`，但现在不再要求。已有包会逐步修正。*不要再用包名作为补丁前缀。*
- 以前可以在包目录下添加 `series` 文件（如 quilt 所用），用于定义补丁应用顺序。该做法已废弃，未来会移除。*不要再用 series 文件。*

### 19.1.3. 全局补丁目录

`BR2_GLOBAL_PATCH_DIR` 配置项可用于指定一个或多个包含全局包补丁的目录（以空格分隔）。详见 [9.8.1 节 “提供额外补丁”](https://buildroot.org/downloads/manual/manual.html#customize-patches)。

## 19.2. 补丁的应用方式

1. 如果定义了 `<packagename>_PRE_PATCH_HOOKS`，先运行这些命令；
2. 清理构建目录，移除所有现有的 `*.rej` 文件；
3. 如果定义了 `<packagename>_PATCH`，则应用这些 tarball 中的补丁；
4. 如果包的 Buildroot 目录或其子目录 `<packageversion>` 下有 `*.patch` 文件，则：
   - 如果包目录下有 `series` 文件，则按 `series` 文件顺序应用补丁；
   - 否则，按字母顺序应用所有 `*.patch` 文件。为确保顺序正确，强烈建议补丁文件命名为 `<number>-<description>.patch`，其中 `<number>` 表示*应用顺序*。
5. 如果定义了 `BR2_GLOBAL_PATCH_DIR`，则按指定顺序遍历这些目录，补丁应用方式同上一步。
6. 如果定义了 `<packagename>_POST_PATCH_HOOKS`，则运行这些命令。

如果第 3 或 4 步出错，构建会失败。
