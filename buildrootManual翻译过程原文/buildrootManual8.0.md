# Part III. Developer guide

## Chapter 15. How Buildroot works

As mentioned above, Buildroot is basically a set of Makefiles that download, configure, and compile software with the correct options. It also includes patches for various software packages - mainly the ones involved in the cross-compilation toolchain (`gcc`, `binutils` and `uClibc`).

There is basically one Makefile per software package, and they are named with the `.mk` extension. Makefiles are split into many different parts.

- The `toolchain/` directory contains the Makefiles and associated files for all software related to the cross-compilation toolchain: `binutils`, `gcc`, `gdb`, `kernel-headers` and `uClibc`.
- The `arch/` directory contains the definitions for all the processor architectures that are supported by Buildroot.
- The `package/` directory contains the Makefiles and associated files for all user-space tools and libraries that Buildroot can compile and add to the target root filesystem. There is one sub-directory per package.
- The `linux/` directory contains the Makefiles and associated files for the Linux kernel.
- The `boot/` directory contains the Makefiles and associated files for the bootloaders supported by Buildroot.
- The `system/` directory contains support for system integration, e.g. the target filesystem skeleton and the selection of an init system.
- The `fs/` directory contains the Makefiles and associated files for software related to the generation of the target root filesystem image.

Each directory contains at least 2 files:

- `something.mk` is the Makefile that downloads, configures, compiles and installs the package `something`.
- `Config.in` is a part of the configuration tool description file. It describes the options related to the package.

The main Makefile performs the following steps (once the configuration is done):

- Create all the output directories: `staging`, `target`, `build`, etc. in the output directory (`output/` by default, another value can be specified using `O=`)
- Generate the toolchain target. When an internal toolchain is used, this means generating the cross-compilation toolchain. When an external toolchain is used, this means checking the features of the external toolchain and importing it into the Buildroot environment.
- Generate all the targets listed in the `TARGETS` variable. This variable is filled by all the individual components' Makefiles. Generating these targets will trigger the compilation of the userspace packages (libraries, programs), the kernel, the bootloader and the generation of the root filesystem images, depending on the configuration.

## Chapter 16. Coding style

Overall, these coding style rules are here to help you to add new files in Buildroot or refactor existing ones.

If you slightly modify some existing file, the important thing is to keep the consistency of the whole file, so you can:

- either follow the potentially deprecated coding style used in this file,
- or entirely rework it in order to make it comply with these rules.

## 16.1. `Config.in` file

`Config.in` files contain entries for almost anything configurable in Buildroot.

An entry has the following pattern:

```
config BR2_PACKAGE_LIBFOO
        bool "libfoo"
        depends on BR2_PACKAGE_LIBBAZ
        select BR2_PACKAGE_LIBBAR
        help
          This is a comment that explains what libfoo is. The help text
          should be wrapped.

          http://foosoftware.org/libfoo/
```

- The `bool`, `depends on`, `select` and `help` lines are indented with one tab.
- The help text itself should be indented with one tab and two spaces.
- The help text should be wrapped to fit 72 columns, where tab counts for 8, so 62 characters in the text itself.

The `Config.in` files are the input for the configuration tool used in Buildroot, which is the regular *Kconfig*. For further details about the *Kconfig* language, refer to http://kernel.org/doc/Documentation/kbuild/kconfig-language.txt.

## 16.2. The `.mk` file

- Header: The file starts with a header. It contains the module name, preferably in lowercase, enclosed between separators made of 80 hashes. A blank line is mandatory after the header:

  ```
  ################################################################################
  #
  # libfoo
  #
  ################################################################################
  ```

- Assignment: use `=` preceded and followed by one space:

  ```
  LIBFOO_VERSION = 1.0
  LIBFOO_CONF_OPTS += --without-python-support
  ```

  Do not align the `=` signs.

- Indentation: use tab only:

  ```
  define LIBFOO_REMOVE_DOC
          $(RM) -r $(TARGET_DIR)/usr/share/libfoo/doc \
                  $(TARGET_DIR)/usr/share/man/man3/libfoo*
  endef
  ```

  Note that commands inside a `define` block should always start with a tab, so *make* recognizes them as commands.

- Optional dependency:

  - Prefer multi-line syntax.

    YES:

    ```
    ifeq ($(BR2_PACKAGE_PYTHON3),y)
    LIBFOO_CONF_OPTS += --with-python-support
    LIBFOO_DEPENDENCIES += python3
    else
    LIBFOO_CONF_OPTS += --without-python-support
    endif
    ```

    NO:

    ```
    LIBFOO_CONF_OPTS += --with$(if $(BR2_PACKAGE_PYTHON3),,out)-python-support
    LIBFOO_DEPENDENCIES += $(if $(BR2_PACKAGE_PYTHON3),python3,)
    ```

  - Keep configure options and dependencies close together.

- Optional hooks: keep hook definition and assignment together in one if block.

  YES:

  ```
  ifneq ($(BR2_LIBFOO_INSTALL_DATA),y)
  define LIBFOO_REMOVE_DATA
          $(RM) -r $(TARGET_DIR)/usr/share/libfoo/data
  endef
  LIBFOO_POST_INSTALL_TARGET_HOOKS += LIBFOO_REMOVE_DATA
  endif
  ```

  NO:

  ```
  define LIBFOO_REMOVE_DATA
          $(RM) -r $(TARGET_DIR)/usr/share/libfoo/data
  endef
  
  ifneq ($(BR2_LIBFOO_INSTALL_DATA),y)
  LIBFOO_POST_INSTALL_TARGET_HOOKS += LIBFOO_REMOVE_DATA
  endif
  ```
