- `LIBFOO_USERS` 列出该软件包需要创建的用户（如需以特定用户身份运行守护进程或定时任务）。语法与 makedevs 类似，详见[Makeusers 语法文档](https://buildroot.org/downloads/manual/manual.html#makeuser-syntax)。该变量为可选项。

- `LIBFOO_LICENSE` 定义软件包的许可证（或多种许可证）。该名称会出现在 `make legal-info` 生成的 manifest 文件中。如果许可证在 [SPDX License List](https://spdx.org/licenses/) 中，建议使用 SPDX 简称以统一 manifest 文件。否则请精确简明地描述许可证，避免使用如 `BSD` 这类模糊名称。该变量为可选项，若未定义则 manifest 文件中该包的 license 字段为 `unknown`。格式要求如下：

  - 若包的不同部分采用不同许可证，用逗号分隔（如 `LIBFOO_LICENSE = GPL-2.0+, LGPL-2.1+`）。如能明确区分各组件的许可证，可用括号注明（如 `LIBFOO_LICENSE = GPL-2.0+ (programs), LGPL-2.1+ (libraries)`）。
  - 若某些许可证依赖于子选项启用，则用逗号追加条件许可证（如 `FOO_LICENSE += , GPL-2.0+ (programs)`）；基础设施会自动去除逗号前的空格。
  - 若包为双重许可，用 `or` 关键字分隔（如 `LIBFOO_LICENSE = AFL-2.1 or GPL-2.0+`）。

- `LIBFOO_LICENSE_FILES` 是包 tarball 中包含许可证的文件列表（以空格分隔）。`make legal-info` 会将这些文件全部复制到 `legal-info` 目录。详见[法律声明与许可章节](https://buildroot.org/downloads/manual/manual.html#legal-info)。该变量为可选项，若未定义则会有警告，manifest 文件中 license files 字段为 `not saved`。

- `LIBFOO_ACTUAL_SOURCE_TARBALL` 仅适用于 `LIBFOO_SITE` / `LIBFOO_SOURCE` 指向的归档实际上不包含源码而是二进制代码的情况。这种情况极为罕见，主要用于外部工具链（已编译），理论上也可用于其他包。此时通常会有单独的源码归档。设置 `LIBFOO_ACTUAL_SOURCE_TARBALL` 后，Buildroot 会在执行 `make legal-info` 时下载并使用该源码归档收集法律相关材料。注意该文件不会在常规构建或 `make source` 时下载。

- `LIBFOO_ACTUAL_SOURCE_SITE` 指定实际源码归档的位置。默认值为 `LIBFOO_SITE`，若二进制和源码归档在同一目录无需设置。若未设置 `LIBFOO_ACTUAL_SOURCE_TARBALL`，则无需定义该变量。

- `LIBFOO_REDISTRIBUTE` 可设为 `YES`（默认）或 `NO`，用于指示该包源码是否允许再分发。对于非开源包请设为 `NO`，Buildroot 在收集 `legal-info` 时不会保存该包源码。

- `LIBFOO_FLAT_STACKSIZE` 定义以 FLAT 二进制格式构建的应用程序的堆栈大小。NOMMU 架构处理器的应用堆栈运行时无法扩展，FLAT 格式默认堆栈仅 4k 字节。如需更大堆栈可在此指定。

- `LIBFOO_BIN_ARCH_EXCLUDE` 是需在交叉编译二进制检查时忽略的路径列表（以空格分隔，相对于目标目录）。除非包在默认位置（如 `/lib/firmware`、`/usr/lib/firmware`、`/lib/modules`、`/usr/lib/modules`、`/usr/share`）外安装二进制 blob，否则很少需要设置。

- `LIBFOO_IGNORE_CVES` 是需 Buildroot CVE 跟踪工具忽略的 CVE 列表（以空格分隔）。通常用于补丁已修复或该 CVE 实际不影响 Buildroot 包的情况。每次添加 CVE 前必须有 Makefile 注释。例如：

  ```
  # 0001-fix-cve-2020-12345.patch
  LIBFOO_IGNORE_CVES += CVE-2020-12345
  # 仅在与 libbaz 一起构建时受影响，Buildroot 不支持 libbaz
  LIBFOO_IGNORE_CVES += CVE-2020-54321
  ```
