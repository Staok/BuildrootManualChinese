## 11.7 为什么 Buildroot 不生成二进制软件包（.deb、.ipkg 等）？

在 Buildroot 邮件列表中经常讨论的一个功能是“软件包管理（package management）”的主题。简而言之，就是希望能够追踪 Buildroot 各软件包安装了哪些文件，目标包括：

- 当某个软件包在 menuconfig 中被取消选择时，能够删除该软件包安装的文件；
- 能够生成二进制软件包（如 ipk 或其他格式），可在目标系统上直接安装，而无需重新生成新的根文件系统镜像。

通常，大多数人认为这很容易：只需追踪每个软件包安装了哪些文件，在取消选择时删除即可。但实际上远比这复杂：

- 不仅仅是 `target/` 目录，还包括 `host/<tuple>/sysroot` 和 `host/` 目录本身。所有软件包在这些目录中安装的文件都必须被追踪；
- 当某个软件包在配置中被取消选择时，仅删除它安装的文件是不够的。还必须删除所有反向依赖（即依赖它的软件包），并重建这些软件包。例如，A 软件包可选依赖 OpenSSL 库。两者都被选中并构建，A 软件包会用 OpenSSL 支持构建。后来 OpenSSL 被取消选择，但 A 软件包仍然保留（因为 OpenSSL 是可选依赖）。如果只删除 OpenSSL 文件，A 软件包安装的文件就会出错：它们依赖的库已不在目标系统。虽然技术上可行，但会极大增加 Buildroot 的复杂性，这与 Buildroot 追求的简洁性背道而驰；
- 除了上述问题，还有一种情况是可选依赖甚至 Buildroot 并不知道。例如，A 软件包 1.0 版本从未用过 OpenSSL，但 2.0 版本如果检测到 OpenSSL 就会自动使用。如果 Buildroot 的 .mk 文件没有及时更新，A 软件包就不会被列为 OpenSSL 的反向依赖，在 OpenSSL 被移除时也不会被删除和重建。理论上 .mk 文件应及时修正，但在此期间就会出现不可复现的行为；
- 还有人希望 menuconfig 的更改能直接应用到输出目录，而无需全部重建。但要可靠地实现这一点非常困难：如果更改了软件包的子选项（需要检测并重建该包及其所有反向依赖），如果更改了工具链选项等。目前 Buildroot 的做法清晰简单，因此行为非常可靠，易于支持用户。如果配置更改后只需下一次 make 就能生效，那么必须保证所有场景都能正确工作，不能有奇怪的边角问题。否则会收到类似“我启用了 A、B、C，然后 make，后来禁用 C 启用 D 又 make，再启用 C 启用 E 又 make，结果构建失败”这样的 bug 报告。更糟的是“我做了一些配置，构建了，然后又改了点，构建了，又改了点，构建了……现在失败了，但我不记得都改了什么顺序”。这种情况将无法支持。

因此，结论是：为删除被取消选择的软件包或生成二进制软件包而追踪已安装文件，是非常难以可靠实现的，会极大增加复杂性。

在这个问题上，Buildroot 开发者的立场声明如下：

- Buildroot 致力于让生成根文件系统（root filesystem）变得简单（这也是 Buildroot 名字的由来）。我们希望 Buildroot 擅长的就是构建根文件系统；
- Buildroot 并不打算成为一个发行版（distribution）或发行版生成器。大多数 Buildroot 开发者认为这不是我们应追求的目标。我们认为有其他工具更适合生成发行版，比如 [Open Embedded](http://openembedded.org/) 或 [openWRT](https://openwrt.org/)；
- 我们更愿意推动 Buildroot 朝着更容易生成完整根文件系统的方向发展。这也是 Buildroot 能脱颖而出的原因之一；
- 我们认为对于大多数嵌入式 Linux 系统，二进制软件包并非必需，甚至有害。使用二进制包意味着系统可以被部分升级，这会带来大量可能的软件包版本组合，在升级前都需要测试。而通过整体升级根文件系统镜像，部署到嵌入式设备的镜像就是经过测试和验证的。

## 11.8 如何加快构建速度？

由于 Buildroot 经常需要完整重建整个系统，过程可能很长，下面提供一些加快构建速度的建议：

- 使用预构建的外部工具链（external toolchain），而不是默认的 Buildroot 内部工具链。比如 ARM 平台用 Linaro 工具链，ARM、x86、x86-64、MIPS 等可用 Sourcery CodeBench 工具链，这样每次完整重建时可节省 15-20 分钟的工具链构建时间。临时用外部工具链不影响后续切换回内部工具链（后者可高度定制），等系统其它部分调通后再切换；
- 使用 `ccache` 编译器缓存（见[8.13.3节“在 Buildroot 中使用 ccache”](https://buildroot.org/downloads/manual/manual.html#ccache)）；
- 学会只重建关心的少数软件包（见[8.3节“理解如何重建软件包”](https://buildroot.org/downloads/manual/manual.html#rebuild-pkg)），但有时还是必须完整重建（见[8.2节“理解何时需要完整重建”](https://buildroot.org/downloads/manual/manual.html#full-rebuild)）；
- 确保运行 Buildroot 的 Linux 系统不是虚拟机。大多数虚拟机技术会显著影响 I/O 性能，而 I/O 对源码构建非常重要；
- 确保只用本地文件：不要在 NFS 上构建，这会极大拖慢速度。Buildroot 下载目录也应本地化；
- 升级硬件。SSD 和大内存对加速构建非常关键；
- 尝试顶层并行构建（top-level parallel build），见[8.12节“顶层并行构建”](https://buildroot.org/downloads/manual/manual.html#top-level-parallel-build)。
