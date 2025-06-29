## 18.8. Infrastructure for CMake-based packages

### 18.8.1. `cmake-package` tutorial

First, let’s see how to write a `.mk` file for a CMake-based package, with an example :

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
10: LIBFOO_INSTALL_STAGING = YES
11: LIBFOO_INSTALL_TARGET = NO
12: LIBFOO_CONF_OPTS = -DBUILD_DEMOS=ON
13: LIBFOO_DEPENDENCIES = libglib2 host-pkgconf
14:
15: $(eval $(cmake-package))
```

On line 7, we declare the version of the package.

On line 8 and 9, we declare the name of the tarball (xz-ed tarball recommended) and the location of the tarball on the Web. Buildroot will automatically download the tarball from this location.

On line 10, we tell Buildroot to install the package to the staging directory. The staging directory, located in `output/staging/` is the directory where all the packages are installed, including their development files, etc. By default, packages are not installed to the staging directory, since usually, only libraries need to be installed in the staging directory: their development files are needed to compile other libraries or applications depending on them. Also by default, when staging installation is enabled, packages are installed in this location using the `make install` command.

On line 11, we tell Buildroot to not install the package to the target directory. This directory contains what will become the root filesystem running on the target. For purely static libraries, it is not necessary to install them in the target directory because they will not be used at runtime. By default, target installation is enabled; setting this variable to NO is almost never needed. Also by default, packages are installed in this location using the `make install` command.

On line 12, we tell Buildroot to pass custom options to CMake when it is configuring the package.

On line 13, we declare our dependencies, so that they are built before the build process of our package starts.

Finally, on line line 15, we invoke the `cmake-package` macro that generates all the Makefile rules that actually allows the package to be built.

### 18.8.2. `cmake-package` reference

The main macro of the CMake package infrastructure is `cmake-package`. It is similar to the `generic-package` macro. The ability to have target and host packages is also available, with the `host-cmake-package` macro.

Just like the generic infrastructure, the CMake infrastructure works by defining a number of variables before calling the `cmake-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the CMake infrastructure.

A few additional variables, specific to the CMake infrastructure, can also be defined. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them.

- `LIBFOO_SUBDIR` may contain the name of a subdirectory inside the package that contains the main CMakeLists.txt file. This is useful, if for example, the main CMakeLists.txt file is not at the root of the tree extracted by the tarball. If `HOST_LIBFOO_SUBDIR` is not specified, it defaults to `LIBFOO_SUBDIR`.
- `LIBFOO_CMAKE_BACKEND` specifies the cmake backend to use, one of `make` (to use the GNU Makefiles generator, the default) or `ninja` (to use the Ninja generator).
- `LIBFOO_CONF_ENV`, to specify additional environment variables to pass to CMake. By default, empty.
- `LIBFOO_CONF_OPTS`, to specify additional configure options to pass to CMake. By default, empty. A number of common CMake options are set by the `cmake-package` infrastructure; so it is normally not necessary to set them in the package’s `*.mk` file unless you want to override them:
  - `CMAKE_BUILD_TYPE` is driven by `BR2_ENABLE_RUNTIME_DEBUG`;
  - `CMAKE_INSTALL_PREFIX`;
  - `BUILD_SHARED_LIBS` is driven by `BR2_STATIC_LIBS`;
  - `BUILD_DOC`, `BUILD_DOCS` are disabled;
  - `BUILD_EXAMPLE`, `BUILD_EXAMPLES` are disabled;
  - `BUILD_TEST`, `BUILD_TESTS`, `BUILD_TESTING` are disabled.
- `LIBFOO_BUILD_ENV` and `LIBFOO_BUILD_OPTS` to specify additional environment variables, or command line options, to pass to the backend at build time.
- `LIBFOO_SUPPORTS_IN_SOURCE_BUILD = NO` should be set when the package cannot be built inside the source tree but needs a separate build directory.
- `LIBFOO_MAKE`, to specify an alternate `make` command. This is typically useful when parallel make is enabled in the configuration (using `BR2_JLEVEL`) but that this feature should be disabled for the given package, for one reason or another. By default, set to `$(MAKE)`. If parallel building is not supported by the package, then it should be set to `LIBFOO_MAKE=$(MAKE1)`.
- `LIBFOO_MAKE_ENV`, to specify additional environment variables to pass to make in the build step. These are passed before the `make` command. By default, empty.
- `LIBFOO_MAKE_OPTS`, to specify additional variables to pass to make in the build step. These are passed after the `make` command. By default, empty.
- `LIBFOO_INSTALL_OPTS` contains the make options used to install the package to the host directory. By default, the value is `install`, which is correct for most CMake packages. It is still possible to override it.
- `LIBFOO_INSTALL_STAGING_OPTS` contains the make options used to install the package to the staging directory. By default, the value is `DESTDIR=$(STAGING_DIR) install/fast`, which is correct for most CMake packages. It is still possible to override it.
- `LIBFOO_INSTALL_TARGET_OPTS` contains the make options used to install the package to the target directory. By default, the value is `DESTDIR=$(TARGET_DIR) install/fast`. The default value is correct for most CMake packages, but it is still possible to override it if needed.

With the CMake infrastructure, all the steps required to build and install the packages are already defined, and they generally work well for most CMake-based packages. However, when required, it is still possible to customize what is done in any particular step:

- By adding a post-operation hook (after extract, patch, configure, build or install). See [Section 18.23, “Hooks available in the various build steps”](https://buildroot.org/downloads/manual/manual.html#hooks) for details.
- By overriding one of the steps. For example, even if the CMake infrastructure is used, if the package `.mk` file defines its own `LIBFOO_CONFIGURE_CMDS` variable, it will be used instead of the default CMake one. However, using this method should be restricted to very specific cases. Do not use it in the general case.

## 18.9. Infrastructure for Python packages

This infrastructure applies to Python packages that use the standard Python setuptools, pep517, flit or maturin mechanisms as their build system, generally recognizable by the usage of a `setup.py` script or `pyproject.toml` file.

### 18.9.1. `python-package` tutorial

First, let’s see how to write a `.mk` file for a Python package, with an example :

```
01: ################################################################################
02: #
03: # python-foo
04: #
05: ################################################################################
06:
07: PYTHON_FOO_VERSION = 1.0
08: PYTHON_FOO_SOURCE = python-foo-$(PYTHON_FOO_VERSION).tar.xz
09: PYTHON_FOO_SITE = http://www.foosoftware.org/download
10: PYTHON_FOO_LICENSE = BSD-3-Clause
11: PYTHON_FOO_LICENSE_FILES = LICENSE
12: PYTHON_FOO_ENV = SOME_VAR=1
13: PYTHON_FOO_DEPENDENCIES = libmad
14: PYTHON_FOO_SETUP_TYPE = setuptools
15:
16: $(eval $(python-package))
```

On line 7, we declare the version of the package.

On line 8 and 9, we declare the name of the tarball (xz-ed tarball recommended) and the location of the tarball on the Web. Buildroot will automatically download the tarball from this location.

On line 10 and 11, we give licensing details about the package (its license on line 10, and the file containing the license text on line 11).

On line 12, we tell Buildroot to pass custom options to the Python `setup.py` script when it is configuring the package.

On line 13, we declare our dependencies, so that they are built before the build process of our package starts.

On line 14, we declare the specific Python build system being used. In this case the `setuptools` Python build system is used. The seven supported ones are `flit`, `hatch`, `pep517`, `poetry`, `setuptools`, `setuptools-rust` and `maturin`.

Finally, on line 16, we invoke the `python-package` macro that generates all the Makefile rules that actually allow the package to be built.

