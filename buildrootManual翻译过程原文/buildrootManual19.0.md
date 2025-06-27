## 22.7. Using the runtime tests framework

Buildroot includes a run-time testing framework built upon Python scripting and QEMU runtime execution. The goals of the framework are the following:

- build a well defined Buildroot configuration
- optionally, verify some properties of the build output
- optionally, boot the build results under Qemu, and verify that a given feature is working as expected

The entry point to use the runtime tests framework is the `support/testing/run-tests` tool, which has a series of options documented in the tool’s help *-h* description. Some common options include setting the download folder, the output folder, keeping build output, and for multiple test cases, you can set the JLEVEL for each.

Here is an example walk through of running a test case.

- For a first step, let us see what all the test case options are. The test cases can be listed by executing `support/testing/run-tests -l`. These tests can all be run individually during test development from the console. Both one at a time and selectively as a group of a subset of tests.

```
$ support/testing/run-tests -l
List of tests
test_run (tests.utils.test_check_package.TestCheckPackage)
test_run (tests.toolchain.test_external.TestExternalToolchainBuildrootMusl) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainBuildrootuClibc) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainCCache) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainCtngMusl) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainLinaroArm) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainSourceryArmv4) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainSourceryArmv5) ... ok
test_run (tests.toolchain.test_external.TestExternalToolchainSourceryArmv7) ... ok
[snip]
test_run (tests.init.test_systemd.TestInitSystemSystemdRoFull) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRoIfupdown) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRoNetworkd) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRwFull) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRwIfupdown) ... ok
test_run (tests.init.test_systemd.TestInitSystemSystemdRwNetworkd) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRo) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRoNet) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRw) ... ok
test_run (tests.init.test_busybox.TestInitSystemBusyboxRwNet) ... ok

Ran 157 tests in 0.021s

OK
```

- Then, to run one test case:

```
$ support/testing/run-tests -d dl -o output_folder -k tests.init.test_busybox.TestInitSystemBusyboxRw
15:03:26 TestInitSystemBusyboxRw                  Starting
15:03:28 TestInitSystemBusyboxRw                  Building
15:08:18 TestInitSystemBusyboxRw                  Building done
15:08:27 TestInitSystemBusyboxRw                  Cleaning up
.
Ran 1 test in 301.140s

OK
```

The standard output indicates if the test is successful or not. By default, the output folder for the test is deleted automatically unless the option `-k` is passed to ***\*keep\**** the output directory.

### 22.7.1. Creating a test case

Within the Buildroot repository, the testing framework is organized at the top level in `support/testing/` by folders of `conf`, `infra` and `tests`. All the test cases live under the `tests` folder and are organized in various folders representing the category of test.

The best way to get familiar with how to create a test case is to look at a few of the basic file system `support/testing/tests/fs/` and init `support/testing/tests/init/` test scripts. Those tests give good examples of a basic tests that include both checking the build results, and doing runtime tests. There are other more advanced cases that use things like nested `br2-external` folders to provide skeletons and additional packages.

Creating a basic test case involves:

- Defining a test class that inherits from `infra.basetest.BRTest`
- Defining the `config` member of the test class, to the Buildroot configuration to build for this test case. It can optionally rely on configuration snippets provided by the runtime test infrastructure: `infra.basetest.BASIC_TOOLCHAIN_CONFIG` to get a basic architecture/toolchain configuration, and `infra.basetest.MINIMAL_CONFIG` to not build any filesystem. The advantage of using `infra.basetest.BASIC_TOOLCHAIN_CONFIG` is that a matching Linux kernel image is provided, which allows to boot the resulting image in Qemu without having to build a Linux kernel image as part of the test case, therefore significant decreasing the build time required for the test case.
- Implementing a `def test_run(self):` function to implement the actual tests to run after the build has completed. They may be tests that verify the build output, by running command on the host using the `run_cmd_on_host()` helper function. Or they may boot the generated system in Qemu using the `Emulator` object available as `self.emulator` in the test case. For example `self.emulator.boot()` allows to boot the system in Qemu, `self.emulator.login()` allows to login, `self.emulator.run()` allows to run shell commands inside Qemu.

After creating the test script, add yourself to the `DEVELOPERS` file to be the maintainer of that test case.

### 22.7.2. Debugging a test case

When a test case runs, the `output_folder` will contain the following:

```
$ ls output_folder/
TestInitSystemBusyboxRw/
TestInitSystemBusyboxRw-build.log
TestInitSystemBusyboxRw-run.log
```

`TestInitSystemBusyboxRw/` is the Buildroot output directory, and it is preserved only if the `-k` option is passed.

`TestInitSystemBusyboxRw-build.log` is the log of the Buildroot build.

`TestInitSystemBusyboxRw-run.log` is the log of the Qemu boot and test. This file will only exist if the build was successful and the test case involves booting under Qemu.

If you want to manually run Qemu to do manual tests of the build result, the first few lines of `TestInitSystemBusyboxRw-run.log` contain the Qemu command line to use.

You can also make modifications to the current sources inside the `output_folder` (e.g. for debug purposes) and rerun the standard Buildroot make targets (in order to regenerate the complete image with the new modifications) and then rerun the test.

### 22.7.3. Runtime tests and Gitlab CI

All runtime tests are regularly executed by Buildroot Gitlab CI infrastructure, see .gitlab.yml and https://gitlab.com/buildroot.org/buildroot/-/jobs.

You can also use Gitlab CI to test your new test cases, or verify that existing tests continue to work after making changes in Buildroot.

In order to achieve this, you need to create a fork of the Buildroot project on Gitlab, and be able to push branches to your Buildroot fork on Gitlab.

The name of the branch that you push will determine if a Gitlab CI pipeline will be triggered or not, and for which test cases.

In the examples below, the <name> component of the branch name is an arbitrary string you choose.

- To trigger all run-test test case jobs, push a branch that ends with `-runtime-tests`:

```
 $ git push gitlab HEAD:<name>-runtime-tests
```

- To trigger one or several test case jobs, push a branch that ends with the complete test case name (`tests.init.test_busybox.TestInitSystemBusyboxRo`) or with the name of a category of tests (`tests.init.test_busybox`):

```
 $ git push gitlab HEAD:<name>-<test case name>
```

Example to run one test:

```
 $ git push gitlab HEAD:foo-tests.init.test_busybox.TestInitSystemBusyboxRo
```

Examples to run several tests part of the same group:

```
 $ git push gitlab HEAD:foo-tests.init.test_busybox
 $ git push gitlab HEAD:foo-tests.init
```

------

[[4\] ](https://buildroot.org/downloads/manual/manual.html#idm6107)RFC: (Request for comments) change proposal

## Chapter 23. DEVELOPERS file and get-developers

The main Buildroot directory contains a file named `DEVELOPERS` that lists the developers involved with various areas of Buildroot. Thanks to this file, the `get-developers` tool allows to:

- Calculate the list of developers to whom patches should be sent, by parsing the patches and matching the modified files with the relevant developers. See [Section 22.5, “Submitting patches”](https://buildroot.org/downloads/manual/manual.html#submitting-patches) for details.
- Find which developers are taking care of a given architecture or package, so that they can be notified when a build failure occurs on this architecture or package. This is done in interaction with Buildroot’s autobuild infrastructure.

We ask developers adding new packages, new boards, or generally new functionality in Buildroot, to register themselves in the `DEVELOPERS` file. As an example, we expect a developer contributing a new package to include in his patch the appropriate modification to the `DEVELOPERS` file.

The `DEVELOPERS` file format is documented in detail inside the file itself.

The `get-developers` tool, located in `utils/` allows to use the `DEVELOPERS` file for various tasks:

- When passing one or several patches as command line argument, `get-developers` will return the appropriate `git send-email` command. If the `-e` option is passed, only the email addresses are printed in a format suitable for `git send-email --cc-cmd`.
- When using the `-a <arch>` command line option, `get-developers` will return the list of developers in charge of the given architecture.
- When using the `-p <package>` command line option, `get-developers` will return the list of developers in charge of the given package.
- When using the `-c` command line option, `get-developers` will look at all files under version control in the Buildroot repository, and list the ones that are not handled by any developer. The purpose of this option is to help completing the `DEVELOPERS` file.
- When using the `-v` command line option, it validates the integrity of the DEVELOPERS file and will note WARNINGS for items that don’t match.

## Chapter 24. Release Engineering

## 24.1. Releases

The Buildroot project makes quarterly releases with monthly bugfix releases. The first release of each year is a long term support release, LTS.

- Quarterly releases: 2020.02, 2020.05, 2020.08, and 2020.11
- Bugfix releases: 2020.02.1, 2020.02.2, …
- LTS releases: 2020.02, 2021.02, …

Releases are supported until the first bugfix release of the next release, e.g., 2020.05.x is EOL when 2020.08.1 is released.

LTS releases are supported until the first bugfix release of the next LTS, e.g., 2020.02.x is supported until 2021.02.1 is released.

## 24.2. Development

Each release cycle consist of two months of development on the `master` branch and one month stabilization before the release is made. During this phase no new features are added to `master`, only bugfixes.

The stabilization phase starts with tagging `-rc1`, and every week until the release, another release candidate is tagged.

To handle new features and version bumps during the stabilization phase, a `next` branch may be created for these features. Once the current release has been made, the `next` branch is merged into `master` and the development cycle for the next release continues there.

# Part IV. Appendix

## Chapter 25. Makedev syntax documentation

The makedev syntax is used in several places in Buildroot to define changes to be made for permissions, or which device files to create and how to create them, in order to avoid calls to mknod.

This syntax is derived from the makedev utility, and more complete documentation can be found in the `package/makedevs/README` file.

It takes the form of a space separated list of fields, one file per line; the fields are:

| name | type | mode | uid  | gid  | major | minor | start | inc  | count |
| ---- | ---- | ---- | ---- | ---- | ----- | ----- | ----- | ---- | ----- |
|      |      |      |      |      |       |       |       |      |       |

There are a few non-trivial blocks:

- `name` is the path to the file you want to create/modify
- `type` is the type of the file, being one of:
  - `f`: a regular file, which must already exist
  - `F`: a regular file, which is ignored and not created if missing
  - `d`: a directory, which is created, as well as its parents, if missing
  - `r`: a directory recursively, which must already exist
  - `c`: a character device file, which parent directory must exist
  - `b`: a block device file, which parent directory must exist
  - `p`: a named pipe, which parent directory must exist
- `mode` are the usual permissions settings (only numerical values are allowed); for type `d`, the mode of existing parents is not changed, but the mode of created parents is set; for types `f`, `F`, and `r`, `mode` can also be set to `-1` to not change the mode (and only change uid and gid)
- `uid` and `gid` are the UID and GID to set on this file; can be either numerical values or actual names
- `major` and `minor` are here for device files, set to `-` for other files
- `start`, `inc` and `count` are for when you want to create a batch of files, and can be reduced to a loop, beginning at `start`, incrementing its counter by `inc` until it reaches `count`

Let’s say you want to change the ownership and permissions of a given file; using this syntax, you will need to write:

```
/usr/bin/foo f 755 0 0 - - - - -
/usr/bin/bar f 755 root root - - - - -
/data/buz f 644 buz-user buz-group - - - - -
/data/baz f -1 baz-user baz-group - - - - -
```

Alternatively, if you want to change owner of a directory recursively, you can write (to set UID to `foo` and GID to `bar` for the directory `/usr/share/myapp` and all files and directories below it):

```
/usr/share/myapp r -1 foo bar - - - - -
```

On the other hand, if you want to create the device file `/dev/hda` and the corresponding 15 files for the partitions, you will need for `/dev/hda`:

```
/dev/hda b 640 root root 3 0 0 0 -
```

and then for device files corresponding to the partitions of `/dev/hda`, `/dev/hdaX`, `X` ranging from 1 to 15:

```
/dev/hda b 640 root root 3 1 1 1 15
```

Extended attributes are supported if `BR2_ROOTFS_DEVICE_TABLE_SUPPORTS_EXTENDED_ATTRIBUTES` is enabled. This is done by adding a line starting with `|xattr` after the line describing the file. Right now, only capability is supported as extended attribute.

| \|xattr | capability |
| ------- | ---------- |
|         |            |

- `|xattr` is a "flag" that indicate an extended attribute
- `capability` is a capability to add to the previous file

If you want to add the capability cap_sys_admin to the binary foo, you will write :

```
/usr/bin/foo f 755 root root - - - - -
|xattr cap_sys_admin+eip
```

You can add several capabilities to a file by using several `|xattr` lines. If you want to add the capability cap_sys_admin and cap_net_admin to the binary foo, you will write :

```
/usr/bin/foo f 755 root root - - - - -
|xattr cap_sys_admin+eip
|xattr cap_net_admin+eip
```
