## 18.14. Infrastructure for rebar-based packages

### 18.14.1. `rebar-package` tutorial

First, let’s see how to write a `.mk` file for a rebar-based package, with an example :

```
01: ################################################################################
02: #
03: # erlang-foobar
04: #
05: ################################################################################
06:
07: ERLANG_FOOBAR_VERSION = 1.0
08: ERLANG_FOOBAR_SOURCE = erlang-foobar-$(ERLANG_FOOBAR_VERSION).tar.xz
09: ERLANG_FOOBAR_SITE = http://www.foosoftware.org/download
10: ERLANG_FOOBAR_DEPENDENCIES = host-libaaa libbbb
11:
12: $(eval $(rebar-package))
```

On line 7, we declare the version of the package.

On line 8 and 9, we declare the name of the tarball (xz-ed tarball recommended) and the location of the tarball on the Web. Buildroot will automatically download the tarball from this location.

On line 10, we declare our dependencies, so that they are built before the build process of our package starts.

Finally, on line 12, we invoke the `rebar-package` macro that generates all the Makefile rules that actually allows the package to be built.

### 18.14.2. `rebar-package` reference

The main macro of the `rebar` package infrastructure is `rebar-package`. It is similar to the `generic-package` macro. The ability to have host packages is also available, with the `host-rebar-package` macro.

Just like the generic infrastructure, the `rebar` infrastructure works by defining a number of variables before calling the `rebar-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the `rebar` infrastructure.

A few additional variables, specific to the `rebar` infrastructure, can also be defined. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them.

- `ERLANG_FOOBAR_USE_AUTOCONF`, to specify that the package uses *autoconf* at the configuration step. When a package sets this variable to `YES`, the `autotools` infrastructure is used.

  **Note.** You can also use some of the variables from the `autotools` infrastructure: `ERLANG_FOOBAR_CONF_ENV`, `ERLANG_FOOBAR_CONF_OPTS`, `ERLANG_FOOBAR_AUTORECONF`, `ERLANG_FOOBAR_AUTORECONF_ENV` and `ERLANG_FOOBAR_AUTORECONF_OPTS`.

- `ERLANG_FOOBAR_USE_BUNDLED_REBAR`, to specify that the package has a bundled version of *rebar* ***\*and\**** that it shall be used. Valid values are `YES` or `NO` (the default).

  **Note.** If the package bundles a *rebar* utility, but can use the generic one that Buildroot provides, just say `NO` (i.e., do not specify this variable). Only set if it is mandatory to use the *rebar* utility bundled in this package.

- `ERLANG_FOOBAR_REBAR_ENV`, to specify additional environment variables to pass to the *rebar* utility.

- `ERLANG_FOOBAR_KEEP_DEPENDENCIES`, to keep the dependencies described in the rebar.config file. Valid values are `YES` or `NO` (the default). Unless this variable is set to `YES`, the *rebar* infrastructure removes such dependencies in a post-patch hook to ensure rebar does not download nor compile them.

With the rebar infrastructure, all the steps required to build and install the packages are already defined, and they generally work well for most rebar-based packages. However, when required, it is still possible to customize what is done in any particular step:

- By adding a post-operation hook (after extract, patch, configure, build or install). See [Section 18.23, “Hooks available in the various build steps”](https://buildroot.org/downloads/manual/manual.html#hooks) for details.
- By overriding one of the steps. For example, even if the rebar infrastructure is used, if the package `.mk` file defines its own `ERLANG_FOOBAR_BUILD_CMDS` variable, it will be used instead of the default rebar one. However, using this method should be restricted to very specific cases. Do not use it in the general case.

## 18.15. Infrastructure for Waf-based packages

### 18.15.1. `waf-package` tutorial

First, let’s see how to write a `.mk` file for a Waf-based package, with an example :

```
01: ################################################################################
02: #
03: # libfoo
04: #
05: ################################################################################
06:
07: LIBFOO_VERSION = 1.0
08: LIBFOO_SOURCE = libfoo-$(LIBFOO_VERSION).tar.gz
09: LIBFOO_SITE = http://www.foosoftware.org/download
10: LIBFOO_CONF_OPTS = --enable-bar --disable-baz
11: LIBFOO_DEPENDENCIES = bar
12:
13: $(eval $(waf-package))
```

On line 7, we declare the version of the package.

On line 8 and 9, we declare the name of the tarball (xz-ed tarball recommended) and the location of the tarball on the Web. Buildroot will automatically download the tarball from this location.

On line 10, we tell Buildroot what options to enable for libfoo.

On line 11, we tell Buildroot the dependencies of libfoo.

Finally, on line line 13, we invoke the `waf-package` macro that generates all the Makefile rules that actually allows the package to be built.

### 18.15.2. `waf-package` reference

The main macro of the Waf package infrastructure is `waf-package`. It is similar to the `generic-package` macro.

Just like the generic infrastructure, the Waf infrastructure works by defining a number of variables before calling the `waf-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the Waf infrastructure.

A few additional variables, specific to the Waf infrastructure, can also be defined.

- `LIBFOO_SUBDIR` may contain the name of a subdirectory inside the package that contains the main wscript file. This is useful, if for example, the main wscript file is not at the root of the tree extracted by the tarball. If `HOST_LIBFOO_SUBDIR` is not specified, it defaults to `LIBFOO_SUBDIR`.
- `LIBFOO_NEEDS_EXTERNAL_WAF` can be set to `YES` or `NO` to tell Buildroot to use the bundled `waf` executable. If set to `NO`, the default, then Buildroot will use the waf executable provided in the package source tree; if set to `YES`, then Buildroot will download, install waf as a host tool and use it to build the package.
- `LIBFOO_WAF_OPTS`, to specify additional options to pass to the `waf` script at every step of the package build process: configure, build and installation. By default, empty.
- `LIBFOO_CONF_OPTS`, to specify additional options to pass to the `waf` script for the configuration step. By default, empty.
- `LIBFOO_BUILD_OPTS`, to specify additional options to pass to the `waf` script during the build step. By default, empty.
- `LIBFOO_INSTALL_STAGING_OPTS`, to specify additional options to pass to the `waf` script during the staging installation step. By default, empty.
- `LIBFOO_INSTALL_TARGET_OPTS`, to specify additional options to pass to the `waf` script during the target installation step. By default, empty.
