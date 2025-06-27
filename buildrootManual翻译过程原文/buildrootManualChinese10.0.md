## 18.3. `.mk` 文件

最后，这里是最难的部分。创建一个名为 `libfoo.mk` 的文件。它描述了该软件包应该如何被下载、配置、构建、安装等。

根据软件包类型，`.mk` 文件的编写方式不同，需要使用不同的基础架构：

- **通用软件包的 Makefile**（不使用 autotools 或 CMake）：这些基于与 autotools 软件包类似的基础架构，但开发者需要做更多工作。需要指定软件包的配置、编译和安装过程。所有不使用 autotools 作为构建系统的软件包都必须使用此基础架构。将来，可能会为其他构建系统编写专用基础架构。我们在[教程](https://buildroot.org/downloads/manual/manual.html#generic-package-tutorial)和[参考文档](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中有详细介绍。
- **基于 autotools 的软件包 Makefile**（autoconf、automake 等）：我们为此类软件包提供了专用基础架构，因为 autotools 是非常常见的构建系统。所有依赖 autotools 作为构建系统的新软件包*必须*使用此基础架构。详见[教程](https://buildroot.org/downloads/manual/manual.html#autotools-package-tutorial)和[参考文档](https://buildroot.org/downloads/manual/manual.html#autotools-package-reference)。
- **基于 cmake 的软件包 Makefile**：我们为此类软件包提供了专用基础架构，因为 CMake 越来越常用且行为标准化。所有依赖 CMake 的新软件包*必须*使用此基础架构。详见[教程](https://buildroot.org/downloads/manual/manual.html#cmake-package-tutorial)和[参考文档](https://buildroot.org/downloads/manual/manual.html#cmake-package-reference)。
- **Python 模块的 Makefile**：我们为使用 `flit`、`hatch`、`pep517`、`poetry`、`setuptools`、`setuptools-rust` 或 `maturin` 机制的 Python 模块提供了专用基础架构。详见[教程](https://buildroot.org/downloads/manual/manual.html#python-package-tutorial)和[参考文档](https://buildroot.org/downloads/manual/manual.html#python-package-reference)。
- **Lua 模块的 Makefile**：我们为通过 LuaRocks 网站提供的 Lua 模块提供了专用基础架构。详见[教程](https://buildroot.org/downloads/manual/manual.html#luarocks-package-tutorial)和[参考文档](https://buildroot.org/downloads/manual/manual.html#luarocks-package-reference)。

更多格式细节请参见[编写规则](https://buildroot.org/downloads/manual/manual.html#writing-rules-mk)。

## 18.4. `.hash` 文件

如果可能，你必须添加第三个文件，名为 `libfoo.hash`，其中包含 `libfoo` 软件包下载文件的哈希值。只有在无法校验哈希（如下载方式特殊）时，才可以不添加 `.hash` 文件。

当一个软件包有多个版本可选时，哈希文件可以存放在以版本号命名的子目录下，例如 `package/libfoo/1.2.3/libfoo.hash`。如果不同版本有不同的许可条款但存储在同一文件中，这一点尤其重要。否则，哈希文件应保留在软件包目录下。

该文件中存储的哈希用于校验下载文件和许可证文件的完整性。

该文件的格式为：每个需要校验哈希的文件一行，每行包含以下三个字段，字段之间用两个空格分隔：

- 哈希类型，取值为：
  - `md5`、`sha1`、`sha224`、`sha256`、`sha384`、`sha512`
- 文件的哈希值：
  - `md5` 为 32 个十六进制字符
  - `sha1` 为 40 个十六进制字符
  - `sha224` 为 56 个十六进制字符
  - `sha256` 为 64 个十六进制字符
  - `sha384` 为 96 个十六进制字符
  - `sha512` 为 128 个十六进制字符
- 文件名：
  - 对于源码归档包：文件的基本名（不含目录）
  - 对于许可证文件：在 `FOO_LICENSE_FILES` 中出现的路径

以 `#` 开头的行为注释，会被忽略。空行也会被忽略。

同一个文件可以有多条哈希记录，每条一行。这种情况下，所有哈希都必须匹配。

**注意：** 理想情况下，文件中的哈希应与上游（upstream）发布的哈希一致，例如在其官网、邮件公告等处。如果上游提供多种哈希（如 `sha1` 和 `sha512`），建议都写入 `.hash` 文件。如果上游未提供哈希或只提供 `md5`，则请自行计算至少一种强哈希（优先 `sha256`，不要只用 `md5`），并在哈希上方用注释说明。

**注意：** 许可证文件的哈希用于在软件包版本升级时检测许可证变更。哈希会在执行 make legal-info 目标时校验。对于有多个版本的软件包（如 Qt5），请在该软件包的 `<packageversion>` 子目录下创建哈希文件（参见[第 19.2 节，“补丁的应用顺序”](https://buildroot.org/downloads/manual/manual.html#patch-apply-order)）。

下面的例子定义了主 `libfoo-1.2.3.tar.bz2` 压缩包的上游 `sha1` 和 `sha256`，二进制文件的上游 `md5` 和本地计算的 `sha256`，下载补丁的 `sha256`，以及一个没有哈希的归档包：

```
# Hashes from: http://www.foosoftware.org/download/libfoo-1.2.3.tar.bz2.{sha1,sha256}:
sha1  486fb55c3efa71148fe07895fd713ea3a5ae343a  libfoo-1.2.3.tar.bz2
sha256  efc8103cc3bcb06bda6a781532d12701eb081ad83e8f90004b39ab81b65d4369  libfoo-1.2.3.tar.bz2

# md5 from: http://www.foosoftware.org/download/libfoo-1.2.3.tar.bz2.md5, sha256 locally computed:
md5  2d608f3c318c6b7557d551a5a09314f03452f1a1  libfoo-data.bin
sha256  01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b  libfoo-data.bin

# Locally computed:
sha256  ff52101fb90bbfc3fe9475e425688c660f46216d7e751c4bbdb1dc85cdccacb9  libfoo-fix-blabla.patch

# Hash for license files:
sha256  a45a845012742796534f7e91fe623262ccfb99460a2bd04015bd28d66fba95b8  COPYING
sha256  01b1f9f2c8ee648a7a596a1abe8aa4ed7899b1c9e5551bda06da6e422b04aa55  doc/COPYING.LGPL
```

如果 `.hash` 文件存在，并且其中包含一个或多个下载文件的哈希，Buildroot（下载后）计算的哈希必须与 `.hash` 文件中存储的哈希一致。如果有一个或多个哈希不匹配，Buildroot 会认为这是错误，删除下载的文件并中止。

如果 `.hash` 文件存在，但未包含某个下载文件的哈希，Buildroot 会认为这是错误并中止。但下载的文件会保留在下载目录，因为这通常表示 `.hash` 文件有误，但下载的文件可能没问题。

目前，Buildroot 会校验通过 http/ftp 服务器、Git 或 subversion 仓库、scp 复制和本地文件获取的文件的哈希。对于其他版本控制系统（如 CVS、mercurial），Buildroot 不会校验哈希，因为从这些系统获取源码时无法生成可复现的压缩包。

此外，对于可以指定自定义版本的软件包（如自定义版本字符串、远程 tarball URL 或 VCS 仓库位置和变更集），Buildroot 无法为这些情况提供哈希。不过可以[提供额外哈希列表](https://buildroot.org/downloads/manual/manual.html#customize-hashes)来覆盖这些情况。

只有保证文件内容稳定时，才应在 `.hash` 文件中添加哈希。例如，Github 自动生成的补丁不保证稳定，其哈希可能随时变化，因此不应下载此类补丁，而应将其本地添加到软件包文件夹。

如果缺少 `.hash` 文件，则不会进行任何校验。
