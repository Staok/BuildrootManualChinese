## Chapter 11. Frequently Asked Questions & Troubleshooting

## 11.1. The boot hangs after *Starting network…*

If the boot process seems to hang after the following messages (messages not necessarily exactly similar, depending on the list of packages selected):

```
Freeing init memory: 3972K
Initializing random number generator... done.
Starting network...
Starting dropbear sshd: generating rsa key... generating dsa key... OK
```

then it means that your system is running, but didn’t start a shell on the serial console. In order to have the system start a shell on your serial console, you have to go into the Buildroot configuration, in `System configuration`, modify `Run a getty (login prompt) after boot` and set the appropriate port and baud rate in the `getty options` submenu. This will automatically tune the `/etc/inittab` file of the generated system so that a shell starts on the correct serial port.

## 11.2. Why is there no compiler on the target?

It has been decided that support for the *native compiler on the target* would be stopped from the Buildroot-2012.11 release because:

- this feature was neither maintained nor tested, and often broken;
- this feature was only available for Buildroot toolchains;
- Buildroot mostly targets *small* or *very small* target hardware with limited resource onboard (CPU, ram, mass-storage), for which compiling on the target does not make much sense;
- Buildroot aims at easing the cross-compilation, making native compilation on the target unnecessary.

If you need a compiler on your target anyway, then Buildroot is not suitable for your purpose. In such case, you need a *real distribution* and you should opt for something like:

- [openembedded](http://www.openembedded.org/)
- [yocto](https://www.yoctoproject.org/)
- [Debian](https://www.debian.org/ports/)
- [Fedora](https://fedoraproject.org/wiki/Architectures)
- [openSUSE ARM](http://en.opensuse.org/Portal:ARM)
- [Arch Linux ARM](http://archlinuxarm.org/)
- …

## 11.3. Why are there no development files on the target?

Since there is no compiler available on the target (see [Section 11.2, “Why is there no compiler on the target?”](https://buildroot.org/downloads/manual/manual.html#faq-no-compiler-on-target)), it does not make sense to waste space with headers or static libraries.

Therefore, those files are always removed from the target since the Buildroot-2012.11 release.

## 11.4. Why is there no documentation on the target?

Because Buildroot mostly targets *small* or *very small* target hardware with limited resource onboard (CPU, ram, mass-storage), it does not make sense to waste space with the documentation data.

If you need documentation data on your target anyway, then Buildroot is not suitable for your purpose, and you should look for a *real distribution* (see: [Section 11.2, “Why is there no compiler on the target?”](https://buildroot.org/downloads/manual/manual.html#faq-no-compiler-on-target)).

## 11.5. Why are some packages not visible in the Buildroot config menu?

If a package exists in the Buildroot tree and does not appear in the config menu, this most likely means that some of the package’s dependencies are not met.

To know more about the dependencies of a package, search for the package symbol in the config menu (see [Section 8.1, “*make* tips”](https://buildroot.org/downloads/manual/manual.html#make-tips)).

Then, you may have to recursively enable several options (which correspond to the unmet dependencies) to finally be able to select the package.

If the package is not visible due to some unmet toolchain options, then you should certainly run a full rebuild (see [Section 8.1, “*make* tips”](https://buildroot.org/downloads/manual/manual.html#make-tips) for more explanations).

## 11.6. Why not use the target directory as a chroot directory?

There are plenty of reasons to ***\*not\**** use the target directory a chroot one, among these:

- file ownerships, modes and permissions are not correctly set in the target directory;
- device nodes are not created in the target directory.

For these reasons, commands run through chroot, using the target directory as the new root, will most likely fail.

If you want to run the target filesystem inside a chroot, or as an NFS root, then use the tarball image generated in `images/` and extract it as root.

