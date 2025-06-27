## 第10章 集成主题

本章讨论了在系统层面上如何集成各种内容。Buildroot 具有高度可配置性，这里讨论的几乎所有内容都可以通过 [rootfs 覆盖或自定义 skeleton](https://buildroot.org/downloads/manual/manual.html#rootfs-custom) 配置进行更改或覆盖。

## 10.1 可配置的软件包（Configurable packages）

一些基础软件包，如 Busybox 和 uClibc，可以根据需要启用或禁用某些功能。在编写使用这些软件包的 Buildroot 代码时，贡献者可以假设 Buildroot 提供的配置中已启用相关选项。例如，`package/busybox/busybox.config` 设置了 `CONFIG_FEATURE_START_STOP_DAEMON_LONG_OPTIONS=y`，因此用于 Busybox init 的初始化脚本可以使用带有长选项的 `start-stop-daemon`。

如果用户使用自定义配置禁用了这些默认选项，则需要自行确保相关脚本/软件包仍然可用，如果不可用，则需要相应地进行调整。以 Busybox 为例，禁用长选项后，需要替换使用长选项的初始化脚本（可通过 overlay 覆盖）。

## 10.2 Systemd

本节描述了 Buildroot 集成 systemd 时所做的决策，以及如何实现各种用例。

### 10.2.1 DBus 守护进程（DBus daemon）

Systemd 需要一个 DBus 守护进程。有两种选择：传统 dbus（`BR2_PACKAGE_DBUS`）和 bus1 dbus-broker（`BR2_PACKAGE_DBUS_BROKER`）。必须至少选择其中之一。如果两者都包含在配置中，则 dbus-broker 将作为系统总线（system bus）使用，但传统的 dbus-daemon 仍会被安装，并可作为会话总线（session bus）使用。同时，其工具（如 `dbus-send`）也可用（systemd 本身有 `busctl` 作为替代）。此外，传统 dbus 软件包是唯一提供 `libdbus` 的软件包，许多软件包将其用作 dbus 集成库。

无论是 dbus 还是 dbus-broker，守护进程都以 `dbus` 用户身份运行。两者的 DBus 配置文件也完全相同。

为确保只有一个守护进程作为系统总线启动，当选择 dbus-broker 时，会移除 dbus 软件包的 systemd 激活文件（`dbus.socket` 和 `multi-user.target.wants` 目录下的 `dbus.service` 符号链接）。

## 10.3 在 Buildroot 中使用 SELinux

[SELinux](https://selinuxproject.org/) 是一个 Linux 内核安全模块，用于强制访问控制策略。除了传统的文件权限和访问控制列表外，`SELinux` 还允许为用户或进程访问特定资源（文件、套接字等）的功能编写规则。

*SELinux* 有三种运行模式：

- *禁用*（Disabled）：不应用策略
- *宽容*（Permissive）：应用策略，未授权操作仅被记录。此模式常用于 SELinux 问题的排查。
- *强制*（Enforcing）：应用策略，未授权操作被拒绝

在 Buildroot 中，运行模式由 `BR2_PACKAGE_REFPOLICY_POLICY_STATE_*` 配置选项控制。Linux 内核也有多个配置选项影响 `SELinux` 的启用方式（参见 Linux 内核源码中的 `security/selinux/Kconfig`）。

默认情况下，Buildroot 提供的 `SELinux` 策略来自上游 [refpolicy](https://github.com/SELinuxProject/refpolicy) 项目，通过启用 `BR2_PACKAGE_REFPOLICY` 实现。

### 10.3.1 启用 SELinux 支持

要在 Buildroot 生成的系统中正确支持 `SELinux`，必须启用以下配置选项：

- `BR2_PACKAGE_LIBSELINUX`
- `BR2_PACKAGE_REFPOLICY`

此外，文件系统镜像格式必须支持扩展属性（extended attributes）。

### 10.3.2 SELinux 策略调整（policy tweaking）

`SELinux refpolicy` 包含可在构建时启用或禁用的模块。每个模块都提供若干 `SELinux` 规则。在 Buildroot 中，非基础模块默认禁用，提供了多种方式启用这些模块：

- 软件包可通过 `<packagename>_SELINUX_MODULES` 变量，在 `refpolicy` 中启用一组 `SELinux` 模块。
- 软件包可通过在 `package/<packagename>/selinux/` 目录下放置 .fc、.if 和 .te 文件，提供额外的 `SELinux` 模块。
- 可通过 `BR2_REFPOLICY_EXTRA_MODULES_DIRS` 配置选项，添加额外的 `SELinux` 模块目录。
- 可通过 `BR2_REFPOLICY_EXTRA_MODULES_DEPENDENCIES` 配置选项，启用 `refpolicy` 中的其他模块。

Buildroot 还允许完全覆盖 `refpolicy`，即为特定系统提供完全自定义的策略。采用此方式时，上述所有机制均被禁用：不会向策略中添加额外的 `SElinux` 模块，且自定义策略中的所有可用模块都会被启用并编译进最终的二进制策略。自定义策略必须是官方 [refpolicy](https://github.com/SELinuxProject/refpolicy) 的分支。

要完全覆盖 `refpolicy`，需设置以下配置变量：

- `BR2_PACKAGE_REFPOLICY_CUSTOM_GIT`
- `BR2_PACKAGE_REFPOLICY_CUSTOM_REPO_URL`
- `BR2_PACKAGE_REFPOLICY_CUSTOM_REPO_VERSION`
