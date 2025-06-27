## 8.8 关于软件包的详细信息

Buildroot 可生成 JSON 格式的摘要，描述当前配置下已启用软件包的集合，包括其依赖关系、许可证及其他元数据。可通过 `show-info` make 目标生成：

```
make show-info
```

Buildroot 还可通过 `pkg-stats` make 目标生成软件包详细信息（HTML 和 JSON 格式）。其中包括当前配置下软件包是否受已知 CVE（安全漏洞）影响，以及是否有上游新版本。

```
make pkg-stats
```

## 8.9 软件包依赖关系图

Buildroot 负责管理软件包间的依赖关系，确保按正确顺序构建。这些依赖有时很复杂，理解某个包为何被 Buildroot 引入并不容易。

为帮助理解依赖关系、理清嵌入式 Linux 系统各组件作用，Buildroot 可生成依赖关系图。

生成全系统依赖关系图：

```
make graph-depends
```

生成的图位于 `output/graphs/graph-depends.pdf`。

如系统较大，依赖图可能过于复杂。可仅为某个包生成依赖图：

```
make <pkg>-graph-depends
```

生成的图位于 `output/graph/<pkg>-graph-depends.pdf`。

依赖关系图由 *Graphviz* 项目的 `dot` 工具生成，需预先安装（大多数发行版包名为 `graphviz`）。

默认输出为 PDF 格式。可通过 `BR2_GRAPH_OUT` 环境变量切换为 PNG、PostScript、SVG 等（所有 `dot` 工具 `-T` 选项支持的格式均可用）。

```
BR2_GRAPH_OUT=svg make graph-depends
```

`graph-depends` 行为可通过 `BR2_GRAPH_DEPS_OPTS` 环境变量设置。可用选项：

- `--depth N` 或 `-d N`：限制依赖深度为 N 层，默认 0 表示无限制。
- `--stop-on PKG` 或 `-s PKG`：在包 `PKG` 处停止绘制。`PKG` 可为包名、通配符、*virtual*（虚包）、*host*（主机包）。该包仍在图中，但不再展开其依赖。
- `--exclude PKG` 或 `-x PKG`：同 `--stop-on`，但图中不显示该包。
- `--transitive`、`--no-transitive`：是否绘制传递依赖。默认不绘制。
- `--colors R,T,H`：设置根包（R）、目标包（T）、主机包（H）的颜色，默认 `lightblue,grey,gainsboro`。

```
BR2_GRAPH_DEPS_OPTS='-d 3 --no-transitive --colors=red,green,blue' make graph-depends
```

## 8.10 构建时长分析图

如系统构建耗时较长，可分析各包构建耗时以优化构建速度。Buildroot 会收集每个包各步骤的构建时长，并可生成图表。

构建后生成时长分析图：

```
make graph-build
```

生成文件位于 `output/graphs`：

- `build.hist-build.pdf`：按构建顺序排列的各包构建时长直方图
- `build.hist-duration.pdf`：按时长降序排列的各包构建时长直方图
- `build.hist-name.pdf`：按包名排列的各包构建时长直方图
- `build.pie-packages.pdf`：各包构建时长饼图
- `build.pie-steps.pdf`：各步骤总耗时饼图

`graph-build` 目标需安装 Python Matplotlib 和 Numpy 库（大多数发行版为 `python-matplotlib` 和 `python-numpy`），如 Python 版本低于 2.7 还需 `argparse` 模块。

默认输出为 PDF 格式，可用 `BR2_GRAPH_OUT` 环境变量切换为 PNG：

```
BR2_GRAPH_OUT=png make graph-build
```
