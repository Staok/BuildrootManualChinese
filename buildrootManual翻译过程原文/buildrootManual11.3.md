- `LIBFOO_USERS` lists the users to create for this package, if it installs a program you want to run as a specific user (e.g. as a daemon, or as a cron-job). The syntax is similar in spirit to the makedevs one, and is described in the [Chapter 26, *Makeusers syntax documentation*](https://buildroot.org/downloads/manual/manual.html#makeuser-syntax). This variable is optional.

- `LIBFOO_LICENSE` defines the license (or licenses) under which the package is released. This name will appear in the manifest file produced by `make legal-info`. If the license appears in [the SPDX License List](https://spdx.org/licenses/), use the SPDX short identifier to make the manifest file uniform. Otherwise, describe the license in a precise and concise way, avoiding ambiguous names such as `BSD` which actually name a family of licenses. This variable is optional. If it is not defined, `unknown` will appear in the `license` field of the manifest file for this package. The expected format for this variable must comply with the following rules:

  - If different parts of the package are released under different licenses, then `comma` separate licenses (e.g. `LIBFOO_LICENSE = GPL-2.0+, LGPL-2.1+`). If there is clear distinction between which component is licensed under what license, then annotate the license with that component, between parenthesis (e.g. `LIBFOO_LICENSE = GPL-2.0+ (programs), LGPL-2.1+ (libraries)`).
  - If some licenses are conditioned on a sub-option being enabled, append the conditional licenses with a comma (e.g.: `FOO_LICENSE += , GPL-2.0+ (programs)`); the infrastructure will internally remove the space before the comma.
  - If the package is dual licensed, then separate licenses with the `or` keyword (e.g. `LIBFOO_LICENSE = AFL-2.1 or GPL-2.0+`).

- `LIBFOO_LICENSE_FILES` is a space-separated list of files in the package tarball that contain the license(s) under which the package is released. `make legal-info` copies all of these files in the `legal-info` directory. See [Chapter 13, *Legal notice and licensing*](https://buildroot.org/downloads/manual/manual.html#legal-info) for more information. This variable is optional. If it is not defined, a warning will be produced to let you know, and `not saved` will appear in the `license files` field of the manifest file for this package.

- `LIBFOO_ACTUAL_SOURCE_TARBALL` only applies to packages whose `LIBFOO_SITE` / `LIBFOO_SOURCE` pair points to an archive that does not actually contain source code, but binary code. This a very uncommon case, only known to apply to external toolchains which come already compiled, although theoretically it might apply to other packages. In such cases a separate tarball is usually available with the actual source code. Set `LIBFOO_ACTUAL_SOURCE_TARBALL` to the name of the actual source code archive and Buildroot will download it and use it when you run `make legal-info` to collect legally-relevant material. Note this file will not be downloaded during regular builds nor by `make source`.

- `LIBFOO_ACTUAL_SOURCE_SITE` provides the location of the actual source tarball. The default value is `LIBFOO_SITE`, so you don’t need to set this variable if the binary and source archives are hosted on the same directory. If `LIBFOO_ACTUAL_SOURCE_TARBALL` is not set, it doesn’t make sense to define `LIBFOO_ACTUAL_SOURCE_SITE`.

- `LIBFOO_REDISTRIBUTE` can be set to `YES` (default) or `NO` to indicate if the package source code is allowed to be redistributed. Set it to `NO` for non-opensource packages: Buildroot will not save the source code for this package when collecting the `legal-info`.

- `LIBFOO_FLAT_STACKSIZE` defines the stack size of an application built into the FLAT binary format. The application stack size on the NOMMU architecture processors can’t be enlarged at run time. The default stack size for the FLAT binary format is only 4k bytes. If the application consumes more stack, append the required number here.

- `LIBFOO_BIN_ARCH_EXCLUDE` is a space-separated list of paths (relative to the target directory) to ignore when checking that the package installs correctly cross-compiled binaries. You seldom need to set this variable, unless the package installs binary blobs outside the default locations, `/lib/firmware`, `/usr/lib/firmware`, `/lib/modules`, `/usr/lib/modules`, and `/usr/share`, which are automatically excluded.

- `LIBFOO_IGNORE_CVES` is a space-separated list of CVEs that tells Buildroot CVE tracking tools which CVEs should be ignored for this package. This is typically used when the CVE is fixed by a patch in the package, or when the CVE for some reason does not affect the Buildroot package. A Makefile comment must always precede the addition of a CVE to this variable. Example:

  ```
  # 0001-fix-cve-2020-12345.patch
  LIBFOO_IGNORE_CVES += CVE-2020-12345
  # only when built with libbaz, which Buildroot doesn't support
  LIBFOO_IGNORE_CVES += CVE-2020-54321
  ```
