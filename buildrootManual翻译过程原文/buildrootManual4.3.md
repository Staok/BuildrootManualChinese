### 8.13.4. Location of downloaded packages

The various tarballs that are downloaded by Buildroot are all stored in `BR2_DL_DIR`, which by default is the `dl` directory. If you want to keep a complete version of Buildroot which is known to be working with the associated tarballs, you can make a copy of this directory. This will allow you to regenerate the toolchain and the target filesystem with exactly the same versions.

If you maintain several Buildroot trees, it might be better to have a shared download location. This can be achieved by pointing the `BR2_DL_DIR` environment variable to a directory. If this is set, then the value of `BR2_DL_DIR` in the Buildroot configuration is overridden. The following line should be added to `<~/.bashrc>`.

```
 export BR2_DL_DIR=<shared download location>
```

The download location can also be set in the `.config` file, with the `BR2_DL_DIR` option. Unlike most options in the .config file, this value is overridden by the `BR2_DL_DIR` environment variable.

### 8.13.5. Package-specific *make* targets

Running `make <package>` builds and installs that particular package and its dependencies.

For packages relying on the Buildroot infrastructure, there are numerous special make targets that can be called independently like this:

```
make <package>-<target>
```

The package build targets are (in the order they are executed):

| command/target    | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `source`          | Fetch the source (download the tarball, clone the source repository, etc) |
| `depends`         | Build and install all dependencies required to build the package |
| `extract`         | Put the source in the package build directory (extract the tarball, copy the source, etc) |
| `patch`           | Apply the patches, if any                                    |
| `configure`       | Run the configure commands, if any                           |
| `build`           | Run the compilation commands                                 |
| `install-staging` | ***\*target package:\**** Run the installation of the package in the staging directory, if necessary |
| `install-target`  | ***\*target package:\**** Run the installation of the package in the target directory, if necessary |
| `install`         | ***\*target package:\**** Run the 2 previous installation commands***\*host package:\**** Run the installation of the package in the host directory |

Additionally, there are some other useful make targets:

| command/target            | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `show-depends`            | Displays the first-order dependencies required to build the package |
| `show-recursive-depends`  | Recursively displays the dependencies required to build the package |
| `show-rdepends`           | Displays the first-order reverse dependencies of the package (i.e packages that directly depend on it) |
| `show-recursive-rdepends` | Recursively displays the reverse dependencies of the package (i.e the packages that depend on it, directly or indirectly) |
| `graph-depends`           | Generate a dependency graph of the package, in the context of the current Buildroot configuration. See [this section](https://buildroot.org/downloads/manual/manual.html#graph-depends) for more details about dependency graphs. |
| `graph-rdepends`          | Generate a graph of this package reverse dependencies (i.e the packages that depend on it, directly or indirectly) |
| `graph-both-depends`      | Generate a graph of this package in both directions (i.e the packages that depend on it and on which it depends, directly or indirectly) |
| `dirclean`                | Remove the whole package build directory                     |
| `reinstall`               | Re-run the install commands                                  |
| `rebuild`                 | Re-run the compilation commands - this only makes sense when using the `OVERRIDE_SRCDIR` feature or when you modified a file directly in the build directory |
| `reconfigure`             | Re-run the configure commands, then rebuild - this only makes sense when using the `OVERRIDE_SRCDIR` feature or when you modified a file directly in the build directory |

### 8.13.6. Using Buildroot during development

The normal operation of Buildroot is to download a tarball, extract it, configure, compile and install the software component found inside this tarball. The source code is extracted in `output/build/<package>-<version>`, which is a temporary directory: whenever `make clean` is used, this directory is entirely removed, and re-created at the next `make` invocation. Even when a Git or Subversion repository is used as the input for the package source code, Buildroot creates a tarball out of it, and then behaves as it normally does with tarballs.

This behavior is well-suited when Buildroot is used mainly as an integration tool, to build and integrate all the components of an embedded Linux system. However, if one uses Buildroot during the development of certain components of the system, this behavior is not very convenient: one would instead like to make a small change to the source code of one package, and be able to quickly rebuild the system with Buildroot.

Making changes directly in `output/build/<package>-<version>` is not an appropriate solution, because this directory is removed on `make clean`.

Therefore, Buildroot provides a specific mechanism for this use case: the `<pkg>_OVERRIDE_SRCDIR` mechanism. Buildroot reads an *override* file, which allows the user to tell Buildroot the location of the source for certain packages.

The default location of the override file is `$(CONFIG_DIR)/local.mk`, as defined by the `BR2_PACKAGE_OVERRIDE_FILE` configuration option. `$(CONFIG_DIR)` is the location of the Buildroot `.config` file, so `local.mk` by default lives side-by-side with the `.config` file, which means:

- In the top-level Buildroot source directory for in-tree builds (i.e., when `O=` is not used)
- In the out-of-tree directory for out-of-tree builds (i.e., when `O=` is used)

If a different location than these defaults is required, it can be specified through the `BR2_PACKAGE_OVERRIDE_FILE` configuration option.

In this *override* file, Buildroot expects to find lines of the form:

```
<pkg1>_OVERRIDE_SRCDIR = /path/to/pkg1/sources
<pkg2>_OVERRIDE_SRCDIR = /path/to/pkg2/sources
```

For example:

```
LINUX_OVERRIDE_SRCDIR = /home/bob/linux/
BUSYBOX_OVERRIDE_SRCDIR = /home/bob/busybox/
```

When Buildroot finds that for a given package, an `<pkg>_OVERRIDE_SRCDIR` has been defined, it will no longer attempt to download, extract and patch the package. Instead, it will directly use the source code available in the specified directory and `make clean` will not touch this directory. This allows to point Buildroot to your own directories, that can be managed by Git, Subversion, or any other version control system. To achieve this, Buildroot will use *rsync* to copy the source code of the component from the specified `<pkg>_OVERRIDE_SRCDIR` to `output/build/<package>-custom/`.

This mechanism is best used in conjunction with the `make <pkg>-rebuild` and `make <pkg>-reconfigure` targets. A `make <pkg>-rebuild all` sequence will *rsync* the source code from `<pkg>_OVERRIDE_SRCDIR` to `output/build/<package>-custom` (thanks to *rsync*, only the modified files are copied), and restart the build process of just this package.

In the example of the `linux` package above, the developer can then make a source code change in `/home/bob/linux` and then run:

```
make linux-rebuild all
```

and in a matter of seconds gets the updated Linux kernel image in `output/images`. Similarly, a change can be made to the BusyBox source code in `/home/bob/busybox`, and after:

```
make busybox-rebuild all
```

the root filesystem image in `output/images` contains the updated BusyBox.

Source trees for big projects often contain hundreds or thousands of files which are not needed for building, but will slow down the process of copying the sources with *rsync*. Optionally, it is possible define `<pkg>_OVERRIDE_SRCDIR_RSYNC_EXCLUSIONS` to skip syncing certain files from the source tree. For example, when working on the `webkitgtk` package, the following will exclude the tests and in-tree builds from a local WebKit source tree:

```
WEBKITGTK_OVERRIDE_SRCDIR = /home/bob/WebKit
WEBKITGTK_OVERRIDE_SRCDIR_RSYNC_EXCLUSIONS = \
        --exclude JSTests --exclude ManualTests --exclude PerformanceTests \
        --exclude WebDriverTests --exclude WebKitBuild --exclude WebKitLibraries \
        --exclude WebKit.xcworkspace --exclude Websites --exclude Examples
```

By default, Buildroot skips syncing of VCS artifacts (e.g., the ***\*.git\**** and ***\*.svn\**** directories). Some packages prefer to have these VCS directories available during build, for example for automatically determining a precise commit reference for version information. To undo this built-in filtering at a cost of a slower speed, add these directories back:

```
LINUX_OVERRIDE_SRCDIR_RSYNC_EXCLUSIONS = --include .git
```

