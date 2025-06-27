### 18.25.3. 如何测试你的软件包

在你添加新软件包后，重要的是要在各种条件下对其进行测试：它能否为所有架构编译？能否与不同的 C 库一起编译？是否需要线程、NPTL？等等。

Buildroot 运行着 [autobuilders](http://autobuild.buildroot.org/)，这些自动构建器会持续测试随机配置。但它们只会构建 git 树的 `master` 分支，而你新添加的软件包还未合并到主线。

Buildroot 在 `utils/test-pkg` 提供了一个脚本，使用与 autobuilders 相同的基础配置，这样你可以在相同条件下测试你的软件包。

首先，创建一个配置片段（config snippet），包含启用你的软件包所需的所有选项，但不包含任何架构或工具链选项。例如，下面创建一个只启用 `libcurl`（libcurl），不带任何 TLS 后端的配置片段：

```
$ cat libcurl.config
BR2_PACKAGE_LIBCURL=y
```

如果你的软件包需要更多配置选项，可以将它们添加到配置片段。例如，下面是如何测试带有 `openssl`（openssl）作为 TLS 后端并启用 `curl` 程序的 `libcurl`：

```
$ cat libcurl.config
BR2_PACKAGE_LIBCURL=y
BR2_PACKAGE_LIBCURL_CURL=y
BR2_PACKAGE_OPENSSL=y
```

然后运行 `test-pkg` 脚本，指定要使用的配置片段和要测试的软件包：

```
$ ./utils/test-pkg -c libcurl.config -p libcurl
```

默认情况下，`test-pkg` 会将你的软件包与 autobuilders 使用的工具链子集进行构建测试，这个子集由 Buildroot 开发者选定，最具代表性。如果你想测试所有工具链，可以加上 `-a` 选项。无论哪种情况，内部工具链都被排除，因为它们构建时间太长。

输出会列出所有被测试的工具链及对应结果（以下为示例，结果为假）：

```
$ ./utils/test-pkg -c libcurl.config -p libcurl
                armv5-ctng-linux-gnueabi [ 1/11]: OK
              armv7-ctng-linux-gnueabihf [ 2/11]: OK
                        br-aarch64-glibc [ 3/11]: SKIPPED
                           br-arcle-hs38 [ 4/11]: SKIPPED
                            br-arm-basic [ 5/11]: FAILED
                  br-arm-cortex-a9-glibc [ 6/11]: OK
                   br-arm-cortex-a9-musl [ 7/11]: FAILED
                   br-arm-cortex-m4-full [ 8/11]: OK
                             br-arm-full [ 9/11]: OK
                    br-arm-full-nothread [10/11]: FAILED
                      br-arm-full-static [11/11]: OK
11 builds, 2 skipped, 2 build failed, 1 legal-info failed
```

结果含义如下：

- `OK`：构建成功。
- `SKIPPED`：配置片段中列出的一个或多个选项未出现在最终配置中。这通常是因为选项有依赖项未被工具链满足，例如某个包 `depends on BR2_USE_MMU`，而工具链为 noMMU。缺失的选项会在输出构建目录（默认 `~/br-test-pkg/TOOLCHAIN_NAME/`）下的 `missing.config` 文件中报告。
- `FAILED`：构建失败。请检查输出构建目录下的 `logfile` 文件，查看出错原因：
  - 实际构建失败，
  - legal-info 检查失败，
  - 前置步骤（下载配置文件、应用配置、为包运行 `dirclean`）失败。

出现失败时，你可以在修复包后用相同选项重新运行脚本；脚本会尝试为所有工具链重新构建指定的 `-p` 包，无需重新构建该包的所有依赖。

`test-pkg` 脚本支持一些选项，可通过运行以下命令获取帮助：

```
$ ./utils/test-pkg -h
```
