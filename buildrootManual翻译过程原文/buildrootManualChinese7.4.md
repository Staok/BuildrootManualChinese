## 13.2 遵守 Buildroot 许可证

Buildroot 本身是开源软件，采用 [GNU 通用公共许可证第2版](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) 或（可选）任何更高版本发布，除了下文所述的包补丁外。然而，作为构建系统，Buildroot 通常不会出现在最终产品中：如果你为设备开发根文件系统、内核、引导加载程序或工具链，Buildroot 代码只存在于开发主机，而不会存储在设备中。

尽管如此，Buildroot 开发者普遍认为，当你发布包含 GPL 许可软件的产品时，应一并发布 Buildroot 源码。这是因为 [GNU GPL](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) 规定“*完整源码*”包括“*它包含的所有模块的源码、所有相关接口定义文件，以及用于控制可执行文件编译和安装的脚本*”。Buildroot 属于*用于控制可执行文件编译和安装的脚本*，因此被认为是必须一同分发的材料。

请注意，这只是 Buildroot 开发者的观点，如有疑问应咨询你的法务部门或律师。

## 13.2.1 软件包补丁（Patches to packages）

Buildroot 还包含补丁文件，这些补丁会应用到各个软件包的源码上。这些补丁不受 Buildroot 许可证约束，而是受被补丁所应用软件的许可证约束。如果该软件有多种许可证，Buildroot 补丁仅在公开可用的许可证下提供。

技术细节见[第19章“为软件包打补丁”](https://buildroot.org/downloads/manual/manual.html#patch-policy)。

## 第14章 超越 Buildroot

## 14.1 启动生成的镜像

### 14.1.1 NFS 启动（NFS boot）

要实现 NFS 启动，在“文件系统镜像（Filesystem images）”菜单中启用 *tar 根文件系统* 选项。

完整构建后，运行以下命令设置 NFS 根目录：

```
sudo tar -xavf /path/to/output_dir/rootfs.tar -C /path/to/nfs_root_dir
```

记得将该路径添加到 `/etc/exports`。

然后即可在目标机上通过 NFS 启动。

### 14.1.2 Live CD

要构建 Live CD 镜像，在“文件系统镜像”菜单中启用 *iso 镜像* 选项。注意，该选项仅在 x86 和 x86-64 架构下可用，且需用 Buildroot 构建内核。

Live CD 镜像可用 IsoLinux、Grub 或 Grub 2 作为引导加载程序，但只有 Isolinux 支持将该镜像同时作为 Live CD 和 Live USB（通过 *构建混合镜像* 选项）。

可用 QEMU 测试 Live CD 镜像：

```
qemu-system-i386 -cdrom output/images/rootfs.iso9660
```

如果是混合 ISO，也可作为硬盘镜像使用：

```
qemu-system-i386 -hda output/images/rootfs.iso9660
```

可用 `dd` 轻松写入 U 盘：

```
dd if=output/images/rootfs.iso9660 of=/dev/sdb
```

## 14.2 Chroot

如果你想在生成的镜像中 chroot，需要注意：

- 应从 *tar 根文件系统* 镜像设置新根目录；
- 目标架构要么与主机兼容，要么需用 `qemu-*` 二进制并正确设置 `binfmt` 属性，才能在主机上运行为目标构建的二进制文件；
- Buildroot 当前不提供已正确构建和设置的 `host-qemu` 和 `binfmt` 用于此类用途。
