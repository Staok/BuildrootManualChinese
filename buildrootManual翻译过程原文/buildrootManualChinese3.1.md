## 8.2 何时需要全量重构

Buildroot 不会尝试自动检测在通过 `make menuconfig`、`make xconfig` 或其他配置工具更改系统配置后，哪些部分需要重建。有时应重建整个系统，有时只需重建部分软件包，但要完全可靠地检测这些情况非常困难，因此 Buildroot 开发者选择不做自动检测。

因此，用户需自行判断何时需要全量重构。以下经验法则可帮助你理解 Buildroot 的工作方式：

- 更改目标体系结构配置时，需全量重构。例如更改架构变体、二进制格式或浮点策略等会影响整个系统。
- 更改工具链配置时，通常也需全量重构。工具链配置变更通常涉及编译器版本、C 库类型或其配置等，这些都会影响整个系统。
- 新增软件包时，不一定需要全量重构。Buildroot 会检测到该包未被构建过并自动构建。但如该包为可被其他已构建包可选依赖的库，Buildroot 不会自动重建这些包。你需手动重建相关包，或全量重构。例如，已构建带 `ctorrent`、不带 `openssl` 的系统，后续启用 `openssl` 并重建，Buildroot 只会构建 `openssl`，不会自动重建 `ctorrent` 以启用 SSL。此时需手动重建 `ctorrent` 或全量重构。
- 移除软件包时，Buildroot 不会做特殊处理。不会从目标根文件系统或工具链 *sysroot* 中移除该包安装的文件。需全量重构才能彻底移除该包。但通常无需立即移除，可等下次完整构建时再处理。
- 更改软件包子选项时，Buildroot 不会自动重建该包。此时通常只需重建该包，除非新选项为其他已构建包提供了新特性。Buildroot 不追踪包的重建依赖：包一旦构建，除非显式指定，否则不会重建。
- 更改根文件系统 skeleton 时，需全量重构。但更改 rootfs overlay、post-build 脚本或 post-image 脚本时，无需全量重构，直接 `make` 即可生效。
- 当 `FOO_DEPENDENCIES` 中的软件包被重建或移除时，Buildroot 不会自动重建依赖该包的 `foo` 包。例如 `FOO_DEPENDENCIES = bar`，如更改 `bar` 配置，`foo` 不会自动重建。此时需手动重建所有依赖 `bar` 的包，或全量重构以确保依赖关系正确。

一般来说，如遇构建错误且不确定配置更改的影响，建议全量重构。若错误依旧，则可确定与部分重建无关。如为官方 Buildroot 包，欢迎报告问题！随着经验积累，你会逐步掌握何时需全量重构，从而节省时间。

全量重构命令如下：

```
$ make clean all
```

## 8.3 如何重建软件包

Buildroot 用户常问如何重建某个软件包或在不全量重构的情况下移除软件包。

Buildroot 不支持在不全量重构的情况下移除软件包。因为 Buildroot 不追踪各包在 `output/staging` 和 `output/target` 目录下安装了哪些文件，也不追踪哪些包会因其他包的可用性而编译出不同内容。

重建单个软件包最简单的方法是删除其在 `output/build` 下的构建目录。Buildroot 会重新解包、配置、编译并安装该包。可用 `make <package>-dirclean` 命令自动完成。

如只需从编译步骤重新构建包，可用 `make <package>-rebuild`，会重新编译和安装，但不会从头开始，仅重新构建有变更的文件。

如需从配置步骤重新构建包，可用 `make <package>-reconfigure`，会重新配置、编译和安装。

`<package>-rebuild` 隐含 `<package>-reinstall`，`<package>-reconfigure` 隐含 `<package>-rebuild`，这些目标及 `<package>` 只作用于该包本身，不会自动重建根文件系统镜像。如需重建根文件系统，需额外运行 `make` 或 `make all`。

Buildroot 内部用*标记文件（stamp files）*记录每个包的构建步骤，存于 `output/build/<package>-<version>/`，文件名为 `.stamp_<step-name>`。上述命令通过操作这些标记文件，强制 Buildroot 重新执行包的特定构建步骤。

更多包专用 make 目标详见[第 8.13.5 节 包专用 make 目标](https://buildroot.org/downloads/manual/manual.html#pkg-build-steps)。

## 8.4 离线构建

如需离线构建，仅想下载配置器（*menuconfig*、*nconfig*、*xconfig* 或 *gconfig*）中已选软件包的所有源码，可运行：

```
 $ make source
```

此后即可断网或将 `dl` 目录内容拷贝到构建主机。
