### 18.9.2. `python-package` reference

As a policy, packages that merely provide Python modules should all be named `python-<something>` in Buildroot. Other packages that use the Python build system, but are not Python modules, can freely choose their name (existing examples in Buildroot are `scons` and `supervisor`).

The main macro of the Python package infrastructure is `python-package`. It is similar to the `generic-package` macro. It is also possible to create Python host packages with the `host-python-package` macro.

Just like the generic infrastructure, the Python infrastructure works by defining a number of variables before calling the `python-package` or `host-python-package` macros.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the Python infrastructure.

Note that:

- It is not necessary to add `python` or `host-python` in the `PYTHON_FOO_DEPENDENCIES` variable of a package, since these basic dependencies are automatically added as needed by the Python package infrastructure.
- Similarly, it is not needed to add `host-python-setuptools` to `PYTHON_FOO_DEPENDENCIES` for setuptools-based packages, since it’s automatically added by the Python infrastructure as needed.

One variable specific to the Python infrastructure is mandatory:

- `PYTHON_FOO_SETUP_TYPE`, to define which Python build system is used by the package. The seven supported values are `flit`, `hatch`, `pep517`, `poetry`, `setuptools`, `setuptools-rust` and `maturin`. If you don’t know which one is used in your package, look at the `setup.py` or `pyproject.toml` file in your package source code, and see whether it imports things from the `flit` module or the `setuptools` module. If the package is using a `pyproject.toml` file without any build-system requires and with a local in-tree backend-path one should use `pep517`.

A few additional variables, specific to the Python infrastructure, can optionally be defined, depending on the package’s needs. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them, or none.

- `PYTHON_FOO_SUBDIR` may contain the name of a subdirectory inside the package that contains the main `setup.py` or `pyproject.toml` file. This is useful, if for example, the main `setup.py` or `pyproject.toml` file is not at the root of the tree extracted by the tarball. If `HOST_PYTHON_FOO_SUBDIR` is not specified, it defaults to `PYTHON_FOO_SUBDIR`.
- `PYTHON_FOO_ENV`, to specify additional environment variables to pass to the Python `setup.py` script (for setuptools packages) or the `support/scripts/pyinstaller.py` script (for flit/pep517 packages) for both the build and install steps. Note that the infrastructure is automatically passing several standard variables, defined in `PKG_PYTHON_SETUPTOOLS_ENV` (for setuptools target packages), `HOST_PKG_PYTHON_SETUPTOOLS_ENV` (for setuptools host packages), `PKG_PYTHON_PEP517_ENV` (for flit/pep517 target packages) and `HOST_PKG_PYTHON_PEP517_ENV` (for flit/pep517 host packages).
- `PYTHON_FOO_BUILD_OPTS`, to specify additional options to pass to the Python `python -m build` call. To pass additional options to the build backend use the `--config-setting=` (`-C`) flag of the `build` module.
- `PYTHON_FOO_INSTALL_TARGET_OPTS`, `PYTHON_FOO_INSTALL_STAGING_OPTS`, `HOST_PYTHON_FOO_INSTALL_OPTS` to specify additional options to pass to the Python `setup.py` script (for setuptools packages) or `support/scripts/pyinstaller.py` (for flit/pep517 packages) during the target installation step, the staging installation step or the host installation, respectively.

With the Python infrastructure, all the steps required to build and install the packages are already defined, and they generally work well for most Python-based packages. However, when required, it is still possible to customize what is done in any particular step:

- By adding a post-operation hook (after extract, patch, configure, build or install). See [Section 18.23, “Hooks available in the various build steps”](https://buildroot.org/downloads/manual/manual.html#hooks) for details.
- By overriding one of the steps. For example, even if the Python infrastructure is used, if the package `.mk` file defines its own `PYTHON_FOO_BUILD_CMDS` variable, it will be used instead of the default Python one. However, using this method should be restricted to very specific cases. Do not use it in the general case.

### 18.9.3. Generating a `python-package` from a PyPI repository

If the Python package for which you would like to create a Buildroot package is available on PyPI, you may want to use the `scanpypi` tool located in `utils/` to automate the process.

You can find the list of existing PyPI packages [here](https://pypi.python.org/).

`scanpypi` requires Python’s `setuptools` package to be installed on your host.

When at the root of your buildroot directory just do :

```
utils/scanpypi foo bar -o package
```

This will generate packages `python-foo` and `python-bar` in the package folder if they exist on [https://pypi.python.org](https://pypi.python.org/).

Find the `external python modules` menu and insert your package inside. Keep in mind that the items inside a menu should be in alphabetical order.

Please keep in mind that you’ll most likely have to manually check the package for any mistakes as there are things that cannot be guessed by the generator (e.g. dependencies on any of the python core modules such as BR2_PACKAGE_PYTHON_ZLIB). Also, please take note that the license and license files are guessed and must be checked. You also need to manually add the package to the `package/Config.in` file.

If your Buildroot package is not in the official Buildroot tree but in a br2-external tree, use the -o flag as follows:

```
utils/scanpypi foo bar -o other_package_dir
```

This will generate packages `python-foo` and `python-bar` in the `other_package_directory` instead of `package`.

Option `-h` will list the available options:

```
utils/scanpypi -h
```

### 18.9.4. `python-package` CFFI backend

C Foreign Function Interface for Python (CFFI) provides a convenient and reliable way to call compiled C code from Python using interface declarations written in C. Python packages relying on this backend can be identified by the appearance of a `cffi` dependency in the `install_requires` field of their `setup.py` file.

Such a package should:

- add `python-cffi` as a runtime dependency in order to install the compiled C library wrapper on the target. This is achieved by adding `select BR2_PACKAGE_PYTHON_CFFI` to the package `Config.in`.

```
config BR2_PACKAGE_PYTHON_FOO
        bool "python-foo"
        select BR2_PACKAGE_PYTHON_CFFI # runtime
```

- add `host-python-cffi` as a build-time dependency in order to cross-compile the C wrapper. This is achieved by adding `host-python-cffi` to the `PYTHON_FOO_DEPENDENCIES` variable.

```
################################################################################
#
# python-foo
#
################################################################################

...

PYTHON_FOO_DEPENDENCIES = host-python-cffi

$(eval $(python-package))
```

