## 11.9 Buildroot 如何支持 Y2038 问题？

有多种情况需要考虑：

- 在 64 位架构上，没有问题，因为 `time_t` 一直是 64 位；
- 在 32 位架构上，情况取决于 C 库：
  - 使用 *uclibc-ng* 时，自 1.0.46 版本起支持 32 位架构上的 64 位 `time_t`，因此在 32 位平台上使用 *uclibc-ng* 时，只要 UCLIBC_USE_TIME64 设为 y（1.0.49 及以后默认如此），系统就支持 Y2038；
  - 使用 *musl* 时，32 位架构上一直使用 64 位 `time_t`，因此在 32 位平台上使用 *musl* 的系统支持 Y2038；
  - 使用 *glibc* 时，32 位架构上的 64 位 `time_t` 由 Buildroot 选项 `BR2_TIME_BITS_64` 启用。启用后，32 位平台上使用 *glibc* 的系统支持 Y2038。

注意，上述仅说明 C 库的能力。即使在 Y2038 兼容的环境下，单个用户空间库或应用如果未正确使用时间相关 API 和类型，仍可能表现异常。

## 第12章 已知问题

- 如果 `BR2_TARGET_LDFLAGS` 选项中包含 `$` 符号，则无法通过该选项传递额外的链接器参数。例如：`BR2_TARGET_LDFLAGS="-Wl,-rpath='$ORIGIN/../lib'"` 会导致问题；
- `libffi` 软件包不支持 SuperH 2 和 ARMv7-M 架构；
- `prboom` 软件包在使用 Sourcery CodeBench 2012.09 版 SuperH 4 编译器时会触发编译失败。

## 第13章 法律声明与许可

## 13.1 遵守开源许可证

Buildroot 生成的所有最终产品（工具链、根文件系统、内核、引导加载程序）都包含开源软件，采用各种许可证发布。

使用开源软件让你可以自由构建丰富的嵌入式系统，选择多种软件包，但也带来一些你必须了解和遵守的义务。有些许可证要求你在产品文档中公布许可证文本，另一些要求你向产品接收者分发软件源代码。

每种许可证的具体要求都在各自软件包中有说明，你（或你的法务部门）有责任遵守这些要求。为方便你，Buildroot 可以为你收集一些你可能需要的材料。配置好 Buildroot（用 `make menuconfig`、`make xconfig` 或 `make gconfig`）后，运行：

```
make legal-info
```

Buildroot 会在输出目录下的 `legal-info/` 子目录中收集法律相关材料，包括：

- `README` 文件，汇总所收集材料，并提示 Buildroot 未能收集的内容；
- `buildroot.config`：Buildroot 配置文件，通常由 `make menuconfig` 生成，用于复现构建；
- 所有软件包的源代码，分别保存在 `sources/` 和 `host-sources/` 子目录（分别对应目标和主机软件包）。设置了 `<PKG>_REDISTRIBUTE = NO` 的包不会保存源码。应用的补丁也会保存，并有 `series` 文件记录补丁应用顺序。补丁与被修改文件采用相同许可证。注意：Buildroot 会对 autotools 类软件包的 Libtool 脚本应用额外补丁，这些补丁位于 Buildroot 源码的 `support/libtool`，由于技术原因不会和包源码一起保存，需手动收集；
- 清单文件（目标和主机各一份），列出已配置软件包、版本、许可证及相关信息。部分信息 Buildroot 未定义时会标记为“unknown”；
- 所有软件包的许可证文本，分别保存在 `licenses/` 和 `host-licenses/` 子目录（分别对应目标和主机软件包）。如果 Buildroot 未定义许可证文件，则不会生成，并在 `README` 中有警告。

请注意，Buildroot 的 `legal-info` 功能旨在收集所有与软件包许可证合规相关的材料。Buildroot 并不试图生成你必须公开的全部材料。实际上，生成的材料往往比严格合规所需的更多。例如，BSD 类许可证的软件包源码也会被收集，但你并不需要分发这些源码。

此外，由于技术限制，Buildroot 不会生成某些你需要或可能需要的材料，比如部分外部工具链的源码和 Buildroot 自身源码。运行 `make legal-info` 时，Buildroot 会在 `README` 文件中提示未能保存的相关材料。

最后，`make legal-info` 的输出基于各软件包 recipe 中的声明。Buildroot 开发者会尽力保证这些声明的准确性，但不排除有不准确或不完整之处。你（或你的法务部门）*必须*在使用 `make legal-info` 输出作为合规交付材料前自行核查。详见 Buildroot 根目录 `COPYING` 文件第 11、12 条 *无担保* 条款。
