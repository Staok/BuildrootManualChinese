## Chapter 4. Buildroot quick start

***\*Important\****: you can and should ***\*build everything as a normal user\****. There is no need to be root to configure and use Buildroot. By running all commands as a regular user, you protect your system against packages behaving badly during compilation and installation.

The first step when using Buildroot is to create a configuration. Buildroot has a nice configuration tool similar to the one you can find in the [Linux kernel](http://www.kernel.org/) or in [BusyBox](http://www.busybox.net/).

From the buildroot directory, run

```
 $ make menuconfig
```

for the original curses-based configurator, or

```
 $ make nconfig
```

for the new curses-based configurator, or

```
 $ make xconfig
```

for the Qt-based configurator, or

```
 $ make gconfig
```

for the GTK-based configurator.

All of these "make" commands will need to build a configuration utility (including the interface), so you may need to install "development" packages for relevant libraries used by the configuration utilities. Refer to [Chapter 2, *System requirements*](https://buildroot.org/downloads/manual/manual.html#requirement) for more details, specifically the [optional requirements](https://buildroot.org/downloads/manual/manual.html#requirement-optional) to get the dependencies of your favorite interface.

For each menu entry in the configuration tool, you can find associated help that describes the purpose of the entry. Refer to [Chapter 6, *Buildroot configuration*](https://buildroot.org/downloads/manual/manual.html#configure) for details on some specific configuration aspects.

Once everything is configured, the configuration tool generates a `.config` file that contains the entire configuration. This file will be read by the top-level Makefile.

To start the build process, simply run:

```
 $ make
```

By default, Buildroot does not support top-level parallel build, so running `make -jN` is not necessary. There is however experimental support for top-level parallel build, see [Section 8.12, “Top-level parallel build”](https://buildroot.org/downloads/manual/manual.html#top-level-parallel-build).

The `make` command will generally perform the following steps:

- download source files (as required);
- configure, build and install the cross-compilation toolchain, or simply import an external toolchain;
- configure, build and install selected target packages;
- build a kernel image, if selected;
- build a bootloader image, if selected;
- create a root filesystem in selected formats.

Buildroot output is stored in a single directory, `output/`. This directory contains several subdirectories:

- `images/` where all the images (kernel image, bootloader and root filesystem images) are stored. These are the files you need to put on your target system.
- `build/` where all the components are built (this includes tools needed by Buildroot on the host and packages compiled for the target). This directory contains one subdirectory for each of these components.
- `host/` contains both the tools built for the host, and the sysroot of the target toolchain. The former is an installation of tools compiled for the host that are needed for the proper execution of Buildroot, including the cross-compilation toolchain. The latter is a hierarchy similar to a root filesystem hierarchy. It contains the headers and libraries of all user-space packages that provide and install libraries used by other packages. However, this directory is *not* intended to be the root filesystem for the target: it contains a lot of development files, unstripped binaries and libraries that make it far too big for an embedded system. These development files are used to compile libraries and applications for the target that depend on other libraries.
- `staging/` is a symlink to the target toolchain sysroot inside `host/`, which exists for backwards compatibility.
- `target/` which contains *almost* the complete root filesystem for the target: everything needed is present except the device files in `/dev/` (Buildroot can’t create them because Buildroot doesn’t run as root and doesn’t want to run as root). Also, it doesn’t have the correct permissions (e.g. setuid for the busybox binary). Therefore, this directory ***\*should not be used on your target\****. Instead, you should use one of the images built in the `images/` directory. If you need an extracted image of the root filesystem for booting over NFS, then use the tarball image generated in `images/` and extract it as root. Compared to `staging/`, `target/` contains only the files and libraries needed to run the selected target applications: the development files (headers, etc.) are not present, the binaries are stripped.

These commands, `make menuconfig|nconfig|gconfig|xconfig` and `make`, are the basic ones that allow to easily and quickly generate images fitting your needs, with all the features and applications you enabled.

More details about the "make" command usage are given in [Section 8.1, “*make* tips”](https://buildroot.org/downloads/manual/manual.html#make-tips).

## Chapter 5. Community resources

Like any open source project, Buildroot has different ways to share information in its community and outside.

Each of those ways may interest you if you are looking for some help, want to understand Buildroot or contribute to the project.

- Mailing List

  Buildroot has a mailing list for discussion and development. It is the main method of interaction for Buildroot users and developers.Only subscribers to the Buildroot mailing list are allowed to post to this list. You can subscribe via the [mailing list info page](http://lists.buildroot.org/mailman/listinfo/buildroot).Mails that are sent to the mailing list are also available in the mailing list archives, available through [Mailman](http://lists.buildroot.org/pipermail/buildroot) or at [lore.kernel.org](https://lore.kernel.org/buildroot/).

- IRC

  The Buildroot IRC channel [#buildroot](irc://irc.oftc.net/#buildroot) is hosted on [OFTC](https://www.oftc.net/WebChat/). It is a useful place to ask quick questions or discuss on certain topics.When asking for help on IRC, share relevant logs or pieces of code using a code sharing website, such as https://paste.ack.tf/.Note that for certain questions, posting to the mailing list may be better as it will reach more people, both developers and users.

- Bug tracker

  Bugs in Buildroot can be reported via the mailing list or alternatively via the [Buildroot bugtracker](https://gitlab.com/buildroot.org/buildroot/-/issues). Please refer to [Section 22.6, “Reporting issues/bugs or getting help”](https://buildroot.org/downloads/manual/manual.html#reporting-bugs) before creating a bug report.

- Wiki

  [The Buildroot wiki page](http://elinux.org/Buildroot) is hosted on the [eLinux](http://elinux.org/) wiki. It contains some useful links, an overview of past and upcoming events, and a TODO list.

- Patchwork

  Patchwork is a web-based patch tracking system designed to facilitate the contribution and management of contributions to an open-source project. Patches that have been sent to a mailing list are 'caught' by the system, and appear on a web page. Any comments posted that reference the patch are appended to the patch page too. For more information on Patchwork see http://jk.ozlabs.org/projects/patchwork/.Buildroot’s Patchwork website is mainly for use by Buildroot’s maintainer to ensure patches aren’t missed. It is also used by Buildroot patch reviewers (see also [Section 22.3.1, “Applying Patches from Patchwork”](https://buildroot.org/downloads/manual/manual.html#apply-patches-patchwork)). However, since the website exposes patches and their corresponding review comments in a clean and concise web interface, it can be useful for all Buildroot developers.The Buildroot patch management interface is available at https://patchwork.ozlabs.org/project/buildroot/list/.