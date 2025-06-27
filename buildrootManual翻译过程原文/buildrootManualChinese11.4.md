- `LIBFOO_CPE_ID_*` 变量是一组用于定义软件包 [CPE 标识符](https://nvd.nist.gov/products/cpe) 的变量。可用变量如下：

  - `LIBFOO_CPE_ID_VALID`，若设为 `YES`，表示以下各变量的默认值均适用，并生成有效的 CPE ID。
  - `LIBFOO_CPE_ID_PREFIX`，指定 CPE 标识符的前缀（前三个字段）。未定义时默认值为 `cpe:2.3:a`。
  - `LIBFOO_CPE_ID_VENDOR`，指定 CPE 标识符的厂商部分。未定义时默认值为 `<pkgname>_project`。
  - `LIBFOO_CPE_ID_PRODUCT`，指定 CPE 标识符的产品部分。未定义时默认值为 `<pkgname>`。
  - `LIBFOO_CPE_ID_VERSION`，指定 CPE 标识符的版本部分。未定义时默认值为 `$(LIBFOO_VERSION)`。
  - `LIBFOO_CPE_ID_UPDATE`，指定 CPE 标识符的 *update* 部分。未定义时默认值为 `*`。

  如果定义了上述任一变量，则通用包基础设施会认为该包提供了有效的 CPE 信息，并据此定义 `LIBFOO_CPE_ID`。

  对于主机包（host package），若未定义其 `LIBFOO_CPE_ID_*` 变量，则会继承对应目标包的变量值。

推荐的定义方式如下：

```
LIBFOO_VERSION = 2.32
```

下面是构建流程各阶段可用的命令变量：

- `LIBFOO_EXTRACT_CMDS` 指定解包操作。一般无需设置，Buildroot 会自动处理 tarball。若包为非标准归档格式（如 ZIP、RAR）或 tarball 结构特殊，可用此变量覆盖默认行为。
- `LIBFOO_CONFIGURE_CMDS` 指定编译前的配置操作。
- `LIBFOO_BUILD_CMDS` 指定编译操作。
- `HOST_LIBFOO_INSTALL_CMDS` 指定主机包的安装操作。包需将所有文件（包括头文件等开发文件）安装到 `$(HOST_DIR)`，以便其他包可在其基础上编译。
- `LIBFOO_INSTALL_TARGET_CMDS` 指定目标包安装到目标目录的操作。包需将运行所需文件安装到 `$(TARGET_DIR)`，头文件、静态库和文档会在最终生成目标文件系统时移除。
- `LIBFOO_INSTALL_STAGING_CMDS` 指定目标包安装到 staging 目录的操作。包需将所有开发文件安装到 `$(STAGING_DIR)`，以便其他包编译时使用。
- `LIBFOO_INSTALL_IMAGES_CMDS` 指定目标包安装到 images 目录的操作。包需将二进制镜像（如内核、引导加载器、根文件系统镜像等）安装到 `$(BINARIES_DIR)`。仅适用于不属于 `TARGET_DIR` 但启动板卡所需的镜像文件。
- `LIBFOO_INSTALL_INIT_SYSV`、`LIBFOO_INSTALL_INIT_OPENRC` 和 `LIBFOO_INSTALL_INIT_SYSTEMD` 分别指定安装 systemV、openrc 或 systemd 启动脚本的操作。仅在配置了对应 init 系统时执行（如配置 systemd，仅执行 `LIBFOO_INSTALL_INIT_SYSTEMD`）。唯一例外是 openrc 作为 init 系统且未设置 `LIBFOO_INSTALL_INIT_OPENRC` 时，会调用 `LIBFOO_INSTALL_INIT_SYSV`，因 openrc 支持 sysv 脚本。使用 systemd 时，buildroot 会在镜像构建最后阶段自动启用所有服务（`systemctl preset-all`），如需禁止某服务自动启用可添加 preset 文件。
- `LIBFOO_HELP_CMDS` 指定打印包帮助信息的操作，会包含在主 `make help` 输出中。该变量极少用，除非确有自定义规则需求。***除非确有需要，否则不要使用此变量。***
- `LIBFOO_LINUX_CONFIG_FIXUPS` 指定构建和使用该包所需的 Linux 内核配置选项，否则包将无法正常工作。应为一组 kconfig 调整操作（如 `KCONFIG_ENABLE_OPT`、`KCONFIG_DISABLE_OPT`、`KCONFIG_SET_OPT`）。该变量极少用。
- `LIBFOO_BUSYBOX_CONFIG_FIXUPS` 指定使用该包（尤其在脚本中）所需或不需要的 Busybox 配置选项。应为一组 kconfig 调整操作。

推荐的定义方式如下：

```
define LIBFOO_CONFIGURE_CMDS
        action 1
        action 2
        action 3
endef
```

在操作定义中，可用如下变量：

- `$(LIBFOO_PKGDIR)`：包含 `libfoo.mk` 和 `Config.in` 的目录路径。用于安装 Buildroot 自带的文件（如运行时配置、开机画面等）。
- `$(@D)`：源码解压目录。
- `$(LIBFOO_DL_DIR)`：Buildroot 为 `libfoo` 下载的所有文件存放目录。
- `$(TARGET_CC)`、`$(TARGET_LD)` 等：目标交叉编译工具。
- `$(TARGET_CROSS)`：交叉编译工具链前缀。
- 当然还有 `$(HOST_DIR)`、`$(STAGING_DIR)`、`$(TARGET_DIR)`，用于正确安装包。这些变量指向全局 *host*、*staging*、*target* 目录，若启用 *per-package directory*，则指向当前包的目录。对包来说无差别，直接使用即可。详见[顶层并行构建](https://buildroot.org/downloads/manual/manual.html#top-level-parallel-build)。

最后，还可以使用钩子（hook）。详见[各构建阶段可用钩子](https://buildroot.org/downloads/manual/manual.html#hooks)。
