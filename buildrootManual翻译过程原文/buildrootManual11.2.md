- `LIBFOO_SITE_METHOD` determines the method used to fetch or copy the package source code. In many cases, Buildroot guesses the method from the contents of `LIBFOO_SITE` and setting `LIBFOO_SITE_METHOD` is unnecessary. When `HOST_LIBFOO_SITE_METHOD` is not specified, it defaults to the value of `LIBFOO_SITE_METHOD`. The possible values of `LIBFOO_SITE_METHOD` are:

  - `wget` for normal FTP/HTTP downloads of tarballs. Used by default when `LIBFOO_SITE` begins with `http://`, `https://` or `ftp://`.
  - `scp` for downloads of tarballs over SSH with scp. Used by default when `LIBFOO_SITE` begins with `scp://`.
  - `sftp` for downloads of tarballs over SSH with sftp. Used by default when `LIBFOO_SITE` begins with `sftp://`.
  - `svn` for retrieving source code from a Subversion repository. Used by default when `LIBFOO_SITE` begins with `svn://`. When a `http://` Subversion repository URL is specified in `LIBFOO_SITE`, one *must* specify `LIBFOO_SITE_METHOD=svn`. Buildroot performs a checkout which is preserved as a tarball in the download cache; subsequent builds use the tarball instead of performing another checkout.
  - `cvs` for retrieving source code from a CVS repository. Used by default when `LIBFOO_SITE` begins with `cvs://`. The downloaded source code is cached as with the `svn` method. Anonymous pserver mode is assumed otherwise explicitly defined on `LIBFOO_SITE`. Both `LIBFOO_SITE=cvs://libfoo.net:/cvsroot/libfoo` and `LIBFOO_SITE=cvs://:ext:libfoo.net:/cvsroot/libfoo` are accepted, on the former anonymous pserver access mode is assumed. `LIBFOO_SITE` *must* contain the source URL as well as the remote repository directory. The module is the package name. `LIBFOO_VERSION` is *mandatory* and *must* be a tag, a branch, or a date (e.g. "2014-10-20", "2014-10-20 13:45", "2014-10-20 13:45+01" see "man cvs" for further details).
  - `git` for retrieving source code from a Git repository. Used by default when `LIBFOO_SITE` begins with `git://`. The downloaded source code is cached as with the `svn` method.
  - `hg` for retrieving source code from a Mercurial repository. One *must* specify `LIBFOO_SITE_METHOD=hg` when `LIBFOO_SITE` contains a Mercurial repository URL. The downloaded source code is cached as with the `svn` method.
  - `bzr` for retrieving source code from a Bazaar repository. Used by default when `LIBFOO_SITE` begins with `bzr://`. The downloaded source code is cached as with the `svn` method.
  - `file` for a local tarball. One should use this when `LIBFOO_SITE` specifies a package tarball as a local filename. Useful for software that isn’t available publicly or in version control.
  - `local` for a local source code directory. One should use this when `LIBFOO_SITE` specifies a local directory path containing the package source code. Buildroot copies the contents of the source directory into the package’s build directory. Note that for `local` packages, no patches are applied. If you need to still patch the source code, use `LIBFOO_POST_RSYNC_HOOKS`, see [Section 18.23.1, “Using the `POST_RSYNC` hook”](https://buildroot.org/downloads/manual/manual.html#hooks-rsync).
  - `smb` for retrieving source code from a SMB share. Used by default when `LIBFOO_SITE` begins with `smb://`. It uses `curl` as download backend. Syntax expected: `LIBFOO_SITE=smb://<server>/<share>/<path>`. This method might require to define -u option in `LIBFOO_DL_OPTS`. For more information, please refer to the [curl documentation](https://curl.se/docs/tutorial.html).

- `LIBFOO_GIT_SUBMODULES` can be set to `YES` to create an archive with the git submodules in the repository. This is only available for packages downloaded with git (i.e. when `LIBFOO_SITE_METHOD=git`). Note that we try not to use such git submodules when they contain bundled libraries, in which case we prefer to use those libraries from their own package.

- `LIBFOO_GIT_LFS` should be set to `YES` if the Git repository uses Git LFS to store large files out of band. This is only available for packages downloaded with git (i.e. when `LIBFOO_SITE_METHOD=git`).

- `LIBFOO_SVN_EXTERNALS` can be set to `YES` to create an archive with the svn external references. This is only available for packages downloaded with subversion.

- `LIBFOO_STRIP_COMPONENTS` is the number of leading components (directories) that tar must strip from file names on extraction. The tarball for most packages has one leading component named "<pkg-name>-<pkg-version>", thus Buildroot passes --strip-components=1 to tar to remove it. For non-standard packages that don’t have this component, or that have more than one leading component to strip, set this variable with the value to be passed to tar. Default: 1.

- `LIBFOO_EXCLUDES` is a space-separated list of patterns to exclude when extracting the archive. Each item from that list is passed as a tar’s `--exclude` option. By default, empty.

- `LIBFOO_DEPENDENCIES` lists the dependencies (in terms of package name) that are required for the current target package to compile. These dependencies are guaranteed to be compiled and installed before the configuration of the current package starts. However, modifications to configuration of these dependencies will not force a rebuild of the current package. In a similar way, `HOST_LIBFOO_DEPENDENCIES` lists the dependencies for the current host package.

- `LIBFOO_EXTRACT_DEPENDENCIES` lists the dependencies (in terms of package name) that are required for the current target package to be extracted. These dependencies are guaranteed to be compiled and installed before the extract step of the current package starts. This is only used internally by the package infrastructure, and should typically not be used directly by packages.

- `LIBFOO_PATCH_DEPENDENCIES` lists the dependencies (in terms of package name) that are required for the current package to be patched. These dependencies are guaranteed to be extracted and patched (but not necessarily built) before the current package is patched. In a similar way, `HOST_LIBFOO_PATCH_DEPENDENCIES` lists the dependencies for the current host package. This is seldom used; usually, `LIBFOO_DEPENDENCIES` is what you really want to use.

- `LIBFOO_PROVIDES` lists all the virtual packages `libfoo` is an implementation of. See [Section 18.12, “Infrastructure for virtual packages”](https://buildroot.org/downloads/manual/manual.html#virtual-package-tutorial).

- `LIBFOO_INSTALL_STAGING` can be set to `YES` or `NO` (default). If set to `YES`, then the commands in the `LIBFOO_INSTALL_STAGING_CMDS` variables are executed to install the package into the staging directory.

- `LIBFOO_INSTALL_TARGET` can be set to `YES` (default) or `NO`. If set to `YES`, then the commands in the `LIBFOO_INSTALL_TARGET_CMDS` variables are executed to install the package into the target directory.

- `LIBFOO_INSTALL_IMAGES` can be set to `YES` or `NO` (default). If set to `YES`, then the commands in the `LIBFOO_INSTALL_IMAGES_CMDS` variable are executed to install the package into the images directory.

- `LIBFOO_CONFIG_SCRIPTS` lists the names of the files in *$(STAGING_DIR)/usr/bin* that need some special fixing to make them cross-compiling friendly. Multiple file names separated by space can be given and all are relative to *$(STAGING_DIR)/usr/bin*. The files listed in `LIBFOO_CONFIG_SCRIPTS` are also removed from `$(TARGET_DIR)/usr/bin` since they are not needed on the target.

- `LIBFOO_DEVICES` lists the device files to be created by Buildroot when using the static device table. The syntax to use is the makedevs one. You can find some documentation for this syntax in the [Chapter 25, *Makedev syntax documentation*](https://buildroot.org/downloads/manual/manual.html#makedev-syntax). This variable is optional.

- `LIBFOO_PERMISSIONS` lists the changes of permissions to be done at the end of the build process. The syntax is once again the makedevs one. You can find some documentation for this syntax in the [Chapter 25, *Makedev syntax documentation*](https://buildroot.org/downloads/manual/manual.html#makedev-syntax). This variable is optional.

  
