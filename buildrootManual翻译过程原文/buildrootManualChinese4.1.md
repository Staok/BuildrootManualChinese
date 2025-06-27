## 8.11 软件包对文件系统大小的贡献分析

随着目标系统体积增大，了解各 Buildroot 软件包对根文件系统总体积的贡献变得很有价值。为此，Buildroot 会收集每个包安装的文件数据，并据此生成图表和 CSV 文件，详细展示各包的体积贡献。

构建后生成这些数据：

```
make graph-size
```

将生成：

- `output/graphs/graph-size.pdf`：各包对根文件系统总体积贡献的饼图
- `output/graphs/package-size-stats.csv`：各包对根文件系统总体积贡献的 CSV 文件
- `output/graphs/file-size-stats.csv`：每个已安装文件对所属包及总体积贡献的 CSV 文件

`graph-size` 目标需安装 Python Matplotlib 库（大多数发行版为 `python-matplotlib`），如 Python 版本低于 2.7 还需 `argparse` 模块。

如同时长分析图，可用 `BR2_GRAPH_OUT` 环境变量调整输出格式。详见[第 8.9 节 包依赖关系图](https://buildroot.org/downloads/manual/manual.html#graph-depends)。

还可用环境变量 `BR2_GRAPH_SIZE_OPTS` 进一步控制生成的图表。可用选项：

- `--size-limit X` 或 `-l X`：将单个包贡献低于 X 百分比的包合并为 *Others*，默认 X=0.01（即低于 1% 的包合并）。取值范围 `[0.0..1.0]`。
- `--iec`、`--binary`、`--si`、`--decimal`：选择 IEC（二进制，1024 进制）或 SI（十进制，1000 进制，默认）前缀。
- `--biggest-first`：按包体积降序排列，而非升序。

**注意：** 文件系统体积数据仅在完全 clean 重构后才有意义。请先运行 `make clean all` 再用 `make graph-size`。

如需比较两次 Buildroot 编译的根文件系统体积（如调整配置或切换 Buildroot 版本后），可用 `size-stats-compare` 脚本。该脚本以两份 `file-size-stats.csv`（由 `make graph-size` 生成）为输入。详细用法见脚本帮助：

```
utils/size-stats-compare -h
```

## 8.12 顶层并行构建（Top-level parallel build）

**注意：** 本节介绍的为实验性功能，已知在部分常见场景下会出错，请谨慎使用。

Buildroot 一直支持包内并行构建：每个包用 `make -jN`（或等效命令）构建，默认并行度为 CPU 数 + 1，可通过 `BR2_JLEVEL` 配置调整。

直到 2020.02 版本，Buildroot 仍以串行方式构建各包：每次只构建一个包，包间无并行。自 2020.02 起，Buildroot 实验性支持***顶层并行构建***，可并行构建无依赖关系的包，显著缩短构建时间。但该功能仍为实验性，部分场景下已知不可用。

使用顶层并行构建需：

1. 在 Buildroot 配置中启用 `BR2_PER_PACKAGE_DIRECTORIES` 选项
2. 构建时用 `make -jN`

内部实现上，`BR2_PER_PACKAGE_DIRECTORIES` 会启用***每包独立目录（per-package directories）***机制，具体效果如下：

- 不再有全局 *target* 和 *host* 目录，每个包有独立的 *target* 和 *host* 目录，分别位于 `$(O)/per-package/<pkg>/target/` 和 `$(O)/per-package/<pkg>/host/`。这些目录在 `<pkg>` 构建前从依赖包目录同步。编译器和工具只能访问 `<pkg>` 显式依赖包安装的文件。
- 构建结束后，全局 *target* 和 *host* 目录才会被填充，分别位于 `$(O)/target` 和 `$(O)/host`。构建期间这两个目录为空，只有在构建结束时才填充。
