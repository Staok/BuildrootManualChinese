## 11.9. How does Buildroot support Y2038?

There are multiple situations to consider:

- On 64-bit architectures, there is no problem, as `time_t` has always been 64-bit.
- On 32-bit architectures, the situation depends on the C library:
  - With *uclibc-ng*, there is support for 64-bit `time_t` on 32-bit architectures since version 1.0.46, so systems using *uclibc-ng* on 32-bit platforms will be Y2038 compatible when UCLIBC_USE_TIME64 is y. This is the default since 1.0.49.
  - With *musl*, 64-bit `time_t` has always been used on 32-bit architectures, so systems using *musl* on 32-bit platforms are Y2038 compatible.
  - With *glibc*, 64-bit `time_t` on 32-bit architectures is enabled by the Buildroot option `BR2_TIME_BITS_64`. With this option enabled, systems using *glibc* on 32-bit platforms are Y2038 compatible.

Note that the above only comments about the capabilities of the C library. Individual user-space libraries or applications, even when built in a Y2038-compatible setup, can exhibit incorrect behavior if they do not make correct use of the time APIs and types.

## Chapter 12. Known issues

- It is not possible to pass extra linker options via `BR2_TARGET_LDFLAGS` if such options contain a `$` sign. For example, the following is known to break: `BR2_TARGET_LDFLAGS="-Wl,-rpath='$ORIGIN/../lib'"`
- The `libffi` package is not supported on the SuperH 2 and ARMv7-M architectures.
- The `prboom` package triggers a compiler failure with the SuperH 4 compiler from Sourcery CodeBench, version 2012.09.

## Chapter 13. Legal notice and licensing

## 13.1. Complying with open source licenses

All of the end products of Buildroot (toolchain, root filesystem, kernel, bootloaders) contain open source software, released under various licenses.

Using open source software gives you the freedom to build rich embedded systems, choosing from a wide range of packages, but also imposes some obligations that you must know and honour. Some licenses require you to publish the license text in the documentation of your product. Others require you to redistribute the source code of the software to those that receive your product.

The exact requirements of each license are documented in each package, and it is your responsibility (or that of your legal office) to comply with those requirements. To make this easier for you, Buildroot can collect for you some material you will probably need. To produce this material, after you have configured Buildroot with `make menuconfig`, `make xconfig` or `make gconfig`, run:

```
make legal-info
```

Buildroot will collect legally-relevant material in your output directory, under the `legal-info/` subdirectory. There you will find:

- A `README` file, that summarizes the produced material and contains warnings about material that Buildroot could not produce.
- `buildroot.config`: this is the Buildroot configuration file that is usually produced with `make menuconfig`, and which is necessary to reproduce the build.
- The source code for all packages; this is saved in the `sources/` and `host-sources/` subdirectories for target and host packages respectively. The source code for packages that set `<PKG>_REDISTRIBUTE = NO` will not be saved. Patches that were applied are also saved, along with a file named `series` that lists the patches in the order they were applied. Patches are under the same license as the files that they modify. Note: Buildroot applies additional patches to Libtool scripts of autotools-based packages. These patches can be found under `support/libtool` in the Buildroot source and, due to technical limitations, are not saved with the package sources. You may need to collect them manually.
- A manifest file (one for host and one for target packages) listing the configured packages, their version, license and related information. Some of this information might not be defined in Buildroot; such items are marked as "unknown".
- The license texts of all packages, in the `licenses/` and `host-licenses/` subdirectories for target and host packages respectively. If the license file(s) are not defined in Buildroot, the file is not produced and a warning in the `README` indicates this.

Please note that the aim of the `legal-info` feature of Buildroot is to produce all the material that is somehow relevant for legal compliance with the package licenses. Buildroot does not try to produce the exact material that you must somehow make public. Certainly, more material is produced than is needed for a strict legal compliance. For example, it produces the source code for packages released under BSD-like licenses, that you are not required to redistribute in source form.

Moreover, due to technical limitations, Buildroot does not produce some material that you will or may need, such as the toolchain source code for some of the external toolchains and the Buildroot source code itself. When you run `make legal-info`, Buildroot produces warnings in the `README` file to inform you of relevant material that could not be saved.

Finally, keep in mind that the output of `make legal-info` is based on declarative statements in each of the packages recipes. The Buildroot developers try to do their best to keep those declarative statements as accurate as possible, to the best of their knowledge. However, it is very well possible that those declarative statements are not all fully accurate nor exhaustive. You (or your legal department) *have* to check the output of `make legal-info` before using it as your own compliance delivery. See the *NO WARRANTY* clauses (clauses 11 and 12) in the `COPYING` file at the root of the Buildroot distribution.

