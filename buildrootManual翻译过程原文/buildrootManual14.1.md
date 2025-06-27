## 18.16. Infrastructure for Meson-based packages

### 18.16.1. `meson-package` tutorial

[Meson](http://mesonbuild.com/) is an open source build system meant to be both extremely fast, and, even more importantly, as user friendly as possible. It uses [Ninja](https://ninja-build.org/) as a companion tool to perform the actual build operations.

Let’s see how to write a `.mk` file for a Meson-based package, with an example:

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.0
08: FOO_SOURCE = foo-$(FOO_VERSION).tar.gz
09: FOO_SITE = http://www.foosoftware.org/download
10: FOO_LICENSE = GPL-3.0+
11: FOO_LICENSE_FILES = COPYING
12: FOO_INSTALL_STAGING = YES
13:
14: FOO_DEPENDENCIES = host-pkgconf bar
15:
16: ifeq ($(BR2_PACKAGE_BAZ),y)
17: FOO_CONF_OPTS += -Dbaz=true
18: FOO_DEPENDENCIES += baz
19: else
20: FOO_CONF_OPTS += -Dbaz=false
21: endif
22:
23: $(eval $(meson-package))
```

The Makefile starts with the definition of the standard variables for package declaration (lines 7 to 11).

On line line 23, we invoke the `meson-package` macro that generates all the Makefile rules that actually allows the package to be built.

In the example, `host-pkgconf` and `bar` are declared as dependencies in `FOO_DEPENDENCIES` at line 14 because the Meson build file of `foo` uses `pkg-config` to determine the compilation flags and libraries of package `bar`.

Note that it is not necessary to add `host-meson` in the `FOO_DEPENDENCIES` variable of a package, since this basic dependency is automatically added as needed by the Meson package infrastructure.

If the "baz" package is selected, then support for the "baz" feature in "foo" is activated by adding `-Dbaz=true` to `FOO_CONF_OPTS` at line 17, as specified in the `meson_options.txt` file in "foo" source tree. The "baz" package is also added to `FOO_DEPENDENCIES`. Note that the support for `baz` is explicitly disabled at line 20, if the package is not selected.

To sum it up, to add a new meson-based package, the Makefile example can be copied verbatim then edited to replace all occurrences of `FOO` with the uppercase name of the new package and update the values of the standard variables.

### 18.16.2. `meson-package` reference

The main macro of the Meson package infrastructure is `meson-package`. It is similar to the `generic-package` macro. The ability to have target and host packages is also available, with the `host-meson-package` macro.

Just like the generic infrastructure, the Meson infrastructure works by defining a number of variables before calling the `meson-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the Meson infrastructure.

A few additional variables, specific to the Meson infrastructure, can also be defined. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them.

- `FOO_SUBDIR` may contain the name of a subdirectory inside the package that contains the main meson.build file. This is useful, if for example, the main meson.build file is not at the root of the tree extracted by the tarball. If `HOST_FOO_SUBDIR` is not specified, it defaults to `FOO_SUBDIR`.
- `FOO_CONF_ENV`, to specify additional environment variables to pass to `meson` for the configuration step. By default, empty.
- `FOO_CONF_OPTS`, to specify additional options to pass to `meson` for the configuration step. By default, empty.
- `FOO_CFLAGS`, to specify compiler arguments added to the package specific `cross-compile.conf` file `c_args` property. By default, the value of `TARGET_CFLAGS`.
- `FOO_CXXFLAGS`, to specify compiler arguments added to the package specific `cross-compile.conf` file `cpp_args` property. By default, the value of `TARGET_CXXFLAGS`.
- `FOO_LDFLAGS`, to specify compiler arguments added to the package specific `cross-compile.conf` file `c_link_args` and `cpp_link_args` properties. By default, the value of `TARGET_LDFLAGS`.
- `FOO_MESON_EXTRA_BINARIES`, to specify a space-separated list of programs to add to the `[binaries]` section of the meson `cross-compilation.conf` configuration file. The format is `program-name='/path/to/program'`, with no space around the `=` sign, and with the path of the program between single quotes. By default, empty. Note that Buildroot already sets the correct values for `c`, `cpp`, `ar`, `strip`, and `pkgconfig`.
- `FOO_MESON_EXTRA_PROPERTIES`, to specify a space-separated list of properties to add to the `[properties]` section of the meson `cross-compilation.conf` configuration file. The format is `property-name=<value>` with no space around the `=` sign, and with single quotes around string values. By default, empty. Note that Buildroot already sets values for `needs_exe_wrapper`, `c_args`, `c_link_args`, `cpp_args`, `cpp_link_args`, `sys_root`, and `pkg_config_libdir`.
- `FOO_NINJA_ENV`, to specify additional environment variables to pass to `ninja`, meson companion tool in charge of the build operations. By default, empty.
- `FOO_NINJA_OPTS`, to specify a space-separated list of targets to build. By default, empty, to build the default target(s).

## 18.17. Infrastructure for Cargo-based packages

Cargo is the package manager for the Rust programming language. It allows the user to build programs or libraries written in Rust, but it also downloads and manages their dependencies, to ensure repeatable builds. Cargo packages are called "crates".

### 18.17.1. `cargo-package` tutorial

The `Config.in` file of Cargo-based package *foo* should contain:

```
01: config BR2_PACKAGE_FOO
02:     bool "foo"
03:     depends on BR2_PACKAGE_HOST_RUSTC_TARGET_ARCH_SUPPORTS
04:     select BR2_PACKAGE_HOST_RUSTC
05:     help
06:       This is a comment that explains what foo is.
07:
08:       http://foosoftware.org/foo/
```

And the `.mk` file for this package should contain:

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.0
08: FOO_SOURCE = foo-$(FOO_VERSION).tar.gz
09: FOO_SITE = http://www.foosoftware.org/download
10: FOO_LICENSE = GPL-3.0+
11: FOO_LICENSE_FILES = COPYING
12:
13: $(eval $(cargo-package))
```

The Makefile starts with the definition of the standard variables for package declaration (lines 7 to 11).

As seen in line 13, it is based on the `cargo-package` infrastructure. Cargo will be invoked automatically by this infrastructure to build and install the package.

It is still possible to define custom build commands or install commands (i.e. with FOO_BUILD_CMDS and FOO_INSTALL_TARGET_CMDS). Those will then replace the commands from the cargo infrastructure.

### 18.17.2. `cargo-package` reference

The main macros for the Cargo package infrastructure are `cargo-package` for target packages and `host-cargo-package` for host packages.

Just like the generic infrastructure, the Cargo infrastructure works by defining a number of variables before calling the `cargo-package` or `host-cargo-package` macros.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the Cargo infrastructure.

A few additional variables, specific to the Cargo infrastructure, can also be defined. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them.

- `FOO_SUBDIR` may contain the name of a subdirectory inside the package that contains the Cargo.toml file. This is useful, if for example, it is not at the root of the tree extracted by the tarball. If `HOST_FOO_SUBDIR` is not specified, it defaults to `FOO_SUBDIR`.
- `FOO_CARGO_ENV` can be used to pass additional variables in the environment of `cargo` invocations. It used at both build and installation time
- `FOO_CARGO_BUILD_OPTS` can be used to pass additional options to `cargo` at build time.
- `FOO_CARGO_INSTALL_OPTS` can be used to pass additional options to `cargo` at install time.

A crate can depend on other libraries from crates.io or git repositories, listed in its `Cargo.toml` file. Buildroot automatically takes care of downloading such dependencies as part of the download step of packages that use the `cargo-package` infrastructure. Such dependencies are then kept together with the package source code in the tarball cached in Buildroot’s `DL_DIR`, and therefore the hash of the package’s tarball doesn’t only cover the source of the package itself, but also covers the sources of the dependencies. Thus, a change injected into one of the dependencies will also be discovered by the hash check. In addition, this mechanism allows the build to be performed completely offline since cargo will not do any downloads during the build. This mechanism is called vendoring the dependencies.

