## 16.3. The `genimage.cfg` file

`genimage.cfg` files contain the output image layout that genimage utility uses to create final .img file.

An example follows:

```
image efi-part.vfat {
        vfat {
                file EFI {
                        image = "efi-part/EFI"
                }

                file Image {
                        image = "Image"
                }
        }

        size = 32M
}

image sdimage.img {
        hdimage {
        }

        partition u-boot {
                image = "efi-part.vfat"
                offset = 8K
        }

        partition root {
                image = "rootfs.ext2"
                size = 512M
        }
}
```

- Every `section`(i.e. hdimage, vfat etc.), `partition` must be indented with one tab.
- Every `file` or other `subnode` must be indented with two tabs.
- Every node(`section`, `partition`, `file`, `subnode`) must have an open curly bracket on the same line of the node’s name, while the closing one must be on a newline and after it a newline must be added except for the last one node. Same goes for its option, for example option `size` `=`.
- Every `option`(i.e. `image`, `offset`, `size`) must have the `=` assignment one space from it and one space from the value specified.
- Filename must at least begin with genimage prefix and have the .cfg extension to be easy to recognize.
- Allowed notations for `offset` and `size` options are: `G`, `M`, `K` (not `k`). If it’s not possible to express a precise byte count with notations above then use hexadecimal `0x` prefix or, as last chance, the byte count. In comments instead use `GB`, `MB`, `KB` (not `kb`) in place of `G`, `M`, `K`.
- For GPT partitions, the `partition-type-uuid` value must be `U` for the EFI System Partition (expanded to `c12a7328-f81f-11d2-ba4b-00a0c93ec93b` by *genimage*), `F` for a FAT partition (expanded to `ebd0a0a2-b9e5-4433-87c0-68b6b72699c7` by *genimage*) or `L` for the root filesystem or other filesystems (expanded to `0fc63daf-8483-4772-8e79-3d69d8477de4` by *genimage*). Even though `L` is the default value of *genimage*, we prefer to have it explicitly specified in our `genimage.cfg` files. Finally, these shortcuts should be used without double quotes, e.g `partition-type-uuid = U`. If an explicit GUID is specified, lower-case letters should be used.

The `genimage.cfg` files are the input for the genimage tool used in Buildroot to generate the final image file(i.e. sdcard.img). For further details about the *genimage* language, refer to https://github.com/pengutronix/genimage/blob/master/README.rst.

## 16.4. The documentation

The documentation uses the [asciidoc](https://asciidoc-py.github.io/) format.

For further details about the asciidoc syntax, refer to https://asciidoc-py.github.io/userguide.html.

## 16.5. Support scripts

Some scripts in the `support/` and `utils/` directories are written in Python and should follow the [PEP8 Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/).

## Chapter 17. Adding support for a particular board

Buildroot contains basic configurations for several publicly available hardware boards, so that users of such a board can easily build a system that is known to work. You are welcome to add support for other boards to Buildroot too.

To do so, you need to create a normal Buildroot configuration that builds a basic system for the hardware: (internal) toolchain, kernel, bootloader, filesystem and a simple BusyBox-only userspace. No specific package should be selected: the configuration should be as minimal as possible, and should only build a working basic BusyBox system for the target platform. You can of course use more complicated configurations for your internal projects, but the Buildroot project will only integrate basic board configurations. This is because package selections are highly application-specific.

Once you have a known working configuration, run `make savedefconfig`. This will generate a minimal `defconfig` file at the root of the Buildroot source tree. Move this file into the `configs/` directory, and rename it `<boardname>_defconfig`.

Always use fixed versions or commit hashes for the different components, not the "latest" version. For example, set `BR2_LINUX_KERNEL_CUSTOM_VERSION=y` and `BR2_LINUX_KERNEL_CUSTOM_VERSION_VALUE` to the kernel version you tested with. If you are using the buildroot toolchain `BR2_TOOLCHAIN_BUILDROOT` (which is the default), additionally ensure that the same kernel headers are used (`BR2_KERNEL_HEADERS_AS_KERNEL`, which is also the default) and set the custom kernel headers series to match your kernel version (`BR2_PACKAGE_HOST_LINUX_HEADERS_CUSTOM_*`).

It is recommended to use as much as possible upstream versions of the Linux kernel and bootloaders, and to use as much as possible default kernel and bootloader configurations. If they are incorrect for your board, or no default exists, we encourage you to send fixes to the corresponding upstream projects.

However, in the mean time, you may want to store kernel or bootloader configuration or patches specific to your target platform. To do so, create a directory `board/<manufacturer>` and a subdirectory `board/<manufacturer>/<boardname>`. You can then store your patches and configurations in these directories, and reference them from the main Buildroot configuration. Refer to [Chapter 9, *Project-specific customization*](https://buildroot.org/downloads/manual/manual.html#customize) for more details.

Before submitting patches for new boards it is recommended to test it by building it using latest gitlab-CI docker container. To do this use `utils/docker-run` script and inside it issue these commands:

```
 $ make <boardname>_defconfig
 $ make
```

By default, Buildroot developers use the official image hosted on the [gitlab.com registry](https://gitlab.com/buildroot.org/buildroot/container_registry/2395076) and it should be convenient for most usage. If you still want to build your own docker image, you can base it off the official image as the `FROM` directive of your own *Dockerfile*:

```
FROM registry.gitlab.com/buildroot.org/buildroot/base:YYYYMMDD.HHMM
RUN ...
COPY ...
```

The current version *YYYYMMDD.HHMM* can be found in the `.gitlab-ci.yml` file at the top of the Buildroot source tree; all past versions are listed in the aforementioned registry as well.