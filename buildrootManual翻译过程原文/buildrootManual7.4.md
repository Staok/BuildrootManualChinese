## 13.2. Complying with the Buildroot license

Buildroot itself is an open source software, released under the [GNU General Public License, version 2](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) or (at your option) any later version, with the exception of the package patches detailed below. However, being a build system, it is not normally part of the end product: if you develop the root filesystem, kernel, bootloader or toolchain for a device, the code of Buildroot is only present on the development machine, not in the device storage.

Nevertheless, the general view of the Buildroot developers is that you should release the Buildroot source code along with the source code of other packages when releasing a product that contains GPL-licensed software. This is because the [GNU GPL](http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) defines the "*complete source code*" for an executable work as "*all the source code for all modules it contains, plus any associated interface definition files, plus the scripts used to control compilation and installation of the executable*". Buildroot is part of the *scripts used to control compilation and installation of the executable*, and as such it is considered part of the material that must be redistributed.

Keep in mind that this is only the Buildroot developers' opinion, and you should consult your legal department or lawyer in case of any doubt.

## 13.2.1. Patches to packages

Buildroot also bundles patch files, which are applied to the sources of the various packages. Those patches are not covered by the license of Buildroot. Instead, they are covered by the license of the software to which the patches are applied. When said software is available under multiple licenses, the Buildroot patches are only provided under the publicly accessible licenses.

See [Chapter 19, *Patching a package*](https://buildroot.org/downloads/manual/manual.html#patch-policy) for the technical details.

## Chapter 14. Beyond Buildroot

## 14.1. Boot the generated images

### 14.1.1. NFS boot

To achieve NFS-boot, enable *tar root filesystem* in the *Filesystem images* menu.

After a complete build, just run the following commands to setup the NFS-root directory:

```
sudo tar -xavf /path/to/output_dir/rootfs.tar -C /path/to/nfs_root_dir
```

Remember to add this path to `/etc/exports`.

Then, you can execute a NFS-boot from your target.

### 14.1.2. Live CD

To build a live CD image, enable the *iso image* option in the *Filesystem images* menu. Note that this option is only available on the x86 and x86-64 architectures, and if you are building your kernel with Buildroot.

You can build a live CD image with either IsoLinux, Grub or Grub 2 as a bootloader, but only Isolinux supports making this image usable both as a live CD and live USB (through the *Build hybrid image* option).

You can test your live CD image using QEMU:

```
qemu-system-i386 -cdrom output/images/rootfs.iso9660
```

Or use it as a hard-drive image if it is a hybrid ISO:

```
qemu-system-i386 -hda output/images/rootfs.iso9660
```

It can be easily flashed to a USB drive with `dd`:

```
dd if=output/images/rootfs.iso9660 of=/dev/sdb
```

## 14.2. Chroot

If you want to chroot in a generated image, then there are few thing you should be aware of:

- you should setup the new root from the *tar root filesystem* image;
- either the selected target architecture is compatible with your host machine, or you should use some `qemu-*` binary and correctly set it within the `binfmt` properties to be able to run the binaries built for the target on your host machine;
- Buildroot does not currently provide `host-qemu` and `binfmt` correctly built and set for that kind of use.

