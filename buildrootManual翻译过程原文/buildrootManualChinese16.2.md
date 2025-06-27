### 18.25.4. 如何添加 GitHub 上的软件包

GitHub（GitHub）上的软件包通常没有带有发布 tarball 的下载区。但可以直接从 GitHub 仓库下载 tarball。由于 GitHub 过去曾更改过下载机制，建议如下所示使用 *github* 辅助函数：

```
# 使用标签（tag）或完整的提交 ID
FOO_VERSION = 1.0
FOO_SITE = $(call github,<user>,<package>,v$(FOO_VERSION))
```

**注意事项**

- FOO_VERSION 可以是标签（tag）或提交 ID。
- github 生成的 tarball 名称与 Buildroot 默认一致（如：`foo-f6fb6654af62045239caed5950bc6c7971965e60.tar.gz`），因此无需在 `.mk` 文件中指定。
- 如果用提交 ID 作为版本，需使用完整的 40 位十六进制字符。
- 当标签带有前缀（如 `v1.0`），则 `VERSION` 变量只写 `1.0`，`v` 前缀直接加在 `SITE` 变量里，如上例。这保证 `VERSION` 变量可用于 [release-monitoring.org](http://www.release-monitoring.org/) 匹配。

如果你要添加的软件包在 GitHub 上有 release 区，维护者可能上传了 release tarball，或者 release 只是指向由 git tag 自动生成的 tarball。如果有维护者上传的 tarball，建议优先使用，因为它可能有细微差别（如包含 configure 脚本，无需 AUTORECONF）。

你可以在 release 页面判断：

- 如果像上图那样，则是维护者上传的，建议用该链接（如示例中的 *mongrel2-v1.9.2.tar.bz2*）指定 `FOO_SITE`，不要用 *github* 辅助函数。
- 如果只有“Source code”链接，则是自动生成的 tarball，应使用 *github* 辅助函数。

### 18.25.5. 如何添加 Gitlab 上的软件包

与 [18.25.4 节 “如何添加 GitHub 上的软件包”](https://buildroot.org/downloads/manual/manual.html#github-download-url) 中的 `github` 宏类似，Buildroot 也提供了 `gitlab` 宏用于下载 Gitlab（Gitlab）仓库的自动生成 tarball，可用于特定标签或提交：

```
# 使用标签或完整提交 ID
FOO_VERSION = 1.0
FOO_SITE = $(call gitlab,<user>,<package>,v$(FOO_VERSION))
```

默认使用 `.tar.gz`，但 Gitlab 也提供 `.tar.bz2`，可通过添加 `<pkg>_SOURCE` 变量指定：

```
# 使用标签或完整提交 ID
FOO_VERSION = 1.0
FOO_SITE = $(call gitlab,<user>,<package>,v$(FOO_VERSION))
FOO_SOURCE = foo-$(FOO_VERSION).tar.bz2
```

如果上游开发者在 `https://gitlab.com/<project>/releases/` 上传了特定 tarball，不要用该宏，而应直接用 tarball 链接。

### 18.25.6. 为包访问私有仓库

如果你想在 br2-external 树中创建包，且源码在私有仓库（如 gitlab、github、bitbucket 等），需要保证开发者和 CI 都能构建。这带来挑战，因为需要认证。

有两种最实用的方法：

#### 使用 SSH 和 `insteadOf`

将你的私有包配置为用 SSH 访问。

```
FOO_SITE = git@githosting.com:/<group>/<package>.git
```

开发者通常已配置好 ssh key，可以直接访问。唯一限制是如果在 docker 内构建，需确保 ssh key 可被容器访问。可以通过 `-v ~/.ssh:<homedir>/.ssh` 挂载 SSH 目录，或用 ssh-agent 加载私钥并传递 `--mount type=bind,source=$SSH_AUTH_SOCK,target=/ssh-agent --env SSH_AUTH_SOCK=/ssh-agent`。

CI 构建机通常没有能访问其他仓库的 SSH key。此时需生成访问令牌（access token），然后配置 git 用 HTTPS 替换 SSH。CI 的准备步骤如下：

```
git config --global url."https://<token>:x-oauth-basic@githosting.com/<group>/".insteadOf "git@githosting.com:/<group>/"
```

不同 git 托管商和不同类型的 token，HTTPS 认证方式可能不同。请查阅你的托管商文档，了解如何用 token 通过 HTTPS 访问 git。

#### 使用 HTTPS 和 `.netrc`

如果开发者没有 SSH key，更简单的方法是用 HTTPS 认证。每位开发者都需生成有权访问相关仓库的 token。有些 git 托管商有命令行工具生成 token，否则需在网页端生成。token 有有效期，需定期刷新。

为确保 Buildroot 构建时使用 token，将其写入 `~/.netrc`：

```
machine githosting.com
    login <username>
    password <token>
```

不同 git 托管商 `<username>` 和 `<password>` 的用法可能不同。

在 CI 中，作为准备步骤生成 `.netrc` 文件。

将你的私有包配置为用 HTTPS 访问。

```
FOO_SITE = https://githosting.com/<group>/<package>.git
```

wget（https）和 git 都会用 `.netrc` 获取登录信息。这种方式安全性略低，因为 `.netrc` 不能加密。但优点是用户和 CI 用同样的认证方式。

## 18.26. 总结

如你所见，向 Buildroot 添加软件包只需编写一个 Makefile，参考现有示例并根据包的编译流程进行修改即可。

如果你打包的软件对他人有用，别忘了向 Buildroot 邮件列表提交补丁（见 [22.5 节 “提交补丁”](https://buildroot.org/downloads/manual/manual.html#submitting-patches)）！
