### 18.25.3. How to test your package

Once you have added your new package, it is important that you test it under various conditions: does it build for all architectures? Does it build with the different C libraries? Does it need threads, NPTL? And so on…

Buildroot runs [autobuilders](http://autobuild.buildroot.org/) which continuously test random configurations. However, these only build the `master` branch of the git tree, and your new fancy package is not yet there.

Buildroot provides a script in `utils/test-pkg` that uses the same base configurations as used by the autobuilders so you can test your package in the same conditions.

First, create a config snippet that contains all the necessary options needed to enable your package, but without any architecture or toolchain option. For example, let’s create a config snippet that just enables `libcurl`, without any TLS backend:

```
$ cat libcurl.config
BR2_PACKAGE_LIBCURL=y
```

If your package needs more configuration options, you can add them to the config snippet. For example, here’s how you would test `libcurl` with `openssl` as a TLS backend and the `curl` program:

```
$ cat libcurl.config
BR2_PACKAGE_LIBCURL=y
BR2_PACKAGE_LIBCURL_CURL=y
BR2_PACKAGE_OPENSSL=y
```

Then run the `test-pkg` script, by telling it what config snippet to use and what package to test:

```
$ ./utils/test-pkg -c libcurl.config -p libcurl
```

By default, `test-pkg` will build your package against a subset of the toolchains used by the autobuilders, which has been selected by the Buildroot developers as being the most useful and representative subset. If you want to test all toolchains, pass the `-a` option. Note that in any case, internal toolchains are excluded as they take too long to build.

The output lists all toolchains that are tested and the corresponding result (excerpt, results are fake):

```
$ ./utils/test-pkg -c libcurl.config -p libcurl
                armv5-ctng-linux-gnueabi [ 1/11]: OK
              armv7-ctng-linux-gnueabihf [ 2/11]: OK
                        br-aarch64-glibc [ 3/11]: SKIPPED
                           br-arcle-hs38 [ 4/11]: SKIPPED
                            br-arm-basic [ 5/11]: FAILED
                  br-arm-cortex-a9-glibc [ 6/11]: OK
                   br-arm-cortex-a9-musl [ 7/11]: FAILED
                   br-arm-cortex-m4-full [ 8/11]: OK
                             br-arm-full [ 9/11]: OK
                    br-arm-full-nothread [10/11]: FAILED
                      br-arm-full-static [11/11]: OK
11 builds, 2 skipped, 2 build failed, 1 legal-info failed
```

The results mean:

- `OK`: the build was successful.
- `SKIPPED`: one or more configuration options listed in the config snippet were not present in the final configuration. This is due to options having dependencies not satisfied by the toolchain, such as for example a package that `depends on BR2_USE_MMU` with a noMMU toolchain. The missing options are reported in `missing.config` in the output build directory (`~/br-test-pkg/TOOLCHAIN_NAME/` by default).
- `FAILED`: the build failed. Inspect the `logfile` file in the output build directory to see what went wrong:
  - the actual build failed,
  - the legal-info failed,
  - one of the preliminary steps (downloading the config file, applying the configuration, running `dirclean` for the package) failed.

When there are failures, you can just re-run the script with the same options (after you fixed your package); the script will attempt to re-build the package specified with `-p` for all toolchains, without the need to re-build all the dependencies of that package.

The `test-pkg` script accepts a few options, for which you can get some help by running:

```
$ ./utils/test-pkg -h
```
