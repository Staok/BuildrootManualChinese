### 18.2.4. Dependencies on target and toolchain options

Many packages depend on certain options of the toolchain: the choice of C library, C++ support, thread support, RPC support, wchar support, or dynamic library support. Some packages can only be built on certain target architectures, or if an MMU is available in the processor.

These dependencies have to be expressed with the appropriate *depends on* statements in the Config.in file. Additionally, for dependencies on toolchain options, a `comment` should be displayed when the option is not enabled, so that the user knows why the package is not available. Dependencies on target architecture or MMU support should not be made visible in a comment: since it is unlikely that the user can freely choose another target, it makes little sense to show these dependencies explicitly.

The `comment` should only be visible if the `config` option itself would be visible when the toolchain option dependencies are met. This means that all other dependencies of the package (including dependencies on target architecture and MMU support) have to be repeated on the `comment` definition. To keep it clear, the `depends on` statement for these non-toolchain option should be kept separate from the `depends on` statement for the toolchain options. If there is a dependency on a config option in that same file (typically the main package) it is preferable to have a global `if … endif` construct rather than repeating the `depends on` statement on the comment and other config options.

The general format of a dependency `comment` for package foo is:

```
foo needs a toolchain w/ featA, featB, featC
```

for example:

```
mpd needs a toolchain w/ C++, threads, wchar
```

or

```
crda needs a toolchain w/ threads
```

Note that this text is kept brief on purpose, so that it will fit on a 80-character terminal.

The rest of this section enumerates the different target and toolchain options, the corresponding config symbols to depend on, and the text to use in the comment.

- Target architecture
  - Dependency symbol: `BR2_powerpc`, `BR2_mips`, … (see `arch/Config.in`)
  - Comment string: no comment to be added
- MMU support
  - Dependency symbol: `BR2_USE_MMU`
  - Comment string: no comment to be added
- Gcc `*_sync**` built-ins used for atomic operations. They are available in variants operating on 1 byte, 2 bytes, 4 bytes and 8 bytes. Since different architectures support atomic operations on different sizes, one dependency symbol is available for each size:
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_SYNC_1` for 1 byte, `BR2_TOOLCHAIN_HAS_SYNC_2` for 2 bytes, `BR2_TOOLCHAIN_HAS_SYNC_4` for 4 bytes, `BR2_TOOLCHAIN_HAS_SYNC_8` for 8 bytes.
  - Comment string: no comment to be added
- Gcc `*_atomic**` built-ins used for atomic operations.
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_ATOMIC`.
  - Comment string: no comment to be added
- Kernel headers
  - Dependency symbol: `BR2_TOOLCHAIN_HEADERS_AT_LEAST_X_Y`, (replace `X_Y` with the proper version, see `toolchain/Config.in`)
  - Comment string: `headers >= X.Y` and/or `headers <= X.Y` (replace `X.Y` with the proper version)
- GCC version
  - Dependency symbol: `BR2_TOOLCHAIN_GCC_AT_LEAST_X_Y`, (replace `X_Y` with the proper version, see `toolchain/Config.in`)
  - Comment string: `gcc >= X.Y` and/or `gcc <= X.Y` (replace `X.Y` with the proper version)
- Host GCC version
  - Dependency symbol: `BR2_HOST_GCC_AT_LEAST_X_Y`, (replace `X_Y` with the proper version, see `Config.in`)
  - Comment string: no comment to be added
  - Note that it is usually not the package itself that has a minimum host GCC version, but rather a host-package on which it depends.
- C library
  - Dependency symbol: `BR2_TOOLCHAIN_USES_GLIBC`, `BR2_TOOLCHAIN_USES_MUSL`, `BR2_TOOLCHAIN_USES_UCLIBC`
  - Comment string: for the C library, a slightly different comment text is used: `foo needs a glibc toolchain`, or `foo needs a glibc toolchain w/ C++`
- C++ support
  - Dependency symbol: `BR2_INSTALL_LIBSTDCPP`
  - Comment string: `C++`
- D support
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_DLANG`
  - Comment string: `Dlang`
- Fortran support
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_FORTRAN`
  - Comment string: `fortran`
- thread support
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_THREADS`
  - Comment string: `threads` (unless `BR2_TOOLCHAIN_HAS_THREADS_NPTL` is also needed, in which case, specifying only `NPTL` is sufficient)
- NPTL thread support
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_THREADS_NPTL`
  - Comment string: `NPTL`
- RPC support
  - Dependency symbol: `BR2_TOOLCHAIN_HAS_NATIVE_RPC`
  - Comment string: `RPC`
- wchar support
  - Dependency symbol: `BR2_USE_WCHAR`
  - Comment string: `wchar`
- dynamic library
  - Dependency symbol: `!BR2_STATIC_LIBS`
  - Comment string: `dynamic library`

### 18.2.5. Dependencies on a Linux kernel built by buildroot

Some packages need a Linux kernel to be built by buildroot. These are typically kernel modules or firmware. A comment should be added in the Config.in file to express this dependency, similar to dependencies on toolchain options. The general format is:

```
foo needs a Linux kernel to be built
```

If there is a dependency on both toolchain options and the Linux kernel, use this format:

```
foo needs a toolchain w/ featA, featB, featC and a Linux kernel to be built
```

### 18.2.6. Dependencies on udev /dev management

If a package needs udev /dev management, it should depend on symbol `BR2_PACKAGE_HAS_UDEV`, and the following comment should be added:

```
foo needs udev /dev management
```

If there is a dependency on both toolchain options and udev /dev management, use this format:

```
foo needs udev /dev management and a toolchain w/ featA, featB, featC
```

### 18.2.7. Dependencies on features provided by virtual packages

Some features can be provided by more than one package, such as the openGL libraries.

See [Section 18.12, “Infrastructure for virtual packages”](https://buildroot.org/downloads/manual/manual.html#virtual-package-tutorial) for more on the virtual packages.