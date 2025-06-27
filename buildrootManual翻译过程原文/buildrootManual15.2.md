### 18.21.2. `asciidoc-document` reference

The list of variables that can be set in a `.mk` file to give metadata information is (assuming the document name is `foo`) :

- `FOO_SOURCES`, mandatory, defines the source files for the document.
- `FOO_RESOURCES`, optional, may contain a space-separated list of paths to one or more directories containing so-called resources (like CSS or images). By default, empty.
- `FOO_DEPENDENCIES`, optional, the list of packages (most probably, host-packages) that must be built before building this document.
- `FOO_TOC_DEPTH`, `FOO_TOC_DEPTH_<FMT>`, optionals, the depth of the table of content for this document, which can be overridden for the specified format `<FMT>` (see the list of rendered formats, above, but in uppercase, and with dash replaced by underscore; see example, below). By default: `1`.

There are also additional hooks (see [Section 18.23, “Hooks available in the various build steps”](https://buildroot.org/downloads/manual/manual.html#hooks) for general information on hooks), that a document may set to define extra actions to be done at various steps:

- `FOO_POST_RSYNC_HOOKS` to run additional commands after the sources have been copied by Buildroot. This can for example be used to generate part of the manual with information extracted from the tree. As an example, Buildroot uses this hook to generate the tables in the appendices.
- `FOO_CHECK_DEPENDENCIES_HOOKS` to run additional tests on required components to generate the document. In AsciiDoc, it is possible to call filters, that is, programs that will parse an AsciiDoc block and render it appropriately (e.g. [ditaa](http://ditaa.sourceforge.net/) or [aafigure](https://pythonhosted.org/aafigure/)).
- `FOO_CHECK_DEPENDENCIES_<FMT>_HOOKS`, to run additional tests for the specified format `<FMT>` (see the list of rendered formats, above).

Buildroot sets the following variable that can be used in the definitions above:

- `$(FOO_DOCDIR)`, similar to `$(FOO_PKGDIR)`, contains the path to the directory containing `foo.mk`. It can be used to refer to the document sources, and can be used in the hooks, especially the post-rsync hook if parts of the documentation needs to be generated.
- `$(@D)`, as for traditional packages, contains the path to the directory where the document will be copied and built.

Here is a complete example that uses all variables and all hooks:

```
01: ################################################################################
02: #
03: # foo-document
04: #
05: ################################################################################
06:
07: FOO_SOURCES = $(sort $(wildcard $(FOO_DOCDIR)/*))
08: FOO_RESOURCES = $(sort $(wildcard $(FOO_DOCDIR)/resources))
09:
10: FOO_TOC_DEPTH = 2
11: FOO_TOC_DEPTH_HTML = 1
12: FOO_TOC_DEPTH_SPLIT_HTML = 3
13:
14: define FOO_GEN_EXTRA_DOC
15:     /path/to/generate-script --outdir=$(@D)
16: endef
17: FOO_POST_RSYNC_HOOKS += FOO_GEN_EXTRA_DOC
18:
19: define FOO_CHECK_MY_PROG
20:     if ! which my-prog >/dev/null 2>&1; then \
21:         echo "You need my-prog to generate the foo document"; \
22:         exit 1; \
23:     fi
24: endef
25: FOO_CHECK_DEPENDENCIES_HOOKS += FOO_CHECK_MY_PROG
26:
27: define FOO_CHECK_MY_OTHER_PROG
28:     if ! which my-other-prog >/dev/null 2>&1; then \
29:         echo "You need my-other-prog to generate the foo document as PDF"; \
30:         exit 1; \
31:     fi
32: endef
33: FOO_CHECK_DEPENDENCIES_PDF_HOOKS += FOO_CHECK_MY_OTHER_PROG
34:
35: $(eval $(call asciidoc-document))
```

## 18.22. Infrastructure specific to the Linux kernel package

The Linux kernel package can use some specific infrastructures based on package hooks for building Linux kernel tools or/and building Linux kernel extensions.

### 18.22.1. linux-kernel-tools

Buildroot offers a helper infrastructure to build some userspace tools for the target available within the Linux kernel sources. Since their source code is part of the kernel source code, a special package, `linux-tools`, exists and re-uses the sources of the Linux kernel that runs on the target.

Let’s look at an example of a Linux tool. For a new Linux tool named `foo`, create a new menu entry in the existing `package/linux-tools/Config.in`. This file will contain the option descriptions related to each kernel tool that will be used and displayed in the configuration tool. It would basically look like:

```
01: config BR2_PACKAGE_LINUX_TOOLS_FOO
02:     bool "foo"
03:     select BR2_PACKAGE_LINUX_TOOLS
04:     help
05:       This is a comment that explains what foo kernel tool is.
06:
07:       http://foosoftware.org/foo/
```

The name of the option starts with the prefix `BR2_PACKAGE_LINUX_TOOLS_`, followed by the uppercase name of the tool (like is done for packages).

**Note.** Unlike other packages, the `linux-tools` package options appear in the `linux` kernel menu, under the `Linux Kernel Tools` sub-menu, not under the `Target packages` main menu.

Then for each linux tool, add a new `.mk.in` file named `package/linux-tools/linux-tool-foo.mk.in`. It would basically look like:

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: LINUX_TOOLS += foo
08:
09: FOO_DEPENDENCIES = libbbb
10:
11: define FOO_BUILD_CMDS
12:     $(TARGET_MAKE_ENV) $(MAKE) -C $(LINUX_DIR)/tools foo
13: endef
14:
15: define FOO_INSTALL_STAGING_CMDS
16:     $(TARGET_MAKE_ENV) $(MAKE) -C $(LINUX_DIR)/tools \
17:             DESTDIR=$(STAGING_DIR) \
18:             foo_install
19: endef
20:
21: define FOO_INSTALL_TARGET_CMDS
22:     $(TARGET_MAKE_ENV) $(MAKE) -C $(LINUX_DIR)/tools \
23:             DESTDIR=$(TARGET_DIR) \
24:             foo_install
25: endef
```

On line 7, we register the Linux tool `foo` to the list of available Linux tools.

On line 9, we specify the list of dependencies this tool relies on. These dependencies are added to the Linux package dependencies list only when the `foo` tool is selected.

The rest of the Makefile, lines 11-25 defines what should be done at the different steps of the Linux tool build process like for a [`generic package`](https://buildroot.org/downloads/manual/manual.html#generic-package-tutorial). They will actually be used only when the `foo` tool is selected. The only supported commands are `_BUILD_CMDS`, `_INSTALL_STAGING_CMDS` and `_INSTALL_TARGET_CMDS`.

**Note.** One ***\*must not\**** call `$(eval $(generic-package))` or any other package infrastructure! Linux tools are not packages by themselves, they are part of the `linux-tools` package.

### 18.22.2. linux-kernel-extensions

Some packages provide new features that require the Linux kernel tree to be modified. This can be in the form of patches to be applied on the kernel tree, or in the form of new files to be added to the tree. The Buildroot’s Linux kernel extensions infrastructure provides a simple solution to automatically do this, just after the kernel sources are extracted and before the kernel patches are applied. Examples of extensions packaged using this mechanism are the real-time extensions Xenomai and RTAI, as well as the set of out-of-tree LCD screens drivers `fbtft`.

Let’s look at an example on how to add a new Linux extension `foo`.

First, create the package `foo` that provides the extension: this package is a standard package; see the previous chapters on how to create such a package. This package is in charge of downloading the sources archive, checking the hash, defining the licence information and building user space tools if any.

Then create the *Linux extension* proper: create a new menu entry in the existing `linux/Config.ext.in`. This file contains the option descriptions related to each kernel extension that will be used and displayed in the configuration tool. It would basically look like:

```
01: config BR2_LINUX_KERNEL_EXT_FOO
02:     bool "foo"
03:     help
04:       This is a comment that explains what foo kernel extension is.
05:
06:       http://foosoftware.org/foo/
```

Then for each linux extension, add a new `.mk` file named `linux/linux-ext-foo.mk`. It should basically contain:

```
01: ################################################################################
02: #
03: # foo
04: #
05: ################################################################################
06:
07: LINUX_EXTENSIONS += foo
08:
09: define FOO_PREPARE_KERNEL
10:     $(FOO_DIR)/prepare-kernel-tree.sh --linux-dir=$(@D)
11: endef
```

On line 7, we add the Linux extension `foo` to the list of available Linux extensions.

On line 9-11, we define what should be done by the extension to modify the Linux kernel tree; this is specific to the linux extension and can use the variables defined by the `foo` package, like: `$(FOO_DIR)` or `$(FOO_VERSION)`… as well as all the Linux variables, like: `$(LINUX_VERSION)` or `$(LINUX_VERSION_PROBED)`, `$(KERNEL_ARCH)`… See the [definition of those kernel variables](https://buildroot.org/downloads/manual/manual.html#kernel-variables).

