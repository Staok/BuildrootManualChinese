## 18.12. 虚拟软件包基础设施

在Buildroot中，虚拟软件包（virtual package）是指其功能由一个或多个包（称为“提供者”provider）实现的软件包。虚拟包管理是一种可扩展机制，允许用户选择在rootfs中使用的提供者。

例如，OpenGL ES是嵌入式系统上用于2D和3D图形的API。该API的实现因平台不同而异，如Allwinner Tech Sunxi和Texas Instruments OMAP35xx平台。因此，`libgles` 会是一个虚拟包，而 `sunxi-mali-utgard` 和 `ti-gfx` 则是其提供者。

### 18.12.1. `virtual-package` 教程

以下示例将说明如何添加一个新的虚拟包（something-virtual）及其一个提供者（some-provider）。

首先，创建虚拟包。

### 18.12.2. 虚拟包的 `Config.in` 文件

虚拟包 something-virtual 的 `Config.in` 文件应包含：

```
01: config BR2_PACKAGE_HAS_SOMETHING_VIRTUAL
02:     bool
03:
04: config BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL
05:     depends on BR2_PACKAGE_HAS_SOMETHING_VIRTUAL
06:     string
```

在该文件中，声明了两个选项，`BR2_PACKAGE_HAS_SOMETHING_VIRTUAL` 和 `BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL`，其值将被提供者使用。

### 18.12.3. 虚拟包的 `.mk` 文件

虚拟包的 `.mk` 文件只需调用 `virtual-package` 宏：

```
01: ################################################################################
02: #
03: # something-virtual
04: #
05: ################################################################################
06:
07: $(eval $(virtual-package))
```

同样支持目标和主机包，可使用 `host-virtual-package` 宏。

### 18.12.4. 提供者的 `Config.in` 文件

添加包作为提供者时，仅需修改其 `Config.in` 文件。

提供 something-virtual 功能的包 some-provider 的 `Config.in` 文件应包含：

```
01: config BR2_PACKAGE_SOME_PROVIDER
02:     bool "some-provider"
03:     select BR2_PACKAGE_HAS_SOMETHING_VIRTUAL
04:     help
05:       这里是对some-provider的说明注释。
06:
07:       http://foosoftware.org/some-provider/
08:
09: if BR2_PACKAGE_SOME_PROVIDER
10: config BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL
11:     default "some-provider"
12: endif
```

第3行选择了 `BR2_PACKAGE_HAS_SOMETHING_VIRTUAL`，第11行在被选中时将 `BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL` 的值设为提供者名称。

### 18.12.5. 提供者的 `.mk` 文件

`.mk` 文件还应声明一个额外变量 `SOME_PROVIDER_PROVIDES`，包含其实现的所有虚拟包名称：

```
01: SOME_PROVIDER_PROVIDES = something-virtual
```

当然，不要忘记为该包添加正确的构建和运行时依赖！

### 18.12.6. 关于依赖虚拟包的说明

当添加一个需要某个虚拟包提供的 `FEATURE` 的包时，应使用 `depends on BR2_PACKAGE_HAS_FEATURE`，例如：

```
config BR2_PACKAGE_HAS_FEATURE
    bool

config BR2_PACKAGE_FOO
    bool "foo"
    depends on BR2_PACKAGE_HAS_FEATURE
```

### 18.12.7. 关于依赖特定提供者的说明

如果你的包确实需要特定的提供者，则必须让你的包 `depends on` 该提供者；不能用 `select` 选择提供者。

以有两个 `FEATURE` 提供者为例：

```
config BR2_PACKAGE_HAS_FEATURE
    bool

config BR2_PACKAGE_FOO
    bool "foo"
    select BR2_PACKAGE_HAS_FEATURE

config BR2_PACKAGE_BAR
    bool "bar"
    select BR2_PACKAGE_HAS_FEATURE
```

如果你要添加一个只需要 `foo` 提供的 `FEATURE` 的包，而不需要 `bar`，

如果用 `select BR2_PACKAGE_FOO`，用户仍然可以在menuconfig中选择 `BR2_PACKAGE_BAR`，这会导致配置不一致，即同一 `FEATURE` 的两个提供者同时被启用，一个是用户显式设置，另一个是你的 `select` 隐式设置。

因此，必须用 `depends on BR2_PACKAGE_FOO`，以避免隐式配置不一致。

## 18.13. 使用kconfig进行配置的软件包基础设施

许多软件包采用 `kconfig` 进行用户配置，如Linux内核、Busybox和Buildroot本身。`.config` 文件和 `menuconfig` 目标是kconfig被使用的典型特征。

Buildroot为使用kconfig进行配置的软件包提供了基础设施。该基础设施提供了必要的逻辑，将包的 `menuconfig` 目标暴露为Buildroot中的 `foo-menuconfig`，并正确处理配置文件的复制。

kconfig软件包基础设施的主要宏是 `kconfig-package`，与 `generic-package` 宏类似。

与通用基础设施类似，kconfig基础设施通过在调用 `kconfig-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在kconfig基础设施中同样适用。

要为Buildroot包使用 `kconfig-package` 基础设施，除了 `generic-package` 所需变量外，`.mk` 文件中至少要有：

```
FOO_KCONFIG_FILE = reference-to-source-configuration-file

$(eval $(kconfig-package))
```

该片段会创建如下make目标：

- `foo-menuconfig`，调用包的 `menuconfig` 目标
- `foo-update-config`，将配置复制回源配置文件（有fragment文件时不可用）
- `foo-update-defconfig`，将配置以只包含与默认值不同的选项的方式复制回源配置文件（有fragment文件时不可用）
- `foo-diff-config`，输出当前配置与Buildroot配置中该kconfig包定义的配置的差异，便于识别需要同步到配置片段的更改

并确保在合适时机将源配置文件复制到构建目录。

有两种方式指定要使用的配置文件：`FOO_KCONFIG_FILE`（如上例）或 `FOO_KCONFIG_DEFCONFIG`，必须提供其一：

- `FOO_KCONFIG_FILE` 指定要用于配置包的defconfig或完整配置文件的路径
- `FOO_KCONFIG_DEFCONFIG` 指定用于配置包的defconfig *make* 规则

此外，还可根据包需求设置如下可选变量：

- `FOO_KCONFIG_EDITORS`：支持的kconfig编辑器列表，默认*menuconfig*
- `FOO_KCONFIG_FRAGMENT_FILES`：要合并到主配置文件的配置片段文件列表
- `FOO_KCONFIG_OPTS`：调用kconfig编辑器时的额外选项，可能需包含*$(FOO_MAKE_OPTS)*，默认空
- `FOO_KCONFIG_FIXUP_CMDS`：复制配置文件或运行kconfig编辑器后需执行的shell命令列表，默认空
- `FOO_KCONFIG_DOTCONFIG`：`.config` 文件的路径（含文件名），相对于包源码树，默认`.config`
- `FOO_KCONFIG_DEPENDENCIES`：kconfig解析前需构建的包（多为host包）列表，默认空
- `FOO_KCONFIG_SUPPORTS_DEFCONFIG`：包的kconfig系统是否支持defconfig文件，默认*YES*
