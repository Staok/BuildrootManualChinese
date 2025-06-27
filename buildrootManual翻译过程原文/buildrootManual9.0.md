## Chapter 18. Adding new packages to Buildroot

This section covers how new packages (userspace libraries or applications) can be integrated into Buildroot. It also shows how existing packages are integrated, which is needed for fixing issues or tuning their configuration.

When you add a new package, be sure to test it in various conditions (see [Section 18.25.3, “How to test your package”](https://buildroot.org/downloads/manual/manual.html#testing-package)) and also check it for coding style (see [Section 18.25.2, “How to check the coding style”](https://buildroot.org/downloads/manual/manual.html#check-package)).

## 18.1. Package directory

First of all, create a directory under the `package` directory for your software, for example `libfoo`.

Some packages have been grouped by topic in a sub-directory: `x11r7`, `qt5` and `gstreamer`. If your package fits in one of these categories, then create your package directory in these. New subdirectories are discouraged, however.

## 18.2. Config files

For the package to be displayed in the configuration tool, you need to create a Config file in your package directory. There are two types: `Config.in` and `Config.in.host`.

### 18.2.1. `Config.in` file

For packages used on the target, create a file named `Config.in`. This file will contain the option descriptions related to our `libfoo` software that will be used and displayed in the configuration tool. It should basically contain:

```
config BR2_PACKAGE_LIBFOO
        bool "libfoo"
        help
          This is a comment that explains what libfoo is. The help text
          should be wrapped.

          http://foosoftware.org/libfoo/
```

The `bool` line, `help` line and other metadata information about the configuration option must be indented with one tab. The help text itself should be indented with one tab and two spaces, lines should be wrapped to fit 72 columns, where tab counts for 8, so 62 characters in the text itself. The help text must mention the upstream URL of the project after an empty line.

As a convention specific to Buildroot, the ordering of the attributes is as follows:

1. The type of option: `bool`, `string`… with the prompt
2. If needed, the `default` value(s)
3. Any dependencies on the target in `depends on` form
4. Any dependencies on the toolchain in `depends on` form
5. Any dependencies on other packages in `depends on` form
6. Any dependency of the `select` form
7. The help keyword and help text.

You can add other sub-options into a `if BR2_PACKAGE_LIBFOO…endif` statement to configure particular things in your software. You can look at examples in other packages. The syntax of the `Config.in` file is the same as the one for the kernel Kconfig file. The documentation for this syntax is available at http://kernel.org/doc/Documentation/kbuild/kconfig-language.txt

Finally you have to add your new `libfoo/Config.in` to `package/Config.in` (or in a category subdirectory if you decided to put your package in one of the existing categories). The files included there are *sorted alphabetically* per category and are *NOT* supposed to contain anything but the *bare* name of the package.

```
source "package/libfoo/Config.in"
```

### 18.2.2. `Config.in.host` file

Some packages also need to be built for the host system. There are two options here:

- The host package is only required to satisfy build-time dependencies of one or more target packages. In this case, add `host-foo` to the target package’s `BAR_DEPENDENCIES` variable. No `Config.in.host` file should be created.

- The host package should be explicitly selectable by the user from the configuration menu. In this case, create a `Config.in.host` file for that host package:

  ```
  config BR2_PACKAGE_HOST_FOO
          bool "host foo"
          help
            This is a comment that explains what foo for the host is.
  
            http://foosoftware.org/foo/
  ```

  The same coding style and options as for the `Config.in` file are valid.

  Finally you have to add your new `libfoo/Config.in.host` to `package/Config.in.host`. The files included there are *sorted alphabetically* and are *NOT* supposed to contain anything but the *bare* name of the package.

  ```
  source "package/foo/Config.in.host"
  ```

  The host package will then be available from the `Host utilities` menu.

### 18.2.3. Choosing `depends on` or `select`

The `Config.in` file of your package must also ensure that dependencies are enabled. Typically, Buildroot uses the following rules:

- Use a `select` type of dependency for dependencies on libraries. These dependencies are generally not obvious and it therefore make sense to have the kconfig system ensure that the dependencies are selected. For example, the *libgtk2* package uses `select BR2_PACKAGE_LIBGLIB2` to make sure this library is also enabled. The `select` keyword expresses the dependency with a backward semantic.
- Use a `depends on` type of dependency when the user really needs to be aware of the dependency. Typically, Buildroot uses this type of dependency for dependencies on target architecture, MMU support and toolchain options (see [Section 18.2.4, “Dependencies on target and toolchain options”](https://buildroot.org/downloads/manual/manual.html#dependencies-target-toolchain-options)), or for dependencies on "big" things, such as the X.org system. The `depends on` keyword expresses the dependency with a forward semantic.

**Note.** The current problem with the *kconfig* language is that these two dependency semantics are not internally linked. Therefore, it may be possible to select a package, whom one of its dependencies/requirement is not met.

An example illustrates both the usage of `select` and `depends on`.

```
config BR2_PACKAGE_RRDTOOL
        bool "rrdtool"
        depends on BR2_USE_WCHAR
        select BR2_PACKAGE_FREETYPE
        select BR2_PACKAGE_LIBART
        select BR2_PACKAGE_LIBPNG
        select BR2_PACKAGE_ZLIB
        help
          RRDtool is the OpenSource industry standard, high performance
          data logging and graphing system for time series data.

          http://oss.oetiker.ch/rrdtool/

comment "rrdtool needs a toolchain w/ wchar"
        depends on !BR2_USE_WCHAR
```

Note that these two dependency types are only transitive with the dependencies of the same kind.

This means, in the following example:

```
config BR2_PACKAGE_A
        bool "Package A"

config BR2_PACKAGE_B
        bool "Package B"
        depends on BR2_PACKAGE_A

config BR2_PACKAGE_C
        bool "Package C"
        depends on BR2_PACKAGE_B

config BR2_PACKAGE_D
        bool "Package D"
        select BR2_PACKAGE_B

config BR2_PACKAGE_E
        bool "Package E"
        select BR2_PACKAGE_D
```

- Selecting `Package C` will be visible if `Package B` has been selected, which in turn is only visible if `Package A` has been selected.
- Selecting `Package E` will select `Package D`, which will select `Package B`, it will not check for the dependencies of `Package B`, so it will not select `Package A`.
- Since `Package B` is selected but `Package A` is not, this violates the dependency of `Package B` on `Package A`. Therefore, in such a situation, the transitive dependency has to be added explicitly:

```
config BR2_PACKAGE_D
        bool "Package D"
        depends on BR2_PACKAGE_A
        select BR2_PACKAGE_B

config BR2_PACKAGE_E
        bool "Package E"
        depends on BR2_PACKAGE_A
        select BR2_PACKAGE_D
```

Overall, for package library dependencies, `select` should be preferred.

Note that such dependencies will ensure that the dependency option is also enabled, but not necessarily built before your package. To do so, the dependency also needs to be expressed in the `.mk` file of the package.

Further formatting details: see [the coding style](https://buildroot.org/downloads/manual/manual.html#writing-rules-config-in).

