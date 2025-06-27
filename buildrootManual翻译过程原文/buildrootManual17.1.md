## 19.3. Format and licensing of the package patches

Patches are released under the same license as the software they apply to (see [Section 13.2, “Complying with the Buildroot license”](https://buildroot.org/downloads/manual/manual.html#legal-info-buildroot)).

A message explaining what the patch does, and why it is needed, should be added in the header commentary of the patch.

You should add a `Signed-off-by` statement in the header of the each patch to help with keeping track of the changes and to certify that the patch is released under the same license as the software that is modified.

If the software is under version control, it is recommended to use the upstream SCM software to generate the patch set.

Otherwise, concatenate the header with the output of the `diff -purN package-version.orig/ package-version/` command.

If you update an existing patch (e.g. when bumping the package version), make sure the existing From header and Signed-off-by tags are not removed, but do update the rest of the patch comment when appropriate.

At the end, the patch should look like:

```
configure.ac: add C++ support test

Signed-off-by: John Doe <john.doe@noname.org>

--- configure.ac.orig
+++ configure.ac
@@ -40,2 +40,12 @@

AC_PROG_MAKE_SET
+
+AC_CACHE_CHECK([whether the C++ compiler works],
+               [rw_cv_prog_cxx_works],
+               [AC_LANG_PUSH([C++])
+                AC_LINK_IFELSE([AC_LANG_PROGRAM([], [])],
+                               [rw_cv_prog_cxx_works=yes],
+                               [rw_cv_prog_cxx_works=no])
+                AC_LANG_POP([C++])])
+
+AM_CONDITIONAL([CXX_WORKS], [test "x$rw_cv_prog_cxx_works" = "xyes"])
```

## 19.4. Additional patch documentation

Ideally, all patches should document an upstream patch or patch submission, when applicable, via the `Upstream` trailer.

When backporting an upstream patch that has been accepted into mainline, it is preferred that the URL to the commit is referenced:

```
Upstream: <URL to upstream commit>
```

If a new issue is identified in Buildroot and upstream is generally affected by the issue (it’s not a Buildroot specific issue), users should submit the patch upstream and provide a link to that submission when possible:

```
Upstream: <URL to upstream mailing list submission or merge request>
```

Patches that have been submitted but were denied upstream should note that and include comments about why the patch is being used despite the upstream status.

Note: in any of the above scenarios, it is also sensible to add a few words about any changes to the patch that may have been necessary.

If a patch does not apply upstream then this should be noted with a comment:

```
Upstream: N/A <additional information about why patch is Buildroot specific>
```

Adding this documentation helps streamline the patch review process during package version updates.

## Chapter 20. Download infrastructure

TODO

## Chapter 21. Debugging Buildroot

It is possible to instrument the steps `Buildroot` does when building packages. Define the variable `BR2_INSTRUMENTATION_SCRIPTS` to contain the path of one or more scripts (or other executables), in a space-separated list, you want called before and after each step. The scripts are called in sequence, with three parameters:

- `start` or `end` to denote the start (resp. the end) of a step;
- the name of the step about to be started, or which just ended;
- the name of the package.

For example :

```
make BR2_INSTRUMENTATION_SCRIPTS="/path/to/my/script1 /path/to/my/script2"
```

The list of steps is:

- `extract`
- `patch`
- `configure`
- `build`
- `install-host`, when a host-package is installed in `$(HOST_DIR)`
- `install-target`, when a target-package is installed in `$(TARGET_DIR)`
- `install-staging`, when a target-package is installed in `$(STAGING_DIR)`
- `install-image`, when a target-package installs files in `$(BINARIES_DIR)`

The script has access to the following variables:

- `BR2_CONFIG`: the path to the Buildroot .config file
- `HOST_DIR`, `STAGING_DIR`, `TARGET_DIR`: see [Section 18.6.2, “`generic-package` reference”](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)
- `BUILD_DIR`: the directory where packages are extracted and built
- `BINARIES_DIR`: the place where all binary files (aka images) are stored
- `BASE_DIR`: the base output directory
- `PARALLEL_JOBS`: the number of jobs to use when running parallel processes.

