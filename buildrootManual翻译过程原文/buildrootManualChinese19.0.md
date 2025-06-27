## 22.7 使用运行时测试框架（runtime tests framework）

Buildroot 包含一个基于 Python 脚本和 QEMU 运行时执行的运行时测试框架。该框架的目标如下：

- 构建一个定义明确的 Buildroot 配置
- （可选）验证构建输出的某些属性
- （可选）在 Qemu 下启动构建结果，并验证某个功能是否按预期工作

使用运行时测试框架的入口是 `support/testing/run-tests` 工具，其参数可通过 *-h* 查看帮助文档。常用选项包括设置下载文件夹、输出文件夹、保留构建输出，以及为多个测试用例设置 JLEVEL。

以下是运行测试用例的示例流程：

- 第一步，查看所有测试用例选项。可通过执行 `support/testing/run-tests -l` 列出所有测试用例。这些测试可在测试开发期间单独运行，也可选择性地批量运行。

```
$ support/testing/run-tests -l
List of tests
test_run (tests.utils.test_check_package.TestCheckPackage)
test_run (tests.toolchain.test_external.TestExternalToolchainBuildrootMusl) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainBuildrootuClibc) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainCCache) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainCtngMusl) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainLinaroArm) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainSourceryArmv4) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainSourceryArmv5) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainSourceryArmv7) ... ok
[省略]
test_run (tests.init.test_systemd.TestInitSystemSystemdRoFull) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRoIfupdown) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRoNetworkd) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRwFull) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRwIfupdown) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRwNetworkd) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRo) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRoNet) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRw) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRwNet) ... ok

Ran 157 tests in 0.021s

OK
```

- 然后，运行单个测试用例：

```
$ support/testing/run-tests -d dl -o output_folder -k tests.init.test_busybox.TestInitSystemBusyboxRw
15:03:26 TestInitSystemBusyboxRw                  Starting
15:03:28 TestInitSystemBusyboxRw                  Building
15:08:18 TestInitSystemBusyboxRw                  Building done
15:08:27 TestInitSystemBusyboxRw                  Cleaning up
.
Ran 1 test in 301.140s

OK
```

标准输出会显示测试是否成功。默认情况下，测试的输出文件夹会自动删除，除非使用 `-k` 选项***保留***输出目录。

### 22.7.1 创建测试用例

在 Buildroot 仓库中，测试框架在 `support/testing/` 顶层目录下分为 `conf`、`infra` 和 `tests` 文件夹。所有测试用例都在 `tests` 文件夹下，按测试类别分文件夹组织。

熟悉如何创建测试用例的最佳方式是查看一些基础文件系统测试（`support/testing/tests/fs/`）和 init 测试（`support/testing/tests/init/`）脚本。这些测试很好地展示了如何既检查构建结果，又进行运行时测试。还有更高级的用例会用到嵌套的 `br2-external` 文件夹来提供 skeleton 和额外软件包。

创建基础测试用例包括：

- 定义一个继承自 `infra.basetest.BRTest` 的测试类
- 定义测试类的 `config` 成员，指定本测试用例要构建的 Buildroot 配置。可选地，依赖运行时测试基础设施提供的配置片段：`infra.basetest.BASIC_TOOLCHAIN_CONFIG`（获取基础架构/工具链配置）和 `infra.basetest.MINIMAL_CONFIG`（不构建任何文件系统）。使用 `infra.basetest.BASIC_TOOLCHAIN_CONFIG` 的好处是会提供匹配的 Linux 内核镜像，可以直接在 Qemu 启动生成的镜像，无需在测试用例中构建内核镜像，从而大大减少测试用例所需的构建时间。
- 实现 `def test_run(self):` 方法，编写构建完成后要执行的实际测试。可以是验证构建输出的测试（通过 `run_cmd_on_host()` 辅助函数在主机上运行命令），也可以是通过 Qemu 启动系统并在 Qemu 内部运行命令（通过测试用例中的 `self.emulator` 对象）。例如，`self.emulator.boot()` 启动 Qemu，`self.emulator.login()` 登录，`self.emulator.run()` 在 Qemu 内部运行 shell 命令。

创建测试脚本后，请将自己添加到 `DEVELOPERS` 文件，成为该测试用例的维护者。

### 22.7.2 调试测试用例

测试用例运行时，`output_folder` 会包含如下内容：

```
$ ls output_folder/
TestInitSystemBusyboxRw/
TestInitSystemBusyboxRw-build.log
TestInitSystemBusyboxRw-run.log
```

`TestInitSystemBusyboxRw/` 是 Buildroot 输出目录，仅在使用 `-k` 选项时保留。

`TestInitSystemBusyboxRw-build.log` 是 Buildroot 构建日志。

`TestInitSystemBusyboxRw-run.log` 是 Qemu 启动和测试日志。仅当构建成功且测试用例涉及 Qemu 启动时才会生成。

如需手动运行 Qemu 进行调试，可参考 `TestInitSystemBusyboxRw-run.log` 前几行中的 Qemu 命令行。

你还可以在 `output_folder` 内修改当前源码（如调试用），然后重新运行标准 Buildroot make 目标（以重新生成完整镜像），再重新运行测试。

### 22.7.3 运行时测试与 Gitlab CI

所有运行时测试都会定期由 Buildroot 的 Gitlab CI 基础设施执行，详见 .gitlab.yml 和 https://gitlab.com/buildroot.org/buildroot/-/jobs。

你也可以用 Gitlab CI 测试你新增的测试用例，或在修改 Buildroot 后验证现有测试是否仍然通过。

为此，你需要在 Gitlab 上 fork Buildroot 项目，并能向你的 Buildroot fork 推送分支。

你推送的分支名决定是否触发 Gitlab CI pipeline，以及触发哪些测试用例。

以下示例中，<name> 是你自定义的字符串。

- 触发所有 run-test 测试用例作业，推送以 `-runtime-tests` 结尾的分支：

```
 $ git push gitlab HEAD:<name>-runtime-tests
```

- 触发一个或多个测试用例作业，推送以完整测试用例名（如 `tests.init.test_busybox.TestInitSystemBusyboxRo`）或测试类别名（如 `tests.init.test_busybox`）结尾的分支：

```
 $ git push gitlab HEAD:<name>-<test case name>
```

示例，运行单个测试：

```
 $ git push gitlab HEAD:foo-tests.init.test_busybox.TestInitSystemBusyboxRo
```

示例，运行同一组的多个测试：

```
 $ git push gitlab HEAD:foo-tests.init.test_busybox
 $ git push gitlab HEAD:foo-tests.init
```

------

[[4\] ](https://buildroot.org/downloads/manual/manual.html#idm6107)RFC:（Request for comments，变更提案）

## 第二十三章 DEVELOPERS 文件与 get-developers

Buildroot 主目录下有一个名为 `DEVELOPERS` 的文件，列出了参与 Buildroot 各领域的开发者。借助该文件，`get-developers` 工具可以：

- 通过解析补丁并匹配被修改文件与相关开发者，计算补丁应发送给哪些开发者。详见[22.5节，提交补丁（Submitting patches）](https://buildroot.org/downloads/manual/manual.html#submitting-patches)。
- 查找负责某个架构或软件包的开发者，以便在该架构或软件包构建失败时通知他们。这与 Buildroot 的自动构建基础设施配合完成。

我们要求添加新软件包、新板卡或新功能的开发者，在 `DEVELOPERS` 文件中注册自己。例如，贡献新软件包的开发者应在补丁中包含对 `DEVELOPERS` 文件的相应修改。

`DEVELOPERS` 文件的格式在文件内部有详细说明。

`get-developers` 工具位于 `utils/` 目录下，可用于多种任务：

- 命令行传入一个或多个补丁时，`get-developers` 会返回合适的 `git send-email` 命令。如加 `-e` 选项，则只输出适合 `git send-email --cc-cmd` 的邮件地址。
- 用 `-a <arch>` 参数可返回负责指定架构的开发者列表。
- 用 `-p <package>` 参数可返回负责指定软件包的开发者列表。
- 用 `-c` 参数会检查 Buildroot 仓库下所有版本控制文件，列出未被任何开发者负责的文件，便于补全 `DEVELOPERS` 文件。
- 用 `-v` 参数可校验 DEVELOPERS 文件的完整性，并对不匹配项给出警告。

## 第二十四章 发布工程（Release Engineering）

### 24.1 发布（Releases）

Buildroot 项目每季度发布一次正式版，每月发布一次 bug 修复版。每年首个正式版为长期支持版（LTS，Long Term Support）。

- 季度发布：2020.02、2020.05、2020.08、2020.11
- bug 修复发布：2020.02.1、2020.02.2、……
- LTS 发布：2020.02、2021.02、……

每个正式版支持到下一个正式版的首个 bug 修复版发布为止，例如 2020.05.x 在 2020.08.1 发布时停止维护（EOL，End of Life）。

LTS 版支持到下一个 LTS 的首个 bug 修复版发布为止，例如 2020.02.x 支持到 2021.02.1 发布为止。

### 24.2 开发（Development）

每个发布周期包括两个月的 `master` 分支开发和一个月的稳定期（stabilization），之后发布正式版。在稳定期内，`master` 分支不再添加新特性，只接受 bug 修复。

稳定期开始时会打 `-rc1` 标签，之后每周再打一个候选发布（release candidate）标签，直到正式发布。

为在稳定期处理新特性和版本升级，可能会创建 `next` 分支。当前版本发布后，`next` 分支合并到 `master`，下一个发布周期的开发继续在 `master` 进行。

# 第四部分 附录

## 第二十五章 Makedev 语法文档（Makedev syntax documentation）

Makedev 语法在 Buildroot 的多个地方用于定义权限变更、要创建哪些设备文件及如何创建，以避免直接调用 mknod。

该语法源自 makedev 工具，更完整文档见 `package/makedevs/README` 文件。

格式为以空格分隔的字段，每行一个文件；字段如下：

| name（名称） | type（类型） | mode（权限） | uid | gid | major | minor | start | inc | count |
| ------------ | ----------- | ------------ | --- | --- | ----- | ----- | ----- | --- | ----- |
|              |             |              |     |     |       |       |       |     |       |

主要字段说明：

- `name`：要创建/修改的文件路径
- `type`：文件类型，可选：
  - `f`：普通文件，必须已存在
  - `F`：普通文件，若不存在则忽略不创建
  - `d`：目录，若不存在则自动创建及其父目录
  - `r`：递归目录，必须已存在
  - `c`：字符设备文件，父目录必须已存在
  - `b`：块设备文件，父目录必须已存在
  - `p`：命名管道，父目录必须已存在
- `mode`：权限设置（仅允许数字），`d` 类型父目录已存在则权限不变，新建父目录则设置权限；`f`、`F`、`r` 类型可设为 `-1` 表示不变（只改 uid/gid）
- `uid`、`gid`：文件的 UID 和 GID，可为数字或名称
- `major`、`minor`：仅设备文件需设置，其他文件设为 `-`
- `start`、`inc`、`count`：用于批量创建文件，相当于循环，从 `start` 开始，每次递增 `inc`，直到 `count`

示例：更改文件所有权和权限：

```
/usr/bin/foo f 755 0 0 - - - - -
/usr/bin/bar f 755 root root - - - - -
/data/buz f 644 buz-user buz-group - - - - -
/data/baz f -1 baz-user baz-group - - - - -
```

递归更改目录所有权：

```
/usr/share/myapp r -1 foo bar - - - - -
```

创建设备文件 `/dev/hda` 及其 15 个分区：

```
/dev/hda b 640 root root 3 0 0 0 -
/dev/hda b 640 root root 3 1 1 1 15
```

如启用 `BR2_ROOTFS_DEVICE_TABLE_SUPPORTS_EXTENDED_ATTRIBUTES`，可支持扩展属性。只需在文件描述行后加以 `|xattr` 开头的行，目前仅支持 capability：

| \|xattr | capability |
| ------- | ---------- |
|         |            |

- `|xattr`：扩展属性标志
- `capability`：要添加的 capability

为二进制 foo 添加 cap_sys_admin 能力：

```
/usr/bin/foo f 755 root root - - - - -
|xattr cap_sys_admin+eip
```

为同一文件添加多个 capability：

```
/usr/bin/foo f 755 root root - - - - -
|xattr cap_sys_admin+eip
|xattr cap_net_admin+eip
```
