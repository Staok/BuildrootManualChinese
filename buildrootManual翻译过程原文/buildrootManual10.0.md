## 18.3. The `.mk` file

Finally, here’s the hardest part. Create a file named `libfoo.mk`. It describes how the package should be downloaded, configured, built, installed, etc.

Depending on the package type, the `.mk` file must be written in a different way, using different infrastructures:

- ***\*Makefiles for generic packages\**** (not using autotools or CMake): These are based on an infrastructure similar to the one used for autotools-based packages, but require a little more work from the developer. They specify what should be done for the configuration, compilation and installation of the package. This infrastructure must be used for all packages that do not use the autotools as their build system. In the future, other specialized infrastructures might be written for other build systems. We cover them through in a [tutorial](https://buildroot.org/downloads/manual/manual.html#generic-package-tutorial) and a [reference](https://buildroot.org/downloads/manual/manual.html#generic-package-reference).
- ***\*Makefiles for autotools-based software\**** (autoconf, automake, etc.): We provide a dedicated infrastructure for such packages, since autotools is a very common build system. This infrastructure *must* be used for new packages that rely on the autotools as their build system. We cover them through a [tutorial](https://buildroot.org/downloads/manual/manual.html#autotools-package-tutorial) and [reference](https://buildroot.org/downloads/manual/manual.html#autotools-package-reference).
- ***\*Makefiles for cmake-based software\****: We provide a dedicated infrastructure for such packages, as CMake is a more and more commonly used build system and has a standardized behaviour. This infrastructure *must* be used for new packages that rely on CMake. We cover them through a [tutorial](https://buildroot.org/downloads/manual/manual.html#cmake-package-tutorial) and [reference](https://buildroot.org/downloads/manual/manual.html#cmake-package-reference).
- ***\*Makefiles for Python modules\****: We have a dedicated infrastructure for Python modules that use the `flit`, `hatch`, `pep517`, `poetry` `setuptools`, `setuptools-rust` or `maturin` mechanisms. We cover them through a [tutorial](https://buildroot.org/downloads/manual/manual.html#python-package-tutorial) and a [reference](https://buildroot.org/downloads/manual/manual.html#python-package-reference).
- ***\*Makefiles for Lua modules\****: We have a dedicated infrastructure for Lua modules available through the LuaRocks web site. We cover them through a [tutorial](https://buildroot.org/downloads/manual/manual.html#luarocks-package-tutorial) and a [reference](https://buildroot.org/downloads/manual/manual.html#luarocks-package-reference).

Further formatting details: see [the writing rules](https://buildroot.org/downloads/manual/manual.html#writing-rules-mk).

## 18.4. The `.hash` file

When possible, you must add a third file, named `libfoo.hash`, that contains the hashes of the downloaded files for the `libfoo` package. The only reason for not adding a `.hash` file is when hash checking is not possible due to how the package is downloaded.

When a package has a version selection choice, then the hash file may be stored in a subdirectory named after the version, e.g. `package/libfoo/1.2.3/libfoo.hash`. This is especially important if the different versions have different licensing terms, but they are stored in the same file. Otherwise, the hash file should stay in the package’s directory.

The hashes stored in that file are used to validate the integrity of the downloaded files and of the license files.

The format of this file is one line for each file for which to check the hash, each line with the following three fields separated by two spaces:

- the type of hash, one of:
  - `md5`, `sha1`, `sha224`, `sha256`, `sha384`, `sha512`
- the hash of the file:
  - for `md5`, 32 hexadecimal characters
  - for `sha1`, 40 hexadecimal characters
  - for `sha224`, 56 hexadecimal characters
  - for `sha256`, 64 hexadecimal characters
  - for `sha384`, 96 hexadecimal characters
  - for `sha512`, 128 hexadecimal characters
- the name of the file:
  - for a source archive: the basename of the file, without any directory component,
  - for a license file: the path as it appears in `FOO_LICENSE_FILES`.

Lines starting with a `#` sign are considered comments, and ignored. Empty lines are ignored.

There can be more than one hash for a single file, each on its own line. In this case, all hashes must match.

**Note.** Ideally, the hashes stored in this file should match the hashes published by upstream, e.g. on their website, in the e-mail announcement… If upstream provides more than one type of hash (e.g. `sha1` and `sha512`), then it is best to add all those hashes in the `.hash` file. If upstream does not provide any hash, or only provides an `md5` hash, then compute at least one strong hash yourself (preferably `sha256`, but not `md5`), and mention this in a comment line above the hashes.

**Note.** The hashes for license files are used to detect a license change when a package version is bumped. The hashes are checked during the make legal-info target run. For a package with multiple versions (like Qt5), create the hash file in a subdirectory `<packageversion>` of that package (see also [Section 19.2, “How patches are applied”](https://buildroot.org/downloads/manual/manual.html#patch-apply-order)).

The example below defines a `sha1` and a `sha256` published by upstream for the main `libfoo-1.2.3.tar.bz2` tarball, an `md5` from upstream and a locally-computed `sha256` hashes for a binary blob, a `sha256` for a downloaded patch, and an archive with no hash:

```
# Hashes from: http://www.foosoftware.org/download/libfoo-1.2.3.tar.bz2.{sha1,sha256}:
sha1  486fb55c3efa71148fe07895fd713ea3a5ae343a  libfoo-1.2.3.tar.bz2
sha256  efc8103cc3bcb06bda6a781532d12701eb081ad83e8f90004b39ab81b65d4369  libfoo-1.2.3.tar.bz2

# md5 from: http://www.foosoftware.org/download/libfoo-1.2.3.tar.bz2.md5, sha256 locally computed:
md5  2d608f3c318c6b7557d551a5a09314f03452f1a1  libfoo-data.bin
sha256  01ba4719c80b6fe911b091a7c05124b64eeece964e09c058ef8f9805daca546b  libfoo-data.bin

# Locally computed:
sha256  ff52101fb90bbfc3fe9475e425688c660f46216d7e751c4bbdb1dc85cdccacb9  libfoo-fix-blabla.patch

# Hash for license files:
sha256  a45a845012742796534f7e91fe623262ccfb99460a2bd04015bd28d66fba95b8  COPYING
sha256  01b1f9f2c8ee648a7a596a1abe8aa4ed7899b1c9e5551bda06da6e422b04aa55  doc/COPYING.LGPL
```

If the `.hash` file is present, and it contains one or more hashes for a downloaded file, the hash(es) computed by Buildroot (after download) must match the hash(es) stored in the `.hash` file. If one or more hashes do not match, Buildroot considers this an error, deletes the downloaded file, and aborts.

If the `.hash` file is present, but it does not contain a hash for a downloaded file, Buildroot considers this an error and aborts. However, the downloaded file is left in the download directory since this typically indicates that the `.hash` file is wrong but the downloaded file is probably OK.

Hashes are currently checked for files fetched from http/ftp servers, Git or subversion repositories, files copied using scp and local files. Hashes are not checked for other version control systems (such as CVS, mercurial) because Buildroot currently does not generate reproducible tarballs when source code is fetched from such version control systems.

Additionally, for packages for which it is possible to specify a custom version (e.g. a custom version string, a remote tarball URL, or a VCS repository location and changeset), Buildroot can’t carry hashes for those. It is however possible to [provide a list of extra hashes](https://buildroot.org/downloads/manual/manual.html#customize-hashes) that can cover such cases.

Hashes should only be added in `.hash` files for files that are guaranteed to be stable. For example, patches auto-generated by Github are not guaranteed to be stable, and therefore their hashes can change over time. Such patches should not be downloaded, and instead be added locally to the package folder.

If the `.hash` file is missing, then no check is done at all.
