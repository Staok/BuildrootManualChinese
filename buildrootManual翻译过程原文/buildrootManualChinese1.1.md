## 第 4 章 Buildroot 快速入门

***重要提示***：你可以且应当***以普通用户身份构建所有内容***。配置和使用 Buildroot 无需 root 权限。以普通用户身份运行所有命令，可以防止在编译和安装过程中因软件包异常行为而影响你的系统。

使用 Buildroot 的第一步是创建配置。Buildroot 提供了类似于 [Linux 内核](http://www.kernel.org/) 或 [BusyBox](http://www.busybox.net/) 的友好配置工具。

在 buildroot 目录下，运行：

```
 $ make menuconfig
```

以启动原始的基于 curses 的配置器，或

```
 $ make nconfig
```

以启动新的基于 curses 的配置器，或

```
 $ make xconfig
```

以启动基于 Qt 的配置器，或

```
 $ make gconfig
```

以启动基于 GTK 的配置器。

所有这些 "make" 命令都需要构建配置工具（包括界面），因此你可能需要为相关库安装“开发”软件包。详细信息请参见[第 2 章 系统要求](https://buildroot.org/downloads/manual/manual.html#requirement)，特别是[可选依赖](https://buildroot.org/downloads/manual/manual.html#requirement-optional)部分，以获取你喜欢的界面所需依赖。

在配置工具的每个菜单项中，你都可以找到相关帮助，说明该项的用途。关于部分具体配置内容，详见[第 6 章 Buildroot 配置](https://buildroot.org/downloads/manual/manual.html#configure)。

配置完成后，配置工具会生成一个 `.config` 文件，包含全部配置信息。该文件将被顶层 Makefile 读取。

要开始构建流程，只需运行：

```
 $ make
```

默认情况下，Buildroot 不支持顶层并行构建（top-level parallel build），因此无需使用 `make -jN`。不过，Buildroot 提供了实验性的顶层并行构建支持，详见[第 8.12 节 “顶层并行构建”](https://buildroot.org/downloads/manual/manual.html#top-level-parallel-build)。

`make` 命令通常会执行以下步骤：

- 下载所需源码文件；
- 配置、构建并安装交叉编译工具链，或直接导入外部工具链；
- 配置、构建并安装所选目标软件包；
- 构建内核镜像（如已选择）；
- 构建引导加载程序镜像（如已选择）；
- 以所选格式创建根文件系统。

Buildroot 的输出统一存放在 `output/` 目录下。该目录包含若干子目录：

- `images/`：存放所有镜像（内核镜像、引导加载程序和根文件系统镜像）。这些文件需要烧录到目标系统。
- `build/`：存放所有构建组件（包括 Buildroot 在主机上需要的工具和为目标编译的软件包）。该目录下每个组件对应一个子目录。
- `host/`：包含为主机构建的工具和目标工具链的 sysroot。前者是为主机编译、Buildroot 正常运行所需的工具（包括交叉编译工具链）；后者结构类似根文件系统，包含所有用户空间软件包的头文件和库（供其他软件包依赖）。但该目录*不*应作为目标设备的根文件系统：其中包含大量开发文件、未剥离（unstripped）二进制和库，体积过大，仅供交叉编译时依赖。
- `staging/`：指向 `host/` 目录下目标工具链 sysroot 的符号链接，仅为兼容历史保留。
- `target/`：包含*几乎*完整的目标根文件系统：除 `/dev/` 下的设备文件（Buildroot 不能创建，因为不以 root 运行且不建议以 root 运行）和部分权限（如 busybox 二进制的 setuid 权限）外，其他内容齐全。因此，***该目录不应直接用于目标设备***。应使用 `images/` 目录下生成的镜像文件。如需用于 NFS 启动的根文件系统解包镜像，请使用 `images/` 目录下生成的 tar 包，并以 root 权限解包。与 `staging/` 不同，`target/` 仅包含运行所选目标应用所需的文件和库：不含开发文件（头文件等），二进制已剥离。

这些命令 `make menuconfig|nconfig|gconfig|xconfig` 和 `make` 是最基本的命令，可帮助你快速生成满足需求的镜像，包含你启用的所有功能和应用。

关于 "make" 命令的更多用法，详见[第 8.1 节 “make 使用技巧”](https://buildroot.org/downloads/manual/manual.html#make-tips)。

## 第 5 章 社区资源

与所有开源项目一样，Buildroot 社区和外部有多种信息交流方式。

如果你需要帮助、想了解 Buildroot 或希望为项目做贡献，这些方式都值得关注。

- 邮件列表（Mailing List）

  Buildroot 拥有用于讨论和开发的邮件列表，是 Buildroot 用户和开发者的主要交流方式。只有订阅者才能向该列表发帖。你可以通过[邮件列表信息页面](http://lists.buildroot.org/mailman/listinfo/buildroot)订阅。发送到邮件列表的邮件也会被归档，可通过 [Mailman](http://lists.buildroot.org/pipermail/buildroot) 或 [lore.kernel.org](https://lore.kernel.org/buildroot/) 查看。

- IRC

  Buildroot 的 IRC 频道 [#buildroot](irc://irc.oftc.net/#buildroot) 托管在 [OFTC](https://www.oftc.net/WebChat/) 上。这里适合快速提问或讨论特定话题。提问时请使用 https://paste.ack.tf/ 等代码分享网站粘贴相关日志或代码片段。对于某些问题，邮件列表可能更合适，因为能覆盖更多开发者和用户。

- Bug 跟踪器（Bug tracker）

  Buildroot 的 bug 可通过邮件列表或 [Buildroot bugtracker](https://gitlab.com/buildroot.org/buildroot/-/issues) 报告。请在提交 bug 报告前参见[第 22.6 节 “报告问题/bug 或获取帮助”](https://buildroot.org/downloads/manual/manual.html#reporting-bugs)。

- Wiki

  [Buildroot wiki 页面](http://elinux.org/Buildroot) 托管在 [eLinux](http://elinux.org/) wiki 上，包含有用链接、往届和即将举行的活动概览及 TODO 列表。

- Patchwork

  Patchwork 是一个基于 Web 的补丁跟踪系统，便于开源项目的补丁贡献和管理。发送到邮件列表的补丁会被系统“捕获”，并显示在网页上，相关评论也会附在补丁页面。更多 Patchwork 信息见 http://jk.ozlabs.org/projects/patchwork/。Buildroot 的 Patchwork 网站主要供维护者确保补丁不被遗漏，也供补丁审核者使用（参见[第 22.3.1 节 “从 Patchwork 应用补丁”](https://buildroot.org/downloads/manual/manual.html#apply-patches-patchwork)）。此外，该网站以简洁明了的界面公开补丁及其评论，对所有 Buildroot 开发者都很有用。Buildroot 补丁管理界面见 https://patchwork.ozlabs.org/project/buildroot/list/。
