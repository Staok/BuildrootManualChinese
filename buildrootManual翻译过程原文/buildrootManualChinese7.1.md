## 第11章 常见问题与故障排查

## 11.1 启动卡在 *Starting network...* 后

如果启动过程在如下信息后似乎卡住（具体信息可能因所选软件包不同而略有差异）：

```
Freeing init memory: 3972K
Initializing random number generator... done.
Starting network...
Starting dropbear sshd: generating rsa key... generating dsa key... OK
```

这意味着系统已经运行，但没有在串口控制台上启动 shell。要让系统在串口控制台上启动 shell，需要进入 Buildroot 配置，在“系统配置（System configuration）”中，修改“启动后运行 getty（登录提示）”选项，并在“getty 选项”子菜单中设置合适的端口和波特率。这将自动调整生成系统的 `/etc/inittab` 文件，使 shell 能在正确的串口上启动。

## 11.2 为什么目标系统上没有编译器？

自 Buildroot-2012.11 版本起，已决定不再支持*目标系统上的本地编译器（native compiler）*，原因如下：

- 该功能既未维护也未测试，经常出错；
- 该功能仅适用于 Buildroot 工具链；
- Buildroot 主要面向*小型*或*超小型*目标硬件，资源有限（CPU、内存、存储），在目标上编译意义不大；
- Buildroot 旨在简化交叉编译，使目标上的本地编译变得不必要。

如果你确实需要在目标系统上有编译器，那么 Buildroot 并不适合你的需求。此时应选择如下“真正的发行版（distribution）”：

- [openembedded](http://www.openembedded.org/)
- [yocto](https://www.yoctoproject.org/)
- [Debian](https://www.debian.org/ports/)
- [Fedora](https://fedoraproject.org/wiki/Architectures)
- [openSUSE ARM](http://en.opensuse.org/Portal:ARM)
- [Arch Linux ARM](http://archlinuxarm.org/)
- ...

## 11.3 为什么目标系统上没有开发文件？

由于目标系统上没有编译器（见[11.2节“为什么目标系统上没有编译器？”](https://buildroot.org/downloads/manual/manual.html#faq-no-compiler-on-target)），因此没有必要浪费空间存放头文件或静态库。

因此，从 Buildroot-2012.11 版本起，这些文件始终会从目标系统中移除。

## 11.4 为什么目标系统上没有文档？

由于 Buildroot 主要面向*小型*或*超小型*目标硬件，资源有限（CPU、内存、存储），因此没有必要浪费空间存放文档数据。

如果你确实需要在目标系统上有文档数据，那么 Buildroot 并不适合你的需求，应选择“真正的发行版”（见[11.2节“为什么目标系统上没有编译器？”](https://buildroot.org/downloads/manual/manual.html#faq-no-compiler-on-target)）。

## 11.5 为什么有些软件包在 Buildroot 配置菜单中不可见？

如果某个软件包在 Buildroot 源码树中存在，但在配置菜单中不可见，通常是因为该软件包的某些依赖项未满足。

要了解软件包的依赖关系，可在配置菜单中搜索该软件包的符号（见[8.1节“make 技巧”](https://buildroot.org/downloads/manual/manual.html#make-tips)）。

然后，可能需要递归地启用多个选项（即未满足的依赖项），才能最终选择该软件包。

如果由于某些工具链选项未满足导致软件包不可见，则应进行完整重建（见[8.1节“make 技巧”](https://buildroot.org/downloads/manual/manual.html#make-tips)获取更多说明）。

## 11.6 为什么不能将 target 目录作为 chroot 目录使用？

有很多理由***不***要将 target 目录作为 chroot 目录，其中包括：

- 目标目录中的文件所有权、模式和权限未正确设置；
- 目标目录中未创建设备节点。

因此，通过 chroot 并将 target 目录作为新根目录运行的命令很可能会失败。

如果想在 chroot 或 NFS 根下运行目标文件系统，应使用 `images/` 目录下生成的 tarball 镜像，并以 root 身份解压。
