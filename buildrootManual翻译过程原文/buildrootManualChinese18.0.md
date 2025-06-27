## 第二十二章 参与 Buildroot 贡献

你可以通过多种方式为 Buildroot（Buildroot）做出贡献：分析和修复 bug（漏洞）、分析和修复 autobuilders（自动构建器）检测到的软件包构建失败、测试和评审其他开发者提交的补丁、处理我们的 TODO（待办事项）列表中的条目，以及向 Buildroot 或其手册提交你自己的改进。以下各节将对这些内容做进一步说明。

如果你有兴趣为 Buildroot 做贡献，首先应该订阅 Buildroot 邮件列表（mailing list）。该列表是与其他 Buildroot 开发者互动和提交贡献的主要方式。如果你还没有订阅，请参考[第五章，社区资源（Community resources）](https://buildroot.org/downloads/manual/manual.html#community-resources)获取订阅链接。

如果你打算修改代码，强烈建议你使用 Buildroot 的 git（Git）仓库，而不是直接从源码压缩包（tarball）开始。Git 是最便捷的开发方式，并且可以直接将你的补丁发送到邮件列表。关于如何获取 Buildroot git 树，请参考[第三章，获取 Buildroot（Getting Buildroot）](https://buildroot.org/downloads/manual/manual.html#getting-buildroot)。

## 22.1 复现、分析和修复 bug（漏洞）

参与贡献的第一种方式是查看 [Buildroot bug 跟踪器（bug tracker）](https://gitlab.com/buildroot.org/buildroot/-/issues) 上的未解决 bug 报告。我们努力保持 bug 数量尽可能少，因此欢迎大家协助复现、分析和修复已报告的 bug。即使你还没有完全弄清楚问题，也请不要犹豫，欢迎在 bug 报告下添加你的发现。

## 22.2 分析和修复自动构建失败

Buildroot 自动构建器（autobuilders）是一组持续运行 Buildroot 构建任务的构建机器，基于随机配置进行构建。它们会针对 Buildroot 支持的所有架构、使用不同的工具链（toolchain），并随机选择软件包。由于 Buildroot 的提交活动非常频繁，这些自动构建器有助于在提交后第一时间发现问题。

所有构建结果可在 [http://autobuild.buildroot.org](http://autobuild.buildroot.org/) 查看，统计信息见 http://autobuild.buildroot.org/stats.php。每天，所有失败软件包的概览会发送到邮件列表。

发现问题固然重要，但更重要的是要修复这些问题。你的贡献在这里非常受欢迎！主要可以做两件事：

- 分析问题。每日汇总邮件不会包含实际失败的详细信息：要了解具体情况，你需要打开构建日志（build log），查看最后的输出。有人为邮件中的所有软件包做这项工作，对其他开发者非常有帮助，因为他们可以基于这些输出做初步分析。
- 修复问题。修复自动构建失败时，你应遵循以下步骤：
  1. 检查你是否可以通过相同配置复现问题。你可以手动操作，也可以使用 [br-reproduce-build](http://git.buildroot.org/buildroot-test/tree/utils/br-reproduce-build) 脚本，该脚本会自动克隆 Buildroot git 仓库、检出正确的版本、下载并设置正确的配置，然后启动构建。
  2. 分析问题并创建修复方案。
  3. 从干净的 Buildroot 树开始，仅应用你的修复，验证问题确实被修复。
  4. 将修复补丁发送到 Buildroot 邮件列表（见[22.5节，提交补丁（Submitting patches）](https://buildroot.org/downloads/manual/manual.html#submitting-patches)）。如果你对软件包源码做了补丁，也应将补丁提交到上游（upstream），以便后续版本修复该问题，Buildroot 中的补丁可以移除。在修复自动构建失败的补丁提交信息中，请添加构建结果目录的引用，例如：

```
Fixes: http://autobuild.buildroot.org/results/51000a9d4656afe9e0ea6f07b9f8ed374c2e4069
```

## 22.3 评审和测试补丁

每天有大量补丁发送到邮件列表，维护者很难判断哪些补丁可以直接合入，哪些还需要完善。贡献者可以通过评审和测试这些补丁，极大地帮助维护者。

在评审过程中，请大胆回复补丁提交，提出意见、建议或任何有助于大家理解和完善补丁的内容。回复补丁时请使用互联网风格的纯文本邮件。

为了表明对补丁的认可，有三种正式标签用于记录这种认可。要为补丁添加你的标签，请在原作者的 Signed-off-by 行下方回复补丁并添加标签。Patchwork（见[22.3.1节，通过 Patchwork 应用补丁（Applying Patches from Patchwork）](https://buildroot.org/downloads/manual/manual.html#apply-patches-patchwork)）会自动识别这些标签，补丁被接受后，这些标签也会成为提交日志的一部分。

- Tested-by（已测试）

  表示该补丁已被成功测试。建议你说明测试的类型（如在 X 和 Y 架构上编译测试，在 A 目标板上运行测试等）。这些附加信息有助于其他测试者和维护者。

- Reviewed-by（已评审）

  表示你已对补丁进行了代码评审，并尽力发现问题，但你对所涉及领域不够熟悉，无法提供 Acked-by 标签。这意味着补丁中可能仍有问题，需要更有经验的人来发现。如果后续发现问题，你的 Reviewed-by 标签依然有效，你不会因此被指责。

- Acked-by（已确认）

  表示你已对补丁进行了代码评审，并且对所涉及领域足够熟悉，认为补丁可以直接合入（无需额外修改）。如果后续发现补丁有问题，你的 Acked-by 可能会被认为不合适。Acked-by 和 Reviewed-by 的主要区别在于，Acked-by 你愿意为补丁负责，而 Reviewed-by 则不承担责任。

如果你评审了补丁并有意见，请直接回复补丁说明你的意见，无需添加 Reviewed-by 或 Acked-by 标签。只有当你认为补丁已经很好时，才应添加这些标签。

需要注意的是，Reviewed-by 和 Acked-by 并不意味着已经进行了测试。要表示你既评审又测试了补丁，请分别添加两个标签（Reviewed/Acked-by 和 Tested-by）。

另外，*任何开发者*都可以提供 Tested/Reviewed/Acked-by 标签，无一例外，我们鼓励大家这样做。Buildroot 没有明确的“核心”开发者群体，只是有些开发者比其他人更活跃。维护者会根据提交者的历史记录来评估标签的可信度。常规贡献者的标签自然比新人的标签更受信任，但*任何*标签都是有价值的。

Buildroot 的 Patchwork 网站可用于拉取补丁进行测试。更多关于如何使用 Patchwork 应用补丁的信息，请参见[22.3.1节，通过 Patchwork 应用补丁（Applying Patches from Patchwork）](https://buildroot.org/downloads/manual/manual.html#apply-patches-patchwork)。

### 22.3.1 通过 Patchwork 应用补丁

Buildroot Patchwork 网站的主要用途是让开发者将补丁拉取到本地 git 仓库进行测试。

在 Patchwork 管理界面浏览补丁时，页面顶部会有一个 `mbox` 链接。复制该链接地址并运行以下命令：

```
$ git checkout -b <test-branch-name>
$ wget -O - <mbox-url> | git am
```

另一种应用补丁的方法是创建补丁集（bundle）。你可以在 Patchwork 界面将多个补丁分组为一个补丁集。补丁集创建并公开后，可以复制该补丁集的 `mbox` 链接，使用上述命令应用。

## 22.4 处理 TODO（待办事项）列表中的条目

如果你想为 Buildroot 做贡献但不知道从哪里开始，且对上述内容不感兴趣，也可以处理 [Buildroot TODO 列表](http://elinux.org/Buildroot#Todo_list) 中的条目。欢迎先在邮件列表或 IRC 上讨论某个条目。请在 wiki 上编辑，标明你已开始处理某个条目，以避免重复劳动。

## 22.5 提交补丁（patch）

### 注意

*请不要将补丁作为附件提交到 bug（漏洞）报告中，而应发送到邮件列表。*

如果你对 Buildroot 做了修改，想要将其贡献给 Buildroot 项目，请按如下流程操作。

### 22.5.1 补丁格式要求

我们要求补丁采用特定格式。这有助于补丁的评审、便于将补丁应用到 git 仓库、方便追溯历史变更的原因和方式，并能使用 `git bisect` 定位问题来源。

首先，补丁必须有良好的提交信息（commit message）。提交信息应以一行简要的变更摘要开头，并以补丁涉及的领域为前缀。以下是一些好的提交标题示例：

- `package/linuxptp: bump version to 2.0`
- `configs/imx23evk: bump Linux version to 4.19`
- `package/pkg-generic: postpone evaluation of dependency conditions`
- `boot/uboot: needs host-{flex,bison}`
- `support/testing: add python-ubjson tests`

前缀后的描述应以小写字母开头（如上述示例中的 "bump"、"needs"、"postpone"、"add"）。

其次，提交信息正文应描述*为什么*需要此更改，如有必要，还应说明*如何*实现。撰写提交信息时，请考虑评审者的阅读体验，也要考虑几年后你自己回看这次更改时的理解。

第三，补丁本身应只做一项更改，并且要做完整。两个无关或关系不大的更改通常应分为两个补丁。这通常意味着一个补丁只影响一个软件包。如果多个更改相关，通常也可以拆分为多个小补丁并按顺序应用。小补丁便于评审，也便于后续理解更改原因。但每个补丁必须是完整的，不能只应用第一个补丁就导致构建失败，这样才能使用 `git bisect`。

当然，在开发过程中，你可能会在不同软件包间来回修改，提交历史也未必一开始就适合提交。因此大多数开发者会重写提交历史，整理出适合提交的补丁集。为此，你需要使用*交互式变基（interactive rebasing）*。可参考 [Pro Git 书籍](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) 了解详情。有时，直接用 `git reset --soft origin/master` 丢弃历史，然后用 `git add -i` 或 `git add -p` 选择性添加更改会更容易。

最后，补丁应签名（signed off）。在提交信息末尾添加 `Signed-off-by: 你的真实姓名 <your@email.address>`。如果配置正确，`git commit -s` 会自动添加。`Signed-off-by` 标签表示你以 Buildroot 许可协议（即 GPL-2.0+，软件包补丁除外，采用上游许可）发布补丁，并且你有权这样做。详情见[开发者认证协议（Developer Certificate of Origin）](http://developercertificate.org/)。

如果要注明补丁由谁赞助或由谁协助上游，可以在 git 身份（即提交作者和邮件 From 字段，以及 Signed-off-by 标签）中使用[邮件子地址（email subaddressing）](https://datatracker.ietf.org/doc/html/rfc5233)；在本地部分后加加号 `+` 和后缀。例如：

- 公司赞助的补丁，后缀用公司名：

  `Your-Name Your-Surname <your-name.your-surname+companyname@mail.com>`

- 个人赞助的补丁，后缀用其姓名：

  `Your-Name Your-Surname <your-name.your-surname+their-name.their-surname@mail.com>`

如果你的邮件服务器不支持子地址，也可以在作者名后加括号注明赞助人，如 "Your Name (Sponsor Name)"。

添加新软件包时，每个软件包应单独提交补丁。补丁应包含对 `package/Config.in` 的更新、软件包的 `Config.in` 文件、`.mk` 文件、`.hash` 文件、任何初始化脚本以及所有软件包补丁。如果软件包有很多子选项，建议分为多个后续补丁。摘要行应为 `<packagename>: new package`。对于简单软件包，提交信息正文可以为空；复杂软件包可写上包描述（如 Config.in 的帮助文本）。如有特殊构建要求，也应在正文中明确说明。

升级软件包版本时，也应为每个软件包单独提交补丁。不要忘记更新 `.hash` 文件，或在不存在时添加。还要检查 `_LICENSE` 和 `_LICENSE_FILES` 是否仍然有效。摘要行应为 `<packagename>: bump to version <new version>`。如果新版本仅包含安全更新，摘要应为 `<packagename>: security bump to version <new version>`，正文应列出修复的 CVE 编号。如果新版本可以移除某些补丁，应明确说明原因，最好附上上游提交 ID。其他必要更改也应明确说明，如不再存在或不再需要的配置选项。

如果你希望在你添加或修改的软件包出现构建失败或后续变更时收到通知，请将自己添加到 DEVELOPERS 文件中。应在创建或修改软件包的同一个补丁中完成。更多信息见 [DEVELOPERS 文件](https://buildroot.org/downloads/manual/manual.html#DEVELOPERS)。

Buildroot 提供了一个名为 `check-package` 的工具，用于检查你创建或修改的文件是否存在常见的代码风格问题，详见[18.25.2节，如何检查代码风格（How to check the coding style）](https://buildroot.org/downloads/manual/manual.html#check-package)。
