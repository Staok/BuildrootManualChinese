## 18.18. Infrastructure for Go packages

This infrastructure applies to Go packages that use the standard build system and use bundled dependencies.

### 18.18.1. `golang-package` tutorial

First, let’s see how to write a `.mk` file for a go package, with an example :

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.0
08: FOO_SITE = $(call github,bar,foo,$(FOO_VERSION))
09: FOO_LICENSE = BSD-3-Clause
10: FOO_LICENSE_FILES = LICENSE
11:
12: $(eval $(golang-package))
```

On line 7, we declare the version of the package.

On line 8, we declare the upstream location of the package, here fetched from Github, since a large number of Go packages are hosted on Github.

On line 9 and 10, we give licensing details about the package.

Finally, on line 12, we invoke the `golang-package` macro that generates all the Makefile rules that actually allow the package to be built.

### 18.18.2. `golang-package` reference

In their `Config.in` file, packages using the `golang-package` infrastructure should depend on `BR2_PACKAGE_HOST_GO_TARGET_ARCH_SUPPORTS` because Buildroot will automatically add a dependency on `host-go` to such packages. If you need CGO support in your package, you must add a dependency on `BR2_PACKAGE_HOST_GO_TARGET_CGO_LINKING_SUPPORTS`; for host packages, add a dependency on `BR2_PACKAGE_HOST_GO_HOST_CGO_LINKING_SUPPORTS`.

The main macro of the Go package infrastructure is `golang-package`. It is similar to the `generic-package` macro. The ability to build host packages is also available, with the `host-golang-package` macro. Host packages built by `host-golang-package` macro should depend on `BR2_PACKAGE_HOST_GO_HOST_ARCH_SUPPORTS`.

Just like the generic infrastructure, the Go infrastructure works by defining a number of variables before calling the `golang-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the Go infrastructure.

Note that it is not necessary to add `host-go` in the `FOO_DEPENDENCIES` variable of a package, since this basic dependency is automatically added as needed by the Go package infrastructure.

A few additional variables, specific to the Go infrastructure, can optionally be defined, depending on the package’s needs. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them, or none.

- The package must specify its Go module name in the `FOO_GOMOD` variable. If not specified, it defaults to `URL-domain/1st-part-of-URL/2nd-part-of-URL`, e.g `FOO_GOMOD` will take the value `github.com/bar/foo` for a package that specifies `FOO_SITE = $(call github,bar,foo,$(FOO_VERSION))`. The Go package infrastructure will automatically generate a minimal `go.mod` file in the package source tree if it doesn’t exist.
- `FOO_LDFLAGS`, `FOO_EXTLDFLAGS`, and `FOO_TAGS` can be used to pass respectively the go `LDFLAGS` (via the `-ldflags` command line flag), the external linker flags `EXTLDFLAGS` (via the `-extldflags` command line flag), or the `TAGS` to the `go` build command.
- `FOO_BUILD_TARGETS` can be used to pass the list of targets that should be built. If `FOO_BUILD_TARGETS` is not specified, it defaults to `.`. We then have two cases:
  - `FOO_BUILD_TARGETS` is `.`. In this case, we assume only one binary will be produced, and that by default we name it after the package name. If that is not appropriate, the name of the produced binary can be overridden using `FOO_BIN_NAME`.
  - `FOO_BUILD_TARGETS` is not `.`. In this case, we iterate over the values to build each target, and for each produced a binary that is the non-directory component of the target. For example if `FOO_BUILD_TARGETS = cmd/docker cmd/dockerd` the binaries produced are `docker` and `dockerd`.
- `FOO_INSTALL_BINS` can be used to pass the list of binaries that should be installed in `/usr/bin` on the target. If `FOO_INSTALL_BINS` is not specified, it defaults to the lower-case name of package.

With the Go infrastructure, all the steps required to build and install the packages are already defined, and they generally work well for most Go-based packages. However, when required, it is still possible to customize what is done in any particular step:

- By adding a post-operation hook (after extract, patch, configure, build or install). See [Section 18.23, “Hooks available in the various build steps”](https://buildroot.org/downloads/manual/manual.html#hooks) for details.
- By overriding one of the steps. For example, even if the Go infrastructure is used, if the package `.mk` file defines its own `FOO_BUILD_CMDS` variable, it will be used instead of the default Go one. However, using this method should be restricted to very specific cases. Do not use it in the general case.

A Go package can depend on other Go modules, listed in its `go.mod` file. Buildroot automatically takes care of downloading such dependencies as part of the download step of packages that use the `golang-package` infrastructure. Such dependencies are then kept together with the package source code in the tarball cached in Buildroot’s `DL_DIR`, and therefore the hash of the package’s tarball includes such dependencies.

This mechanism ensures that any change in the dependencies will be detected, and allows the build to be performed completely offline.

## 18.19. Infrastructure for QMake-based packages

### 18.19.1. `qmake-package` tutorial

First, let’s see how to write a `.mk` file for a QMake-based package, with an example :

```
01: ################################################################################
02: #
03: # libfoo
04: #
05: ################################################################################
06:
07: LIBFOO_VERSION = 1.0
08: LIBFOO_SOURCE = libfoo-$(LIBFOO_VERSION).tar.gz
09: LIBFOO_SITE = http://www.foosoftware.org/download
10: LIBFOO_CONF_OPTS = QT_CONFIG+=bar QT_CONFIG-=baz
11: LIBFOO_DEPENDENCIES = bar
12:
13: $(eval $(qmake-package))
```

On line 7, we declare the version of the package.

On line 8 and 9, we declare the name of the tarball (xz-ed tarball recommended) and the location of the tarball on the Web. Buildroot will automatically download the tarball from this location.

On line 10, we tell Buildroot what options to enable for libfoo.

On line 11, we tell Buildroot the dependencies of libfoo.

Finally, on line line 13, we invoke the `qmake-package` macro that generates all the Makefile rules that actually allows the package to be built.

