## Chapter 10. Integration topics

This chapter discusses how various things are integrated at system level. Buildroot is highly configurable, almost everything discussed here can be changed or overridden by [rootfs overlay or custom skeleton](https://buildroot.org/downloads/manual/manual.html#rootfs-custom) configuration.

## 10.1. Configurable packages

Some foundational packages like Busybox and uClibc can be configured with or without certain features. When writing Buildroot code that uses such packages, contributors may assume that the options enabled in the Buildroot-provided configurations are enabled. For example, `package/busybox/busybox.config` sets `CONFIG_FEATURE_START_STOP_DAEMON_LONG_OPTIONS=y`, so init scripts meant for use with Busybox init may use `start-stop-daemon` with long form options.

People who use custom configurations that disable such default options are responsible for making sure that any relevant scripts/packages still work, and if not, adapting them accordingly. To follow the Busybox example above, disabling long form options will require replacing init scripts that use them (in an overlay).

## 10.2. Systemd

This chapter describes the decisions taken in Buildroot’s integration of systemd, and how various use cases can be implemented.

### 10.2.1. DBus daemon

Systemd requires a DBus daemon. There are two options for it: traditional dbus (`BR2_PACKAGE_DBUS`) and bus1 dbus-broker (`BR2_PACKAGE_DBUS_BROKER`). At least one of them must be chosen. If both are included in the configuration, dbus-broker will be used as system bus, but the traditional dbus-daemon is still installed as well and can be used as session bus. Also its tools (e.g. `dbus-send`) can be used (systemd itself has `busctl` as an alternative). In addition, the traditional dbus package is the only one that provides `libdbus`, which is used by many packages as dbus integration library.

Both in the dbus and in the dbus-broker case, the daemon runs as user `dbus`. The DBus configuration files are also identical for both.

To make sure that only one of the two daemons is started as system bus, the systemd activation files of the dbus package (`dbus.socket` and the `dbus.service` symlink in `multi-user.target.wants`) are removed when dbus-broker is selected.

## 10.3. Using SELinux in Buildroot

[SELinux](https://selinuxproject.org/) is a Linux kernel security module enforcing access control policies. In addition to the traditional file permissions and access control lists, `SELinux` allows to write rules for users or processes to access specific functions of resources (files, sockets…).

*SELinux* has three modes of operation:

- *Disabled*: the policy is not applied
- *Permissive*: the policy is applied, and non-authorized actions are simply logged. This mode is often used for troubleshooting SELinux issues.
- *Enforcing*: the policy is applied, and non-authorized actions are denied

In Buildroot the mode of operation is controlled by the `BR2_PACKAGE_REFPOLICY_POLICY_STATE_*` configuration options. The Linux kernel also has various configuration options that affect how `SELinux` is enabled (see `security/selinux/Kconfig` in the Linux kernel sources).

By default in Buildroot the `SELinux` policy is provided by the upstream [refpolicy](https://github.com/SELinuxProject/refpolicy) project, enabled with `BR2_PACKAGE_REFPOLICY`.

### 10.3.1. Enabling SELinux support

To have proper support for `SELinux` in a Buildroot generated system, the following configuration options must be enabled:

- `BR2_PACKAGE_LIBSELINUX`
- `BR2_PACKAGE_REFPOLICY`

In addition, your filesystem image format must support extended attributes.

### 10.3.2. SELinux policy tweaking

The `SELinux refpolicy` contains modules that can be enabled or disabled when being built. Each module provide a number of `SELinux` rules. In Buildroot the non-base modules are disabled by default and several ways to enable such modules are provided:

- Packages can enable a list of `SELinux` modules within the `refpolicy` using the `<packagename>_SELINUX_MODULES` variable.
- Packages can provide additional `SELinux` modules by putting them (.fc, .if and .te files) in `package/<packagename>/selinux/`.
- Extra `SELinux` modules can be added in directories pointed by the `BR2_REFPOLICY_EXTRA_MODULES_DIRS` configuration option.
- Additional modules in the `refpolicy` can be enabled if listed in the `BR2_REFPOLICY_EXTRA_MODULES_DEPENDENCIES` configuration option.

Buildroot also allows to completely override the `refpolicy`. This allows to provide a full custom policy designed specifically for a given system. When going this way, all of the above mechanisms are disabled: no extra `SElinux` module is added to the policy, and all the available modules within the custom policy are enabled and built into the final binary policy. The custom policy must be a fork of the official [refpolicy](https://github.com/SELinuxProject/refpolicy).

In order to fully override the `refpolicy` the following configuration variables have to be set:

- `BR2_PACKAGE_REFPOLICY_CUSTOM_GIT`
- `BR2_PACKAGE_REFPOLICY_CUSTOM_REPO_URL`
- `BR2_PACKAGE_REFPOLICY_CUSTOM_REPO_VERSION`

