## 第二十六章 Makeusers 语法文档（Makeusers syntax documentation）

创建用户的语法借鉴了上文的 makedev 语法，但为 Buildroot 专用。

添加用户的语法为以空格分隔的字段，每行一个用户；字段如下：

| username（用户名） | uid | group（主组） | gid | password（密码） | home（家目录） | shell（Shell） | groups（附加组） | comment（备注） |
| ------------------ | --- | ------------- | --- | --------------- | -------------- | -------------- | ---------------- | --------------- |
|                    |     |               |     |                 |                |                |                  |                 |

说明：

- `username`：用户登录名，不能为 `root`，且必须唯一。若为 `-`，则只创建组。
- `uid`：用户 UID，必须唯一且不能为 `0`。若为 `-1` 或 `-2`，Buildroot 会自动分配唯一 UID，`-1` 表示系统 UID（100~999），`-2` 表示普通用户 UID（1000~1999）。
- `group`：主组名，不能为 `root`。若组不存在会自动创建。
- `gid`：主组 GID，必须唯一且不能为 `0`。若为 `-1` 或 `-2` 且组不存在，则 Buildroot 会自动分配唯一 GID，`-1` 表示系统 GID（100~999），`-2` 表示普通用户 GID（1000~1999）。
- `password`：crypt(3) 加密密码。以 `!` 开头则禁用登录；以 `=` 开头则为明文密码，会用 MD5 加密；以 `!=` 开头则明文加密且禁用登录。为 `*` 则禁止登录。为 `-` 则不设置密码。
- `home`：用户家目录。为 `-` 则不创建家目录，用户家目录为 `/`。不能显式设置为 `/`。
- `shell`：用户 shell。为 `-` 则 shell 设为 `/bin/false`。
- `groups`：用户所属附加组，逗号分隔。为 `-` 则无附加组。缺失的组会自动创建。
- `comment`：备注（GECOS 字段），几乎可任意填写。

字段限制：

- 除 `comment` 外，所有字段必填。
- 除 `comment` 外，字段不能包含空格。
- 所有字段不能包含冒号（:）。

如 `home` 不为 `-`，则家目录及其下所有文件归该用户及主组所有。

示例：

```
foo -1 bar -1 !=blabla /home/foo /bin/sh alpha,bravo Foo user
```

将创建如下用户：

- 用户名：`foo`
- UID 由 Buildroot 分配
- 主组：`bar`
- 主组 GID 由 Buildroot 分配
- 明文密码为 `blabla`，会加密且禁用登录
- 家目录为 `/home/foo`
- shell 为 `/bin/sh`
- 附加组为 `alpha` 和 `bravo`
- 备注为 `Foo user`

```
test 8000 wheel -1 = - /bin/sh - Test user
```

将创建如下用户：

- 用户名：`test`
- UID 为 8000
- 主组为 `wheel`
- 主组 GID 由 Buildroot 分配，取 rootfs skeleton 中定义值
- 密码为空（无密码）
- 家目录为 `/`，但不归 `test` 所有
- shell 为 `/bin/sh`
- 无附加组
- 备注为 `Test user`

## 26.1 自动 UID 和 GID 的注意事项

升级 buildroot 或增删配置中的软件包时，自动分配的 UID 和 GID 可能会变化。如果有持久化文件由该用户或组创建，升级后这些文件的所有者会变。

因此建议将自动分配的 ID 固定下来。可通过添加包含已生成 ID 的用户表实现。仅需为实际创建持久化文件（如数据库）的 UID 固定。

## 第二十七章 旧版 Buildroot 迁移指南（Migrating from older Buildroot versions）

部分版本引入了不兼容变更。本节说明这些不兼容及迁移方法。

## 27.1 通用迁移流程

从旧版 Buildroot 迁移，建议如下：

1. 针对所有配置，在旧 Buildroot 环境下构建一次。运行 `make graph-size`，保存 `graphs/file-size-stats.csv` 到其他位置。运行 `make clean` 清理其余内容。
2. 阅读下文具体迁移说明，按需调整外部包和自定义构建脚本。
3. 升级 Buildroot。
4. 从现有 `.config` 开始，运行 `make menuconfig`。
5. 若 Legacy 菜单有启用项，查看帮助说明，取消选择并保存配置。
6. 详细情况可查阅相关软件包的 git 提交信息。进入 `packages` 目录，运行 `git log <old version>..--<your packages>`。
7. 在新 Buildroot 环境下构建。
8. 修复外部包的构建问题（通常因依赖升级）。
9. 运行 `make graph-size`。
10. 比较新旧 `file-size-stats.csv`，确保无必需文件丢失，也无新出现的大型无用文件。
11. 对自定义 overlay 中覆盖 Buildroot 生成文件的配置（及其他）文件，检查 Buildroot 生成文件是否有变更需同步到自定义文件。

## 27.2 迁移到 2016.11

2016.11 之前只能同时使用一个 br2-external 树。2016.11 起可同时用多个（详见[9.2节，Buildroot 外部自定义（Keeping customizations outside of Buildroot）](https://buildroot.org/downloads/manual/manual.html#outside-br-custom)）。

因此旧 br2-external 树不能直接用。需做如下小改动：为 br2-external 树添加名称。

操作如下：

- 在 br2-external 树根目录新建 `external.desc` 文件，内容为一行，定义树名：

  ```
  $ echo 'name: NAME_OF_YOUR_TREE' >external.desc
  ```

  **注意**：名称需唯一，仅可用 `[A-Za-z0-9_]` 字符。

- 替换 br2-external 树中所有 `BR2_EXTERNAL` 为新变量：

  ```
  $ find . -type f | xargs sed -i 's/BR2_EXTERNAL/BR2_EXTERNAL_NAME_OF_YOUR_TREE_PATH/g'
  ```

现在你的 br2-external 树可用于 2016.11 及以后版本。

**注意**：此更改会导致 br2-external 树无法用于 2016.11 之前版本。

## 27.3 迁移到 2017.08

2017.08 之前，host 包安装在 `$(HOST_DIR)/usr`（如 autotools 的 `--prefix=$(HOST_DIR)/usr`）。2017.08 起直接安装到 `$(HOST_DIR)`。

如包安装的可执行文件链接了 `$(HOST_DIR)/lib` 下的库，必须有指向该目录的 RPATH。

RPATH 指向 `$(HOST_DIR)/usr/lib` 不再被接受。

## 27.4 迁移到 2023.11

2023.11 之前，subversion 下载后会无条件获取外部引用（`svn:externals` 属性对象）。2023.11 起默认不再获取，如需请设 `LIBFOO_SVN_EXTERNALS=YES`。这意味着：

- 生成的归档内容可能变化，需相应更新 hash；
- 归档版本后缀更新为 `-br3`，hash 文件需相应更新。

2023.11 之前可用架构特定补丁（如 `0001-some-changes.patch.arm` 仅对该架构生效）。2023.11 起不再支持此机制，此类补丁不会被应用。

如仍需架构特定补丁，可提供[预补丁钩子（pre-patch hook）](https://buildroot.org/downloads/manual/manual.html#hooks)，如：

```
define LIBFOO_ARCH_PATCHES
    $(foreach p,$(wildcard $(LIBFOO_PKGDIR)/*.patch.$(ARCH)), \
        cp -f $(p) $(patsubst %.$(ARCH),%,$(p))
    )
endef
LIBFOO_PRE_PATCH_HOOKS += LIBFOO_ARCH_PATCHES
```

注意 Buildroot 官方包没有架构特定补丁，此类补丁一般不会被接受。

## 27.5 迁移到 2024.05

下载后端有多项扩展：

- 所有本地生成的 tarball 更加可复现。2024.05 前，归档中文件访问权限可能因下载目录 ACL（如 `default` ACL）不同而不一致，影响 git、subversion、cargo、go 包归档。
- git 下载后端现在会正确展开 `export-subst` [git 属性](https://git-scm.com/docs/gitattributes)。
- 归档需用新版 `tar`（1.35），该版本更改了部分字段（`devmajor`、`devminor`）的存储方式，影响归档元数据（文件内容不变）。

因此归档后缀有如下变更：

- git：`-git4`
- subversion：`-svn5`
- cargo（rust）包：`-cargo2`
- go 包：`-go2`

如归档有多个前缀（如 cargo 包用 git 下载），需都加，顺序为下载机制前缀在前，如：`libfoo-1.2.3-git4-cargo2.tar.gz`。

因此自定义包或内核/引导程序的 hash 文件需更新。可用如下 sed 脚本自动重命名（假设 hash 文件用 git 管理）：

```
# git、svn 包，原后缀为 -br2、-br3
sed -r -i -e 's/-br2/-git4/; s/-br3/-svn5/' $(
    git grep -l -E -- '-br2|-br3' -- '*.hash'
)

# go 包，原无后缀
sed -r -i -e 's/(\.tar\.gz)$/-go2\1/' $(
    git grep -l -E '\$\(eval \$\((host-)?golang-package\)\)' -- '*.mk' \
    |sed -r -e 's/\.mk$/.hash/' \
    |sort -u
)

# cargo 包，原无后缀
sed -r -i -e 's/(\.tar\.gz)$/-cargo2\1/' $(
    git grep -l -E '\$\(eval \$\((host-)?cargo-package\)\)' -- '*.mk' \
    |sed -r -e 's/\.mk$/.hash/' \
    |sort -u
)
```

注意 hash *必然*已变，需手动更新。

## 27.6 迁移到 2025.02

Mender 现在要求在 `/var/lib/mender` 放置特殊 bootstrap artifact，替代原 `artifact_info` 文件。该 artifact 用 host-mender-artifact 生成。见 `board/mender/x86_64/post-image-efi.sh` 示例。更多信息见[发布说明](https://docs.mender.io/release-information/release-notes-changelog/mender-client#mender-3-5-0-1)的 features 部分。

## 27.7 迁移到 2025.05

2025.05 起，SYS-V 类系统（busybox、sysvinit、openrc）中 `/etc/resolv.conf` 的符号链接指向 `/run/resolv.conf`，不再指向旧的 `/tmp`。自定义 fstab 的用户需确保 `/run` 在 resolv.conf 创建前可写（通常通过 fstab 条目或启动脚本）。

systemd 系统不受影响：systemd 总会确保 `/run` 可写，且 systemd-resolved 早已将 `/etc/resolv.conf` 指向 `/run`。

Rust 升级到 1.84.0 以上后，Cargo.lock 文件变为强制，且 `.cargo/config` 改为 `.cargo/config.toml`，由 Cargo 获取的包的 tarball 也变了。因此此类 tarball 后缀由 `-cargo2` 改为 `-cargo4`。
