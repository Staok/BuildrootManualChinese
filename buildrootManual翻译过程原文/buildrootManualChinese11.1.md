### 18.6.2. `generic-package` 参考

`generic-package` 有两种变体。`generic-package` 宏用于为目标（target）进行交叉编译的软件包。`host-generic-package` 宏用于主机（host）软件包，即为主机本地编译的软件包。在同一个 `.mk` 文件中可以同时调用这两个宏：一次用于生成目标包的规则，一次用于生成主机包的规则：

```
$(eval $(generic-package))
$(eval $(host-generic-package))
```

如果目标包的编译需要在主机上安装某些工具，这种做法会很有用。如果包名为 `libfoo`，那么目标包的名称也是 `libfoo`，而主机包的名称是 `host-libfoo`。如果其他软件包依赖于 `libfoo` 或 `host-libfoo`，应在其 DEPENDENCIES 变量中使用这些名称。

对 `generic-package` 和/或 `host-generic-package` 宏的调用***必须***在 `.mk` 文件的末尾、所有变量定义之后。若两者都调用，`host-generic-package` ***必须***在 `generic-package` 之后调用。

对于目标包，`generic-package` 使用 `.mk` 文件中以大写包名为前缀定义的变量：`LIBFOO_*`。`host-generic-package` 使用 `HOST_LIBFOO_*` 变量。对于*某些*变量，如果没有定义 `HOST_LIBFOO_` 前缀的变量，包基础设施会使用对应的 `LIBFOO_` 前缀变量。这适用于目标包和主机包值可能相同的变量。详见下文。

可以在 `.mk` 文件中设置以下变量来提供元数据信息（假设包名为 `libfoo`）：

- `LIBFOO_VERSION`，必填，必须包含软件包的版本。如果没有 `HOST_LIBFOO_VERSION`，则默认与 `LIBFOO_VERSION` 相同。对于直接从版本控制系统获取的软件包，也可以是修订号或标签。例如：

  - 发布版 tarball 的版本：`LIBFOO_VERSION = 0.1.2`

  - git 树的 sha1：`LIBFOO_VERSION = cb9d6aa9429e838f0e54faa3d455bcbab5eef057`

  - git 树的标签：`LIBFOO_VERSION = v0.1.2`

    **注意：** 不支持将分支名用作 `FOO_VERSION`，因为这实际上无法达到预期效果：

    1. 由于本地缓存，Buildroot 不会重新获取仓库，因此希望跟踪远程仓库的人会感到困惑和失望；
    2. 因为两次构建不可能完全同时进行，且远程仓库的分支可能随时有新提交，所以即使两个用户使用相同的 Buildroot 树和配置，也可能获取到不同的源码，导致构建结果不可复现，这也会让人感到困惑和失望。

- `LIBFOO_SOURCE` 可包含软件包 tarball 的名称，Buildroot 会用它从 `LIBFOO_SITE` 下载 tarball。如果未指定 `HOST_LIBFOO_SOURCE`，则默认与 `LIBFOO_SOURCE` 相同。如果都未指定，则默认值为 `libfoo-$(LIBFOO_VERSION).tar.gz`。例如：`LIBFOO_SOURCE = foobar-$(LIBFOO_VERSION).tar.bz2`

- `LIBFOO_PATCH` 可包含以空格分隔的补丁文件名列表，Buildroot 会下载并应用到软件包源码。如果条目中包含 `://`，Buildroot 会认为它是完整 URL 并用该地址下载补丁，否则会从 `LIBFOO_SITE` 下载补丁。如果未指定 `HOST_LIBFOO_PATCH`，则默认与 `LIBFOO_PATCH` 相同。注意，Buildroot 自带的补丁采用不同机制：所有在 Buildroot 软件包目录下的 `*.patch` 文件会在源码解压后自动应用（详见[补丁策略](https://buildroot.org/downloads/manual/manual.html#patch-policy)）。最后，`LIBFOO_PATCH` 变量中列出的补丁会在 Buildroot 包目录下的补丁之前应用。

- `LIBFOO_SITE` 提供软件包的位置，可以是 URL 或本地文件系统路径。支持 HTTP、FTP 和 SCP 等 URL 类型来获取 tarball。在这些情况下不要加结尾斜杠：Buildroot 会自动在目录和文件名之间添加斜杠。支持 Git、Subversion、Mercurial 和 Bazaar 等源码管理系统直接获取源码。还有辅助函数可简化从 GitHub 下载源码 tarball（详见[如何添加 GitHub 软件包](https://buildroot.org/downloads/manual/manual.html#github-download-url)）。文件系统路径可用于指定 tarball 或源码目录。更多检索细节见下文 `LIBFOO_SITE_METHOD`。注意，SCP URL 格式为 `scp://[user@]host:filepath`，其中 filepath 相对于用户主目录，因此如需绝对路径请加斜杠：`scp://[user@]host:/absolutepath`。SFTP 也同理。如果未指定 `HOST_LIBFOO_SITE`，则默认与 `LIBFOO_SITE` 相同。例如：`LIBFOO_SITE=http://www.libfoosoftware.org/libfoo` `LIBFOO_SITE=http://svn.xiph.org/trunk/Tremor` `LIBFOO_SITE=/opt/software/libfoo.tar.gz` `LIBFOO_SITE=$(TOPDIR)/../src/libfoo`

- `LIBFOO_DL_OPTS` 是传递给下载器的额外选项（以空格分隔）。适用于需要服务器端登录验证或代理的下载。所有 `LIBFOO_SITE_METHOD` 支持的下载方式都支持该选项，具体可查阅相应下载工具的 man 手册。对于 git，`FOO_DL_OPTS` 只会传递给 `git fetch`，不会传递给其他 git 命令（尤其不会传递给 `git lfs fetch` 或 `git submodule update`）。

- `LIBFOO_EXTRA_DOWNLOADS` 是 Buildroot 需要额外下载的文件列表（以空格分隔）。如果条目中包含 `://`，Buildroot 会认为它是完整 URL 并用该地址下载，否则会从 `LIBFOO_SITE` 下载。Buildroot 只负责下载这些文件，不会做其他处理：需要由包配方在 `$(LIBFOO_DL_DIR)` 中自行使用这些文件。
