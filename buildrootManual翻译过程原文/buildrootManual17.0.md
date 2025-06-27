## Chapter 19. Patching a package

While integrating a new package or updating an existing one, it may be necessary to patch the source of the software to get it cross-built within Buildroot.

Buildroot offers an infrastructure to automatically handle this during the builds. It supports three ways of applying patch sets: downloaded patches, patches supplied within buildroot and patches located in a user-defined global patch directory.

## 19.1. Providing patches

### 19.1.1. Downloaded

If it is necessary to apply a patch that is available for download, then add it to the `<packagename>_PATCH` variable. If an entry contains `://`, then Buildroot will assume it is a full URL and download the patch from this location. Otherwise, Buildroot will assume that the patch should be downloaded from `<packagename>_SITE`. It can be a single patch, or a tarball containing a patch series.

Like for all downloads, a hash should be added to the `<packagename>.hash` file.

This method is typically used for packages from Debian.

### 19.1.2. Within Buildroot

Most patches are provided within Buildroot, in the package directory; these typically aim to fix cross-compilation, libc support, or other such issues.

These patch files should be named `<number>-<description>.patch`.

**Notes**

- The patch files coming with Buildroot should not contain any package version reference in their filename.
- The field `<number>` in the patch file name refers to the *apply order*, and shall start at 1; It is preferred to pad the number with zeros up to 4 digits, like *git-format-patch* does. E.g.: `0001-foobar-the-buz.patch`
- The patch email subject prefix shall not be numbered. Patches shall be generated with the `git format-patch -N` command, since this numbering is automatically added for series. For example, the patch subject line should look like `Subject: [PATCH] foobar the buz` rather than `Subject: [PATCH n/m] foobar the buz`.
- Previously, it was mandatory for patches to be prefixed with the name of the package, like `<package>-<number>-<description>.patch`, but that is no longer the case. Existing packages will be fixed as time passes. *Do not prefix patches with the package name.*
- Previously, a `series` file, as used by `quilt`, could also be added in the package directory. In that case, the `series` file defines the patch application order. This is deprecated, and will be removed in the future. *Do not use a series file.*

### 19.1.3. Global patch directory

The `BR2_GLOBAL_PATCH_DIR` configuration file option can be used to specify a space separated list of one or more directories containing global package patches. See [Section 9.8.1, “Providing extra patches”](https://buildroot.org/downloads/manual/manual.html#customize-patches) for details.

## 19.2. How patches are applied

1. Run the `<packagename>_PRE_PATCH_HOOKS` commands if defined;
2. Cleanup the build directory, removing any existing `*.rej` files;
3. If `<packagename>_PATCH` is defined, then patches from these tarballs are applied;
4. If there are some `*.patch` files in the package’s Buildroot directory or in a package subdirectory named `<packageversion>`, then:
   - If a `series` file exists in the package directory, then patches are applied according to the `series` file;
   - Otherwise, patch files matching `*.patch` are applied in alphabetical order. So, to ensure they are applied in the right order, it is highly recommended to name the patch files like this: `<number>-<description>.patch`, where `<number>` refers to the *apply order*.
5. If `BR2_GLOBAL_PATCH_DIR` is defined, the directories will be enumerated in the order they are specified. The patches are applied as described in the previous step.
6. Run the `<packagename>_POST_PATCH_HOOKS` commands if defined.

If something goes wrong in the steps *3* or *4*, then the build fails.
