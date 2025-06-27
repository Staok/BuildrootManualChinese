### 18.19.2. `qmake-package` reference

The main macro of the QMake package infrastructure is `qmake-package`. It is similar to the `generic-package` macro.

Just like the generic infrastructure, the QMake infrastructure works by defining a number of variables before calling the `qmake-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the QMake infrastructure.

A few additional variables, specific to the QMake infrastructure, can also be defined.

- `LIBFOO_CONF_ENV`, to specify additional environment variables to pass to the `qmake` script for the configuration step. By default, empty.
- `LIBFOO_CONF_OPTS`, to specify additional options to pass to the `qmake` script for the configuration step. By default, empty.
- `LIBFOO_MAKE_ENV`, to specify additional environment variables to the `make` command during the build and install steps. By default, empty.
- `LIBFOO_MAKE_OPTS`, to specify additional targets to pass to the `make` command during the build step. By default, empty.
- `LIBFOO_INSTALL_STAGING_OPTS`, to specify additional targets to pass to the `make` command during the staging installation step. By default, `install`.
- `LIBFOO_INSTALL_TARGET_OPTS`, to specify additional targets to pass to the `make` command during the target installation step. By default, `install`.
- `LIBFOO_SYNC_QT_HEADERS`, to run syncqt.pl before qmake. Some packages need this to have a properly populated include directory before running the build.

## 18.20. Infrastructure for packages building kernel modules

Buildroot offers a helper infrastructure to make it easy to write packages that build and install Linux kernel modules. Some packages only contain a kernel module, other packages contain programs and libraries in addition to kernel modules. Buildroot’s helper infrastructure supports either case.

### 18.20.1. `kernel-module` tutorial

Let’s start with an example on how to prepare a simple package that only builds a kernel module, and no other component:

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.2.3
08: FOO_SOURCE = foo-$(FOO_VERSION).tar.xz
09: FOO_SITE = http://www.foosoftware.org/download
10: FOO_LICENSE = GPL-2.0
11: FOO_LICENSE_FILES = COPYING
12:
13: $(eval $(kernel-module))
14: $(eval $(generic-package))
```

Lines 7-11 define the usual meta-data to specify the version, archive name, remote URI where to find the package source, licensing information.

On line 13, we invoke the `kernel-module` helper infrastructure, that generates all the appropriate Makefile rules and variables to build that kernel module.

Finally, on line 14, we invoke the [`generic-package` infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-tutorial).

The dependency on `linux` is automatically added, so it is not needed to specify it in `FOO_DEPENDENCIES`.

What you may have noticed is that, unlike other package infrastructures, we explicitly invoke a second infrastructure. This allows a package to build a kernel module, but also, if needed, use any one of other package infrastructures to build normal userland components (libraries, executables…). Using the `kernel-module` infrastructure on its own is not sufficient; another package infrastructure ***\*must\**** be used.

Let’s look at a more complex example:

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: FOO_VERSION = 1.2.3
08: FOO_SOURCE = foo-$(FOO_VERSION).tar.xz
09: FOO_SITE = http://www.foosoftware.org/download
10: FOO_LICENSE = GPL-2.0
11: FOO_LICENSE_FILES = COPYING
12:
13: FOO_MODULE_SUBDIRS = driver/base
14: FOO_MODULE_MAKE_OPTS = KVERSION=$(LINUX_VERSION_PROBED)
15:
16: ifeq ($(BR2_PACKAGE_LIBBAR),y)
17: FOO_DEPENDENCIES += libbar
18: FOO_CONF_OPTS += --enable-bar
19: FOO_MODULE_SUBDIRS += driver/bar
20: else
21: FOO_CONF_OPTS += --disable-bar
22: endif
23:
24: $(eval $(kernel-module))
26: $(eval $(autotools-package))
```

Here, we see that we have an autotools-based package, that also builds the kernel module located in sub-directory `driver/base` and, if libbar is enabled, the kernel module located in sub-directory `driver/bar`, and defines the variable `KVERSION` to be passed to the Linux buildsystem when building the module(s).

### 18.20.2. `kernel-module` reference

The main macro for the kernel module infrastructure is `kernel-module`. Unlike other package infrastructures, it is not stand-alone, and requires any of the other `*-package` macros be called after it.

The `kernel-module` macro defines post-build and post-target-install hooks to build the kernel modules. If the package’s `.mk` needs access to the built kernel modules, it should do so in a post-build hook, ***\*registered after\**** the call to `kernel-module`. Similarly, if the package’s `.mk` needs access to the kernel module after it has been installed, it should do so in a post-install hook, ***\*registered after\**** the call to `kernel-module`. Here’s an example:

```
$(eval $(kernel-module))

define FOO_DO_STUFF_WITH_KERNEL_MODULE
    # Do something with it...
endef
FOO_POST_BUILD_HOOKS += FOO_DO_STUFF_WITH_KERNEL_MODULE

$(eval $(generic-package))
```

Finally, unlike the other package infrastructures, there is no `host-kernel-module` variant to build a host kernel module.

The following additional variables can optionally be defined to further configure the build of the kernel module:

- `FOO_MODULE_SUBDIRS` may be set to one or more sub-directories (relative to the package source top-directory) where the kernel module sources are. If empty or not set, the sources for the kernel module(s) are considered to be located at the top of the package source tree.
- `FOO_MODULE_MAKE_OPTS` may be set to contain extra variable definitions to pass to the Linux buildsystem.

You may also reference (but you may ***\*not\**** set!) those variables:

- `LINUX_DIR` contains the path to where the Linux kernel has been extracted and built.
- `LINUX_VERSION` contains the version string as configured by the user.
- `LINUX_VERSION_PROBED` contains the real version string of the kernel, retrieved with running `make -C $(LINUX_DIR) kernelrelease`
- `KERNEL_ARCH` contains the name of the current architecture, like `arm`, `mips`…

## 18.21. Infrastructure for asciidoc documents

The Buildroot manual, which you are currently reading, is entirely written using the [AsciiDoc](http://asciidoc.org/) mark-up syntax. The manual is then rendered to many formats:

- html
- split-html
- pdf
- epub
- text

Although Buildroot only contains one document written in AsciiDoc, there is, as for packages, an infrastructure for rendering documents using the AsciiDoc syntax.

Also as for packages, the AsciiDoc infrastructure is available from a [br2-external tree](https://buildroot.org/downloads/manual/manual.html#outside-br-custom). This allows documentation for a br2-external tree to match the Buildroot documentation, as it will be rendered to the same formats and use the same layout and theme.

### 18.21.1. `asciidoc-document` tutorial

Whereas package infrastructures are suffixed with `-package`, the document infrastructures are suffixed with `-document`. So, the AsciiDoc infrastructure is named `asciidoc-document`.

Here is an example to render a simple AsciiDoc document.

```
01: ################################################################################
02: #
03: # foo-document
04: #
05: ################################################################################
06:
07: FOO_SOURCES = $(sort $(wildcard $(FOO_DOCDIR)/*))
08: $(eval $(call asciidoc-document))
```

On line 7, the Makefile declares what the sources of the document are. Currently, it is expected that the document’s sources are only local; Buildroot will not attempt to download anything to render a document. Thus, you must indicate where the sources are. Usually, the string above is sufficient for a document with no sub-directory structure.

On line 8, we call the `asciidoc-document` function, which generates all the Makefile code necessary to render the document.

