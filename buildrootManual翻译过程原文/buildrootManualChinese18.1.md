### 22.5.2 准备补丁集（patch series）

在本地 git（Git）视图中提交更改后，在生成补丁集之前，*rebase*（变基）你的开发分支到上游主线。操作如下：

```
$ git fetch --all --tags
$ git rebase origin/master
```

现在，检查你提交的更改的代码风格：

```
$ utils/docker-run make check-package
```

现在，你可以生成并提交补丁集了。

生成补丁集，运行：

```
$ git format-patch -M -n -s -o outgoing origin/master
```

这将在 `outgoing` 子目录下生成补丁文件，并自动添加 `Signed-off-by` 行。

补丁文件生成后，你可以用自己喜欢的文本编辑器在提交前审查/编辑提交信息。

Buildroot 提供了一个方便的工具 `get-developers`，用于确定补丁应发送给哪些开发者，详见[第二十三章，DEVELOPERS 文件与 get-developers](https://buildroot.org/downloads/manual/manual.html#DEVELOPERS)。该工具会读取你的补丁并输出合适的 `git send-email` 命令：

```
$ ./utils/get-developers outgoing/*
```

使用 `get-developers` 的输出发送补丁：

```
$ git send-email --to buildroot@buildroot.org --cc bob --cc alice outgoing/*
```

另外，也可以直接用 `get-developers -e` 配合 `git send-email` 的 `--cc-cmd` 参数自动 CC 相关开发者：

```
$ git send-email --to buildroot@buildroot.org \
      --cc-cmd './utils/get-developers -e' origin/master
```

你还可以通过如下方式让 git 自动配置这些参数：

```
$ git config sendemail.to buildroot@buildroot.org
$ git config sendemail.ccCmd "$(pwd)/utils/get-developers -e"
```

之后只需：

```
$ git send-email origin/master
```

注意，`git` 需要配置你的邮件账户。配置方法请参见 `man git-send-email` 或 https://git-send-email.io/。

如果你不使用 `git send-email`，请确保***补丁邮件不会被自动换行***，否则补丁将无法被方便地应用。如果遇到此问题，请修正你的邮件客户端，或者更推荐学习使用 `git send-email`。

[https://sr.ht](https://sr.ht/) 也提供了一个轻量级界面用于[准备补丁集](https://man.sr.ht/git.sr.ht/#sending-patches-upstream)，并可帮你发送补丁。该方式有一些缺点，比如无法在 Patchwork 中编辑补丁状态，也无法修改补丁邮件的显示名，但如果你的邮件服务（如 O365）无法用 `git send-email`，这也是一个备选方案。注意，这不是官方推荐的补丁发送方式，仅作补充。

### 22.5.3 封面信（cover letter）

如果你想用单独邮件介绍整个补丁集，在 `git format-patch` 命令中加 `--cover-letter`（详见 `man git-format-patch`）。这会生成一个补丁集介绍邮件的模板。

在以下情况下，*封面信* 很有用：

- 补丁集包含大量提交；
- 变更对项目影响较大；
- RFC（Request for comments，征求意见稿）；
- 或者你觉得有助于介绍你的工作、选择、评审流程等。

### 22.5.4 维护分支的补丁

修复维护分支（maintenance branch）上的 bug 时，应先在 master 分支修复。此类补丁的提交日志可包含一条说明受影响分支的备注：

```
package/foo: fix stuff

Signed-off-by: Your Real Name <your@email.address>
---
Backport to: 2020.02.x, 2020.05.x
(2020.08.x not affected as the version was bumped)
```

这些更改随后会由维护者回合到受影响的分支。

但有些 bug 只影响特定版本（如使用了旧版软件包）。此时，补丁应基于维护分支，并在补丁主题前缀中注明维护分支名（如 "[PATCH 2020.02.x]"）。可用 `git format-patch` 的 `--subject-prefix` 参数实现：

```
$ git format-patch --subject-prefix "PATCH 2020.02.x" \
    -M -s -o outgoing origin/2020.02.x
```

然后用 `git send-email` 发送补丁，如上所述。

### 22.5.5 补丁修订日志（Patch revision changelog）

如有改进建议，每次提交的新版本补丁都应在提交信息中包含本次与上次的修改日志。若补丁集有封面信，可在封面信和各补丁提交信息中分别写修订日志。推荐用交互式变基（interactive rebasing）：`git rebase -i origin/master`。详见 git 手册。

在各补丁提交信息中，修订日志应写在 `Signed-off-by` 之后，`---` 之下。

修订日志会在邮件讨论和 [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/) 中显示，但 `git` 合并补丁时会自动忽略 `---` 之后的内容，这是预期行为：修订日志不应永久保留在项目 git 历史中。

推荐格式如下：

```
Patch title: short explanation, max 72 chars

A paragraph that explains the problem, and how it manifests itself. If
 the problem is complex, it is OK to add more paragraphs. All paragraphs
 should be wrapped at 72 characters.

A paragraph that explains the root cause of the problem. Again, more
 than one paragraph is OK.

Finally, one or more paragraphs that explain how the problem is solved.
Don't hesitate to explain complex solutions in detail.

Signed-off-by: John DOE <john.doe@example.net>

---
Changes v2 -> v3:
  - foo bar  (suggested by Jane)
  - bar buz

Changes v1 -> v2:
  - alpha bravo  (suggested by John)
  - charly delta
```

任何补丁修订都应包含版本号。版本号格式为字母 `v` 加上大于等于 2 的整数（如 "PATCH v2"、"PATCH v3" 等）。

用 `git format-patch` 的 `--subject-prefix` 选项可方便实现：

```
$ git format-patch --subject-prefix "PATCH v4" \
    -M -s -o outgoing origin/master
```

自 git 1.8.1 起，也可用 `-v <n>`（<n> 为版本号）：

```
$ git format-patch -v4 -M -s -o outgoing origin/master
```

提交新版本补丁时，请在 [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/) 标记旧版本为 superseded（已被替代）。你需在 patchwork 注册账号，且注册邮箱需与发送补丁到邮件列表时一致。

你还可以在发送补丁时加 `--in-reply-to=<message-id>` 参数。邮件的 id 可在 [patchwork](https://patchwork.ozlabs.org/project/buildroot/list/) 的 "Message Id" 标签下找到。这样 patchwork 会自动将旧版本补丁标记为 superseded。

## 22.6 报告问题/bug 或寻求帮助

在报告任何问题前，请先在[邮件列表归档](https://buildroot.org/downloads/manual/manual.html#community-resources)中查找是否有人已报告/修复了类似问题。

无论你选择在[bug 跟踪器](https://buildroot.org/downloads/manual/manual.html#community-resources)报告 bug，还是[发邮件到邮件列表](https://buildroot.org/downloads/manual/manual.html#community-resources)寻求帮助，为了方便他人复现和解决问题，请尽量提供以下信息：

- 主机（host）操作系统及版本
- Buildroot 版本
- 构建失败的目标（target）
- 构建失败的软件包（package）
- 失败的命令及其输出
- 你认为可能相关的其他信息

此外，建议附上 `.config` 文件（或如有能力，附上 `defconfig`，见[9.3节，保存 Buildroot 配置（Storing the Buildroot configuration）](https://buildroot.org/downloads/manual/manual.html#customize-store-buildroot-config)）。

如有些信息太大，可使用 pastebin 服务。注意，并非所有 pastebin 服务都能在下载原始内容时保留 Unix 风格换行符。以下 pastebin 服务已知可正常使用：
- https://gist.github.com/
- http://code.bulix.org/
