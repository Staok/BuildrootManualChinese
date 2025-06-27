## 18.23. Hooks available in the various build steps

The generic infrastructure (and as a result also the derived autotools and cmake infrastructures) allow packages to specify hooks. These define further actions to perform after existing steps. Most hooks aren’t really useful for generic packages, since the `.mk` file already has full control over the actions performed in each step of the package construction.

The following hook points are available:

- `LIBFOO_PRE_DOWNLOAD_HOOKS`
- `LIBFOO_POST_DOWNLOAD_HOOKS`
- `LIBFOO_PRE_EXTRACT_HOOKS`
- `LIBFOO_POST_EXTRACT_HOOKS`
- `LIBFOO_PRE_RSYNC_HOOKS`
- `LIBFOO_POST_RSYNC_HOOKS`
- `LIBFOO_PRE_PATCH_HOOKS`
- `LIBFOO_POST_PATCH_HOOKS`
- `LIBFOO_PRE_CONFIGURE_HOOKS`
- `LIBFOO_POST_CONFIGURE_HOOKS`
- `LIBFOO_PRE_BUILD_HOOKS`
- `LIBFOO_POST_BUILD_HOOKS`
- `LIBFOO_PRE_INSTALL_HOOKS` (for host packages only)
- `LIBFOO_POST_INSTALL_HOOKS` (for host packages only)
- `LIBFOO_PRE_INSTALL_STAGING_HOOKS` (for target packages only)
- `LIBFOO_POST_INSTALL_STAGING_HOOKS` (for target packages only)
- `LIBFOO_PRE_INSTALL_TARGET_HOOKS` (for target packages only)
- `LIBFOO_POST_INSTALL_TARGET_HOOKS` (for target packages only)
- `LIBFOO_PRE_INSTALL_IMAGES_HOOKS`
- `LIBFOO_POST_INSTALL_IMAGES_HOOKS`
- `LIBFOO_PRE_LEGAL_INFO_HOOKS`
- `LIBFOO_POST_LEGAL_INFO_HOOKS`
- `LIBFOO_TARGET_FINALIZE_HOOKS`

These variables are *lists* of variable names containing actions to be performed at this hook point. This allows several hooks to be registered at a given hook point. Here is an example:

```
define LIBFOO_POST_PATCH_FIXUP
        action1
        action2
endef

LIBFOO_POST_PATCH_HOOKS += LIBFOO_POST_PATCH_FIXUP
```

### 18.23.1. Using the `POST_RSYNC` hook

The `POST_RSYNC` hook is run only for packages that use a local source, either through the `local` site method or the `OVERRIDE_SRCDIR` mechanism. In this case, package sources are copied using `rsync` from the local location into the buildroot build directory. The `rsync` command does not copy all files from the source directory, though. Files belonging to a version control system, like the directories `.git`, `.hg`, etc. are not copied. For most packages this is sufficient, but a given package can perform additional actions using the `POST_RSYNC` hook.

In principle, the hook can contain any command you want. One specific use case, though, is the intentional copying of the version control directory using `rsync`. The `rsync` command you use in the hook can, among others, use the following variables:

- `$(SRCDIR)`: the path to the overridden source directory
- `$(@D)`: the path to the build directory

### 18.23.2. Target-finalize hook

Packages may also register hooks in `LIBFOO_TARGET_FINALIZE_HOOKS`. These hooks are run after all packages are built, but before the filesystem images are generated. They are seldom used, and your package probably do not need them.

## 18.24. Gettext integration and interaction with packages

Many packages that support internationalization use the gettext library. Dependencies for this library are fairly complicated and therefore, deserve some explanation.

The *glibc* C library integrates a full-blown implementation of *gettext*, supporting translation. Native Language Support is therefore built-in in *glibc*.

On the other hand, the *uClibc* and *musl* C libraries only provide a stub implementation of the gettext functionality, which allows to compile libraries and programs using gettext functions, but without providing the translation capabilities of a full-blown gettext implementation. With such C libraries, if real Native Language Support is necessary, it can be provided by the `libintl` library of the `gettext` package.

Due to this, and in order to make sure that Native Language Support is properly handled, packages in Buildroot that can use NLS support should:

1. Ensure NLS support is enabled when `BR2_SYSTEM_ENABLE_NLS=y`. This is done automatically for *autotools* packages and therefore should only be done for packages using other package infrastructures.
2. Add `$(TARGET_NLS_DEPENDENCIES)` to the package `<pkg>_DEPENDENCIES` variable. This addition should be done unconditionally: the value of this variable is automatically adjusted by the core infrastructure to contain the relevant list of packages. If NLS support is disabled, this variable is empty. If NLS support is enabled, this variable contains `host-gettext` so that tools needed to compile translation files are available on the host. In addition, if *uClibc* or *musl* are used, this variable also contains `gettext` in order to get the full-blown *gettext* implementation.
3. If needed, add `$(TARGET_NLS_LIBS)` to the linker flags, so that the package gets linked with `libintl`. This is generally not needed with *autotools* packages as they usually detect automatically that they should link with `libintl`. However, packages using other build systems, or problematic autotools-based packages may need this. `$(TARGET_NLS_LIBS)` should be added unconditionally to the linker flags, as the core automatically makes it empty or defined to `-lintl` depending on the configuration.

No changes should be made to the `Config.in` file to support NLS.

Finally, certain packages need some gettext utilities on the target, such as the `gettext` program itself, which allows to retrieve translated strings, from the command line. In such a case, the package should:

- use `select BR2_PACKAGE_GETTEXT` in their `Config.in` file, indicating in a comment above that it’s a runtime dependency only.
- not add any `gettext` dependency in the `DEPENDENCIES` variable of their `.mk` file.

## 18.25. Tips and tricks

### 18.25.1. Package name, config entry name and makefile variable relationship

In Buildroot, there is some relationship between:

- the *package name*, which is the package directory name (and the name of the `*.mk` file);
- the config entry name that is declared in the `Config.in` file;
- the makefile variable prefix.

It is mandatory to maintain consistency between these elements, using the following rules:

- the package directory and the `*.mk` name are the *package name* itself (e.g.: `package/foo-bar_boo/foo-bar_boo.mk`);
- the *make* target name is the *package name* itself (e.g.: `foo-bar_boo`);
- the config entry is the upper case *package name* with `.` and `-` characters substituted with `_`, prefixed with `BR2_PACKAGE_` (e.g.: `BR2_PACKAGE_FOO_BAR_BOO`);
- the `*.mk` file variable prefix is the upper case *package name* with `.` and `-` characters substituted with `_` (e.g.: `FOO_BAR_BOO_VERSION`).

### 18.25.2. How to check the coding style

Buildroot provides a script in `utils/check-package` that checks new or changed files for coding style. It is not a complete language validator, but it catches many common mistakes. It is meant to run in the actual files you created or modified, before creating the patch for submission.

This script can be used for packages, filesystem makefiles, Config.in files, etc. It does not check the files defining the package infrastructures and some other files containing similar common code.

To use it, run the `check-package` script, by telling which files you created or changed:

```
$ ./utils/check-package package/new-package/*
```

If you have the `utils` directory in your path you can also run:

```
$ cd package/new-package/
$ check-package *
```

The tool can also be used for packages in a br2-external:

```
$ check-package -b /path/to/br2-ext-tree/package/my-package/*
```

The `check-package` script requires you install `shellcheck` and the Python PyPi packages `flake8` and `python-magic`. The Buildroot code base is currently tested against version 0.7.1 of ShellCheck. If you use a different version of ShellCheck, you may see additional, unfixed, warnings.

If you have Docker or Podman you can run `check-package` without installing dependencies:

```
$ ./utils/docker-run ./utils/check-package
```
