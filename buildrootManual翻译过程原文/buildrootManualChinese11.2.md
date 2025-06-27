- `LIBFOO_SITE_METHOD` 用于指定获取或复制软件包源码的方法。在大多数情况下，Buildroot 会根据 `LIBFOO_SITE` 的内容自动判断方法，无需手动设置 `LIBFOO_SITE_METHOD`。如果未指定 `HOST_LIBFOO_SITE_METHOD`，则默认与 `LIBFOO_SITE_METHOD` 相同。`LIBFOO_SITE_METHOD` 可选值如下：

  - `wget`：用于通过 FTP/HTTP 下载 tarball。当 `LIBFOO_SITE` 以 `http://`、`https://` 或 `ftp://` 开头时默认使用。
  - `scp`：通过 SSH 的 scp 下载 tarball。当 `LIBFOO_SITE` 以 `scp://` 开头时默认使用。
  - `sftp`：通过 SSH 的 sftp 下载 tarball。当 `LIBFOO_SITE` 以 `sftp://` 开头时默认使用。
  - `svn`：从 Subversion 仓库获取源码。当 `LIBFOO_SITE` 以 `svn://` 开头时默认使用。如果 `LIBFOO_SITE` 指定了 `http://` 的 Subversion 仓库 URL，*必须*指定 `LIBFOO_SITE_METHOD=svn`。Buildroot 会执行 checkout 并将其保存为 tarball，后续构建会使用该 tarball 而不是重新 checkout。
  - `cvs`：从 CVS 仓库获取源码。当 `LIBFOO_SITE` 以 `cvs://` 开头时默认使用。下载的源码同样会被缓存为 tarball。默认使用匿名 pserver 模式，除非在 `LIBFOO_SITE` 中显式指定。`LIBFOO_SITE` *必须*包含源码 URL 及远程仓库目录，模块名为包名。`LIBFOO_VERSION` *必须*为标签、分支或日期（如 "2014-10-20"，详见 man cvs）。
  - `git`：从 Git 仓库获取源码。当 `LIBFOO_SITE` 以 `git://` 开头时默认使用。源码同样会被缓存为 tarball。
  - `hg`：从 Mercurial 仓库获取源码。当 `LIBFOO_SITE` 包含 Mercurial 仓库 URL 时，*必须*指定 `LIBFOO_SITE_METHOD=hg`。源码同样会被缓存为 tarball。
  - `bzr`：从 Bazaar 仓库获取源码。当 `LIBFOO_SITE` 以 `bzr://` 开头时默认使用。源码同样会被缓存为 tarball。
  - `file`：本地 tarball。当 `LIBFOO_SITE` 指定为本地文件名时应使用。适用于未公开发布或无版本控制的软件。
  - `local`：本地源码目录。当 `LIBFOO_SITE` 指定为本地源码目录时应使用。Buildroot 会将源码目录内容复制到包的构建目录。注意，`local` 包不会应用补丁。如需补丁可用 `LIBFOO_POST_RSYNC_HOOKS`，详见[POST_RSYNC 钩子用法](https://buildroot.org/downloads/manual/manual.html#hooks-rsync)。
  - `smb`：从 SMB 共享获取源码。当 `LIBFOO_SITE` 以 `smb://` 开头时默认使用，使用 `curl` 作为下载后端。语法：`LIBFOO_SITE=smb://<server>/<share>/<path>`。可能需要在 `LIBFOO_DL_OPTS` 中定义 -u 选项，详见 [curl 文档](https://curl.se/docs/tutorial.html)。

- `LIBFOO_GIT_SUBMODULES` 可设为 `YES`，表示为仓库中的 git 子模块创建归档，仅适用于 git 下载的软件包（即 `LIBFOO_SITE_METHOD=git`）。注意，如果子模块包含内置库，建议单独打包而非使用子模块。

- `LIBFOO_GIT_LFS` 若仓库使用 Git LFS 存储大文件，需设为 `YES`，仅适用于 git 下载的软件包。

- `LIBFOO_SVN_EXTERNALS` 可设为 `YES`，表示为 svn 外部引用创建归档，仅适用于 subversion 下载的软件包。

- `LIBFOO_STRIP_COMPONENTS` 指定 tar 解包时要去除的前导目录层数。大多数包的 tarball 有一级 `<pkg-name>-<pkg-version>` 目录，Buildroot 默认传递 --strip-components=1。若包结构不同，可设置此变量。默认值：1。

- `LIBFOO_EXCLUDES` 是解包时要排除的模式列表（以空格分隔），每项作为 tar 的 `--exclude` 选项。默认空。

- `LIBFOO_DEPENDENCIES` 指定当前目标包编译所需的依赖包（以包名表示）。这些依赖会在当前包配置前编译和安装。但依赖包配置变更不会强制重建当前包。`HOST_LIBFOO_DEPENDENCIES` 以类似方式指定主机包依赖。

- `LIBFOO_EXTRACT_DEPENDENCIES` 指定当前目标包解包前所需依赖（以包名表示）。这些依赖会在当前包解包前编译和安装。该变量仅供包基础设施内部使用，通常不建议包直接使用。

- `LIBFOO_PATCH_DEPENDENCIES` 指定当前包打补丁前所需依赖（以包名表示）。这些依赖会在当前包打补丁前被解包和打补丁（但不一定编译）。`HOST_LIBFOO_PATCH_DEPENDENCIES` 以类似方式指定主机包依赖。该变量很少用，通常应使用 `LIBFOO_DEPENDENCIES`。

- `LIBFOO_PROVIDES` 列出 `libfoo` 实现的所有虚拟包。详见[虚拟包基础设施](https://buildroot.org/downloads/manual/manual.html#virtual-package-tutorial)。

- `LIBFOO_INSTALL_STAGING` 可设为 `YES` 或 `NO`（默认）。若为 `YES`，则会执行 `LIBFOO_INSTALL_STAGING_CMDS` 变量中的命令，将包安装到 staging 目录。

- `LIBFOO_INSTALL_TARGET` 可设为 `YES`（默认）或 `NO`。若为 `YES`，则会执行 `LIBFOO_INSTALL_TARGET_CMDS` 变量中的命令，将包安装到目标目录。

- `LIBFOO_INSTALL_IMAGES` 可设为 `YES` 或 `NO`（默认）。若为 `YES`，则会执行 `LIBFOO_INSTALL_IMAGES_CMDS` 变量中的命令，将包安装到 images 目录。

- `LIBFOO_CONFIG_SCRIPTS` 列出 *$(STAGING_DIR)/usr/bin* 下需要特殊修正以支持交叉编译的文件名。可用空格分隔多个文件名，均为相对路径。`LIBFOO_CONFIG_SCRIPTS` 中的文件也会从 `$(TARGET_DIR)/usr/bin` 移除，因为目标系统不需要。

- `LIBFOO_DEVICES` 列出使用静态设备表时 Buildroot 需创建的设备文件，语法采用 makedevs 格式，详见[Makedev 语法文档](https://buildroot.org/downloads/manual/manual.html#makedev-syntax)。该变量为可选项。

- `LIBFOO_PERMISSIONS` 列出构建结束后需更改权限的文件，语法同样采用 makedevs 格式，详见[Makedev 语法文档](https://buildroot.org/downloads/manual/manual.html#makedev-syntax)。该变量为可选项。
