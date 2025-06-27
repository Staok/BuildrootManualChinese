## Chapter 26. Makeusers syntax documentation

The syntax to create users is inspired by the makedev syntax, above, but is specific to Buildroot.

The syntax for adding a user is a space-separated list of fields, one user per line; the fields are:

| username | uid  | group | gid  | password | home | shell | groups | comment |
| -------- | ---- | ----- | ---- | -------- | ---- | ----- | ------ | ------- |
|          |      |       |      |          |      |       |        |         |

Where:

- `username` is the desired user name (aka login name) for the user. It can not be `root`, and must be unique. If set to `-`, then just a group will be created.
- `uid` is the desired UID for the user. It must be unique, and not `0`. If set to `-1` or `-2`, then a unique UID will be computed by Buildroot, with `-1` denoting a system UID from [100…999] and `-2` denoting a user UID from [1000…1999].
- `group` is the desired name for the user’s main group. It can not be `root`. If the group does not exist, it will be created.
- `gid` is the desired GID for the user’s main group. It must be unique, and not `0`. If set to `-1` or `-2`, and the group does not already exist, then a unique GID will be computed by Buildroot, with `-1` denoting a system GID from [100…999] and `-2` denoting a user GID from [1000…1999].
- `password` is the crypt(3)-encoded password. If prefixed with `!`, then login is disabled. If prefixed with `=`, then it is interpreted as clear-text, and will be crypt-encoded (using MD5). If prefixed with `!=`, then the password will be crypt-encoded (using MD5) and login will be disabled. If set to `*`, then login is not allowed. If set to `-`, then no password value will be set.
- `home` is the desired home directory for the user. If set to *-*, no home directory will be created, and the user’s home will be `/`. Explicitly setting `home` to `/` is not allowed.
- `shell` is the desired shell for the user. If set to `-`, then `/bin/false` is set as the user’s shell.
- `groups` is the comma-separated list of additional groups the user should be part of. If set to `-`, then the user will be a member of no additional group. Missing groups will be created with an arbitrary `gid`.
- `comment` (aka [GECOS](https://en.wikipedia.org/wiki/Gecos_field) field) is an almost-free-form text.

There are a few restrictions on the content of each field:

- except for `comment`, all fields are mandatory.
- except for `comment`, fields may not contain spaces.
- no field may contain a colon (`:`).

If `home` is not `-`, then the home directory, and all files below, will belong to the user and its main group.

Examples:

```
foo -1 bar -1 !=blabla /home/foo /bin/sh alpha,bravo Foo user
```

This will create this user:

- `username` (aka login name) is: `foo`
- `uid` is computed by Buildroot
- main `group` is: `bar`
- main group `gid` is computed by Buildroot
- clear-text `password` is: `blabla`, will be crypt(3)-encoded, and login is disabled.
- `home` is: `/home/foo`
- `shell` is: `/bin/sh`
- `foo` is also a member of `groups`: `alpha` and `bravo`
- `comment` is: `Foo user`

```
test 8000 wheel -1 = - /bin/sh - Test user
```

This will create this user:

- `username` (aka login name) is: `test`
- `uid` is : `8000`
- main `group` is: `wheel`
- main group `gid` is computed by Buildroot, and will use the value defined in the rootfs skeleton
- `password` is empty (aka no password).
- `home` is `/` but will not belong to `test`
- `shell` is: `/bin/sh`
- `test` is not a member of any additional `groups`
- `comment` is: `Test user`

## 26.1. Caveat with automatic UIDs and GIDs

When updating buildroot or when packages are added or removed to/from the configuration, it is possible that the automatic UIDs and GIDs are changed. This can be a problem if persistent files were created with that user or group: after upgrade, they will suddenly have a different owner.

Therefore, it is advisable to perpetuate the automatic IDs. This can be done by adding a users table with the generated IDs. It is only needed to do this for UIDs that actually create persistent files, e.g. database.

## Chapter 27. Migrating from older Buildroot versions

Some versions have introduced backward incompatibilities. This section explains those incompatibilities, and for each explains what to do to complete the migration.

## 27.1. General approach

To migrate from an older Buildroot version, take the following steps.

1. For all your configurations, do a build in the old Buildroot environment. Run `make graph-size`. Save `graphs/file-size-stats.csv` in a different location. Run `make clean` to remove the rest.
2. Review the specific migration notes below and make the required adaptations to external packages and custom build scripts.
3. Update Buildroot.
4. Run `make menuconfig` starting from the existing `.config`.
5. If anything is enabled in the Legacy menu, check its help text, unselect it, and save the configuration.
6. For more details, review the git commit messages for the packages that you need. Change into the `packages` directory and run `git log <old version>.. — <your packages>`.
7. Build in the new Buildroot environment.
8. Fix build issues in external packages (usually due to updated dependencies).
9. Run `make graph-size`.
10. Compare the new `file-size-stats.csv` with the original one, to check if no required files have disappeared and if no new big unneeded files have appeared.
11. For configuration (and other) files in a custom overlay that overwrite files created by Buildroot, check if there are changes in the Buildroot-generated file that need to be propagated to your custom file.

## 27.2. Migrating to 2016.11

Before Buildroot 2016.11, it was possible to use only one br2-external tree at once. With Buildroot 2016.11 came the possibility to use more than one simultaneously (for details, see [Section 9.2, “Keeping customizations outside of Buildroot”](https://buildroot.org/downloads/manual/manual.html#outside-br-custom)).

This however means that older br2-external trees are not usable as-is. A minor change has to be made: adding a name to your br2-external tree.

This can be done very easily in just a few steps:

- First, create a new file named `external.desc`, at the root of your br2-external tree, with a single line defining the name of your br2-external tree:

  ```
  $ echo 'name: NAME_OF_YOUR_TREE' >external.desc
  ```

  **Note.** Be careful when choosing a name: It has to be unique and be made with only ASCII characters from the set `[A-Za-z0-9_]`.

- Then, change every occurrence of `BR2_EXTERNAL` in your br2-external tree with the new variable:

  ```
  $ find . -type f | xargs sed -i 's/BR2_EXTERNAL/BR2_EXTERNAL_NAME_OF_YOUR_TREE_PATH/g'
  ```

Now, your br2-external tree can be used with Buildroot 2016.11 onward.

**Note:** This change makes your br2-external tree incompatible with Buildroot before 2016.11.

## 27.3. Migrating to 2017.08

Before Buildroot 2017.08, host packages were installed in `$(HOST_DIR)/usr` (with e.g. the autotools' `--prefix=$(HOST_DIR)/usr`). With Buildroot 2017.08, they are now installed directly in `$(HOST_DIR)`.

Whenever a package installs an executable that is linked with a library in `$(HOST_DIR)/lib`, it must have an RPATH pointing to that directory.

An RPATH pointing to `$(HOST_DIR)/usr/lib` is no longer accepted.

## 27.4. Migrating to 2023.11

Before Buildroot 2023.11, the subversion download backend unconditionally retrieved the external references (objects with an `svn:externals` property). Starting with 2023.11, externals are no longer retrieved by default; if you need them, set `LIBFOO_SVN_EXTERNALS` to `YES`. This change implies that:

- the generated archive content may change, and thus the hashes may need to be updated appropriately;
- the archive version suffix has been updated to `-br3`, so the hash files must be updated appropriately.

Before Buildroot 2023.11, it was possible (but undocumented and unused) to apply architecture-specific patches, by prefixing the patch filename with the architecture, e.g. `0001-some-changes.patch.arm` and such a patch would only be applied for that architecture. With Buildroot 2023.11, this is no longer supported, and such patches are no longer applied at all.

If you still need per-architecture patches, then you may provide a [pre-patch hook](https://buildroot.org/downloads/manual/manual.html#hooks) that copies the patches applicable to the configured architecture, e.g.:

```
define LIBFOO_ARCH_PATCHES
    $(foreach p,$(wildcard $(LIBFOO_PKGDIR)/*.patch.$(ARCH)), \
        cp -f $(p) $(patsubst %.$(ARCH),%,$(p))
    )
endef
LIBFOO_PRE_PATCH_HOOKS += LIBFOO_ARCH_PATCHES
```

Note that no package in Buildroot has architecture-specific patches, and that such patches will most probably not be accepted.

## 27.5. Migrating to 2024.05

The download backends have been extended in various ways.

- All locally generated tarballs are even more reproducible. Before 2024.05, it was possible that the access mode of files in the archives were not consistent when the download directory has specific ACLs (e.g. with the `default` ACL set). This impacts the archives generated for git and subversion repositories, as well as those for vendored cargo and go packages.
- The git download backend now properly expands the `export-subst` [git attribute](https://git-scm.com/docs/gitattributes) when generating archives.
- A newer `tar` version, *1.35*, is required to generate the archives. For compatibility reasons, `tar` 1.35 changes the way it stores some fields (`devmajor` and `devminor`), which has an impact in the metadata stored in the archives (but the content of files is not affected).

To accommodate those changes, the archive suffix has been updated or added:

- for git: `-git4`
- for subversion: `-svn5`
- for cargo (rust) packages: `-cargo2`
- for go packages: `-go2`

Note that, if two such prefixes would apply to a generated archive, like for a cargo package downloaded from git, both suffixes need to be added, first the one for the download mechanism, then the one for the vendoring, e.g.: `libfoo-1.2.3-git4-cargo2.tar.gz`.

Because of this, the hash file of any custom packages or custom versions for kernel and bootloaders must be updated. The following sed scripts can automate the rename in the hash file (assuming such files are kept under git):

```
# For git and svn packages, which originally had -br2 resp. -br3 suffix
sed -r -i -e 's/-br2/-git4/; s/-br3/-svn5/' $(
    git grep -l -E -- '-br2|-br3' -- '*.hash'
)

# For go packages, which originally had no suffix
sed -r -i -e 's/(\.tar\.gz)$/-go2\1/' $(
    git grep -l -E '\$\(eval \$\((host-)?golang-package\)\)' -- '*.mk' \
    |sed -r -e 's/\.mk$/.hash/' \
    |sort -u
)

# For cargo packages, which originally had no suffix
sed -r -i -e 's/(\.tar\.gz)$/-cargo2\1/' $(
    git grep -l -E '\$\(eval \$\((host-)?cargo-package\)\)' -- '*.mk' \
    |sed -r -e 's/\.mk$/.hash/' \
    |sort -u
)
```

Note that the hash *will* have changed, so that needs to be updated (manually) as well.

## 27.6. Migrating to 2025.02

Mender now requires a special bootstrap artifact to be placed in `/var/lib/mender`. This replaces the `artifact_info` file. Just like a normal artifact, the bootstrap artifact is generated with host-mender-artifact. See `board/mender/x86_64/post-image-efi.sh` for an example of how to generate the bootstrap.mender file. See [the release notes](https://docs.mender.io/release-information/release-notes-changelog/mender-client#mender-3-5-0-1), under features, for more information.

## 27.7. Migrating to 2025.05

In 2025.05, for SYS-V-like systems (busybox, sysvinit, openrc), the `/etc/resolv.conf` symlink was changed to point to `/run/resolv.conf`, rather than the legacy location in `/tmp`. Users of a custom `fstab` will need to ensure that `/run` is writable before resolv.conf is created (usually by a DHCP client), either with an entry for `/run`, or with a startup script.

Note that systems using systemd are not impacted: systemd always ensures that `/run` is writable. Systems further using systemd-resolved already had a `/etc/resolv.conf` that pointed into `/run` anyway.

Due to the update of Rust to a version greater than 1.84.0 making the Cargo.lock file mandatory and the change from `.cargo/config` to `.cargo/config.toml`, tarballs generated by Cargo-fetched packages have changed. Therefore the suffix of such tarballs has been changed from `-cargo2` to `-cargo4`.

