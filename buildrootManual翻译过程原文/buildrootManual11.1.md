### 18.6.2. `generic-package` reference

There are two variants of the generic target. The `generic-package` macro is used for packages to be cross-compiled for the target. The `host-generic-package` macro is used for host packages, natively compiled for the host. It is possible to call both of them in a single `.mk` file: once to create the rules to generate a target package and once to create the rules to generate a host package:

```
$(eval $(generic-package))
$(eval $(host-generic-package))
```

This might be useful if the compilation of the target package requires some tools to be installed on the host. If the package name is `libfoo`, then the name of the package for the target is also `libfoo`, while the name of the package for the host is `host-libfoo`. These names should be used in the DEPENDENCIES variables of other packages, if they depend on `libfoo` or `host-libfoo`.

The call to the `generic-package` and/or `host-generic-package` macro ***\*must\**** be at the end of the `.mk` file, after all variable definitions. The call to `host-generic-package` ***\*must\**** be after the call to `generic-package`, if any.

For the target package, the `generic-package` uses the variables defined by the .mk file and prefixed by the uppercased package name: `LIBFOO_*`. `host-generic-package` uses the `HOST_LIBFOO_*` variables. For *some* variables, if the `HOST_LIBFOO_` prefixed variable doesn’t exist, the package infrastructure uses the corresponding variable prefixed by `LIBFOO_`. This is done for variables that are likely to have the same value for both the target and host packages. See below for details.

The list of variables that can be set in a `.mk` file to give metadata information is (assuming the package name is `libfoo`) :

- `LIBFOO_VERSION`, mandatory, must contain the version of the package. Note that if `HOST_LIBFOO_VERSION` doesn’t exist, it is assumed to be the same as `LIBFOO_VERSION`. It can also be a revision number or a tag for packages that are fetched directly from their version control system. Examples:

  - a version for a release tarball: `LIBFOO_VERSION = 0.1.2`

  - a sha1 for a git tree: `LIBFOO_VERSION = cb9d6aa9429e838f0e54faa3d455bcbab5eef057`

  - a tag for a git tree `LIBFOO_VERSION = v0.1.2`

    **Note:** Using a branch name as `FOO_VERSION` is not supported, because it does not and can not work as people would expect it should:

    1. due to local caching, Buildroot will not re-fetch the repository, so people who expect to be able to follow the remote repository would be quite surprised and disappointed;
    2. because two builds can never be perfectly simultaneous, and because the remote repository may get new commits on the branch anytime, two users, using the same Buildroot tree and building the same configuration, may get different source, thus rendering the build non reproducible, and people would be quite surprised and disappointed.

- `LIBFOO_SOURCE` may contain the name of the tarball of the package, which Buildroot will use to download the tarball from `LIBFOO_SITE`. If `HOST_LIBFOO_SOURCE` is not specified, it defaults to `LIBFOO_SOURCE`. If none are specified, then the value is assumed to be `libfoo-$(LIBFOO_VERSION).tar.gz`. Example: `LIBFOO_SOURCE = foobar-$(LIBFOO_VERSION).tar.bz2`

- `LIBFOO_PATCH` may contain a space-separated list of patch file names, that Buildroot will download and apply to the package source code. If an entry contains `://`, then Buildroot will assume it is a full URL and download the patch from this location. Otherwise, Buildroot will assume that the patch should be downloaded from `LIBFOO_SITE`. If `HOST_LIBFOO_PATCH` is not specified, it defaults to `LIBFOO_PATCH`. Note that patches that are included in Buildroot itself use a different mechanism: all files of the form `*.patch` present in the package directory inside Buildroot will be applied to the package after extraction (see [patching a package](https://buildroot.org/downloads/manual/manual.html#patch-policy)). Finally, patches listed in the `LIBFOO_PATCH` variable are applied *before* the patches stored in the Buildroot package directory.

- `LIBFOO_SITE` provides the location of the package, which can be a URL or a local filesystem path. HTTP, FTP and SCP are supported URL types for retrieving package tarballs. In these cases don’t include a trailing slash: it will be added by Buildroot between the directory and the filename as appropriate. Git, Subversion, Mercurial, and Bazaar are supported URL types for retrieving packages directly from source code management systems. There is a helper function to make it easier to download source tarballs from GitHub (refer to [Section 18.25.4, “How to add a package from GitHub”](https://buildroot.org/downloads/manual/manual.html#github-download-url) for details). A filesystem path may be used to specify either a tarball or a directory containing the package source code. See `LIBFOO_SITE_METHOD` below for more details on how retrieval works. Note that SCP URLs should be of the form `scp://[user@]host:filepath`, and that filepath is relative to the user’s home directory, so you may want to prepend the path with a slash for absolute paths: `scp://[user@]host:/absolutepath`. The same goes for SFTP URLs. If `HOST_LIBFOO_SITE` is not specified, it defaults to `LIBFOO_SITE`. Examples: `LIBFOO_SITE=http://www.libfoosoftware.org/libfoo` `LIBFOO_SITE=http://svn.xiph.org/trunk/Tremor` `LIBFOO_SITE=/opt/software/libfoo.tar.gz` `LIBFOO_SITE=$(TOPDIR)/../src/libfoo`

- `LIBFOO_DL_OPTS` is a space-separated list of additional options to pass to the downloader. Useful for retrieving documents with server-side checking for user logins and passwords, or to use a proxy. All download methods valid for `LIBFOO_SITE_METHOD` are supported; valid options depend on the download method (consult the man page for the respective download utilities). For git, `FOO_DL_OPTS` will only be passed to `git fetch` and no other git command (esp. not to `git lfs fetch` or `git submodule update`).

- `LIBFOO_EXTRA_DOWNLOADS` is a space-separated list of additional files that Buildroot should download. If an entry contains `://` then Buildroot will assume it is a complete URL and will download the file using this URL. Otherwise, Buildroot will assume the file to be downloaded is located at `LIBFOO_SITE`. Buildroot will not do anything with those additional files, except download them: it will be up to the package recipe to use them from `$(LIBFOO_DL_DIR)`.
