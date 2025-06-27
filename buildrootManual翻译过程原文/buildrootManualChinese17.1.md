## 19.3. 补丁的格式与许可

补丁的发布遵循其所修补软件的相同许可证（详见 [13.2 节 “遵守 Buildroot 许可证”](https://buildroot.org/downloads/manual/manual.html#legal-info-buildroot)）。

应在补丁头部注释中添加说明，解释补丁的作用及其必要性。

你应在每个补丁头部添加 `Signed-off-by` 声明，以便追踪变更并证明补丁遵循被修改软件的相同许可证。

如果软件在版本控制系统下，建议用上游 SCM 工具生成补丁集。

否则，将补丁头部与 `diff -purN package-version.orig/ package-version/` 命令的输出拼接。

如果你更新了已有补丁（如升级包版本时），请确保保留原有的 From 头和 Signed-off-by 标签，并在适当时更新补丁注释的其他部分。

最终补丁应如下所示：

```
configure.ac: add C++ support test

Signed-off-by: John Doe <john.doe@noname.org>

--- configure.ac.orig
+++ configure.ac
@@ -40,2 +40,12 @@

AC_PROG_MAKE_SET
+
+AC_CACHE_CHECK([whether the C++ compiler works],
+               [rw_cv_prog_cxx_works],
+               [AC_LANG_PUSH([C++])
+                AC_LINK_IFELSE([AC_LANG_PROGRAM([], [])],
+                               [rw_cv_prog_cxx_works=yes],
+                               [rw_cv_prog_cxx_works=no])
+                AC_LANG_POP([C++])])
+
+AM_CONDITIONAL([CXX_WORKS], [test "x$rw_cv_prog_cxx_works" = "xyes"])
```

## 19.4. 补丁的额外文档

理想情况下，所有补丁都应通过 `Upstream` trailer 记录上游补丁或补丁提交信息。

如果回溯应用了已被主线接受的上游补丁，建议引用该提交的 URL：

```
Upstream: <上游提交的 URL>
```

如果在 Buildroot 发现新问题，且上游也受影响（不是 Buildroot 特有问题），应向上游提交补丁，并尽可能提供提交链接：

```
Upstream: <上游邮件列表提交或合并请求的 URL>
```

如果补丁已提交但被上游拒绝，应注明原因，并说明为何仍需使用该补丁。

注意：在上述任一场景下，如对补丁做了必要修改，也应简要说明。

如果补丁无法应用到上游，应加注释说明：

```
Upstream: N/A <补丁为 Buildroot 专用的原因说明>
```

添加这些文档有助于在包版本更新时简化补丁审核流程。

## 第20章. 下载基础架构

TODO

## 第21章. 调试 Buildroot

可以对 Buildroot 构建包的各步骤进行插桩（instrumentation）。将变量 `BR2_INSTRUMENTATION_SCRIPTS` 定义为一个或多个脚本（或其他可执行文件）的路径（以空格分隔），这些脚本会在每个步骤前后被调用。脚本按顺序调用，带有三个参数：

- `start` 或 `end`，表示步骤开始或结束；
- 即将开始或刚结束的步骤名称；
- 包名称。

例如：

```
make BR2_INSTRUMENTATION_SCRIPTS="/path/to/my/script1 /path/to/my/script2"
```

步骤列表如下：

- `extract`
- `patch`
- `configure`
- `build`
- `install-host`，主机包安装到 `$(HOST_DIR)` 时
- `install-target`，目标包安装到 `$(TARGET_DIR)` 时
- `install-staging`，目标包安装到 `$(STAGING_DIR)` 时
- `install-image`，目标包安装文件到 `$(BINARIES_DIR)` 时

脚本可访问以下变量：

- `BR2_CONFIG`：Buildroot .config 文件路径
- `HOST_DIR`、`STAGING_DIR`、`TARGET_DIR`：见 [18.6.2 节 “generic-package 参考”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)
- `BUILD_DIR`：包解压和构建目录
- `BINARIES_DIR`：所有二进制文件（即镜像）存放位置
- `BASE_DIR`：输出基础目录
- `PARALLEL_JOBS`：并行进程数
