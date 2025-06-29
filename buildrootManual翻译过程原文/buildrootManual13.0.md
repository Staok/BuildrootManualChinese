## 18.10. Infrastructure for LuaRocks-based packages

### 18.10.1. `luarocks-package` tutorial

First, let’s see how to write a `.mk` file for a LuaRocks-based package, with an example :

```
01: ################################################################################
02: #
03: # lua-foo
04: #
05: ################################################################################
06:
07: LUA_FOO_VERSION = 1.0.2-1
08: LUA_FOO_NAME_UPSTREAM = foo
09: LUA_FOO_DEPENDENCIES = bar
10:
11: LUA_FOO_BUILD_OPTS += BAR_INCDIR=$(STAGING_DIR)/usr/include
12: LUA_FOO_BUILD_OPTS += BAR_LIBDIR=$(STAGING_DIR)/usr/lib
13: LUA_FOO_LICENSE = luaFoo license
14: LUA_FOO_LICENSE_FILES = $(LUA_FOO_SUBDIR)/COPYING
15:
16: $(eval $(luarocks-package))
```

On line 7, we declare the version of the package (the same as in the rockspec, which is the concatenation of the upstream version and the rockspec revision, separated by a hyphen *-*).

On line 8, we declare that the package is called "foo" on LuaRocks. In Buildroot, we give Lua-related packages a name that starts with "lua", so the Buildroot name is different from the upstream name. `LUA_FOO_NAME_UPSTREAM` makes the link between the two names.

On line 9, we declare our dependencies against native libraries, so that they are built before the build process of our package starts.

On lines 11-12, we tell Buildroot to pass custom options to LuaRocks when it is building the package.

On lines 13-14, we specify the licensing terms for the package.

Finally, on line 16, we invoke the `luarocks-package` macro that generates all the Makefile rules that actually allows the package to be built.

Most of these details can be retrieved from the `rock` and `rockspec`. So, this file and the Config.in file can be generated by running the command `luarocks buildroot foo lua-foo` in the Buildroot directory. This command runs a specific Buildroot addon of `luarocks` that will automatically generate a Buildroot package. The result must still be manually inspected and possibly modified.

- The `package/Config.in` file has to be updated manually to include the generated Config.in files.

### 18.10.2. `luarocks-package` reference

LuaRocks is a deployment and management system for Lua modules, and supports various `build.type`: `builtin`, `make` and `cmake`. In the context of Buildroot, the `luarocks-package` infrastructure only supports the `builtin` mode. LuaRocks packages that use the `make` or `cmake` build mechanisms should instead be packaged using the `generic-package` and `cmake-package` infrastructures in Buildroot, respectively.

The main macro of the LuaRocks package infrastructure is `luarocks-package`: like `generic-package` it works by defining a number of variables providing metadata information about the package, and then calling the `luarocks-package` macro.

Just like the generic infrastructure, the LuaRocks infrastructure works by defining a number of variables before calling the `luarocks-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the LuaRocks infrastructure.

Two of them are populated by the LuaRocks infrastructure (for the `download` step). If your package is not hosted on the LuaRocks mirror `$(BR2_LUAROCKS_MIRROR)`, you can override them:

- `LUA_FOO_SITE`, which defaults to `$(BR2_LUAROCKS_MIRROR)`
- `LUA_FOO_SOURCE`, which defaults to `$(lowercase LUA_FOO_NAME_UPSTREAM)-$(LUA_FOO_VERSION).src.rock`

A few additional variables, specific to the LuaRocks infrastructure, are also defined. They can be overridden in specific cases.

- `LUA_FOO_NAME_UPSTREAM`, which defaults to `lua-foo`, i.e. the Buildroot package name
- `LUA_FOO_ROCKSPEC`, which defaults to `$(lowercase LUA_FOO_NAME_UPSTREAM)-$(LUA_FOO_VERSION).rockspec`
- `LUA_FOO_SUBDIR`, which defaults to `$(LUA_FOO_NAME_UPSTREAM)-$(LUA_FOO_VERSION_WITHOUT_ROCKSPEC_REVISION)`
- `LUA_FOO_BUILD_OPTS` contains additional build options for the `luarocks build` call.

## 18.11. Infrastructure for Perl/CPAN packages

### 18.11.1. `perl-package` tutorial

First, let’s see how to write a `.mk` file for a Perl/CPAN package, with an example :

```
01: ################################################################################
02: #
03: # perl-foo-bar
04: #
05: ################################################################################
06:
07: PERL_FOO_BAR_VERSION = 0.02
08: PERL_FOO_BAR_SOURCE = Foo-Bar-$(PERL_FOO_BAR_VERSION).tar.gz
09: PERL_FOO_BAR_SITE = $(BR2_CPAN_MIRROR)/authors/id/M/MO/MONGER
10: PERL_FOO_BAR_DEPENDENCIES = perl-strictures
11: PERL_FOO_BAR_LICENSE = Artistic or GPL-1.0+
12: PERL_FOO_BAR_LICENSE_FILES = LICENSE
13: PERL_FOO_BAR_DISTNAME = Foo-Bar
14:
15: $(eval $(perl-package))
```

On line 7, we declare the version of the package.

On line 8 and 9, we declare the name of the tarball and the location of the tarball on a CPAN server. Buildroot will automatically download the tarball from this location.

On line 10, we declare our dependencies, so that they are built before the build process of our package starts.

On line 11 and 12, we give licensing details about the package (its license on line 11, and the file containing the license text on line 12).

On line 13, the name of the distribution as needed by the script `utils/scancpan` (in order to regenerate/upgrade these package files).

Finally, on line 15, we invoke the `perl-package` macro that generates all the Makefile rules that actually allow the package to be built.

Most of these data can be retrieved from https://metacpan.org/. So, this file and the Config.in can be generated by running the script `utils/scancpan Foo-Bar` in the Buildroot directory (or in a br2-external tree). This script creates a Config.in file and foo-bar.mk file for the requested package, and also recursively for all dependencies specified by CPAN. You should still manually edit the result. In particular, the following things should be checked.

- If the perl module links with a shared library that is provided by another (non-perl) package, this dependency is not added automatically. It has to be added manually to `PERL_FOO_BAR_DEPENDENCIES`.
- The `package/Config.in` file has to be updated manually to include the generated Config.in files. As a hint, the `scancpan` script prints out the required `source "…"` statements, sorted alphabetically.

### 18.11.2. `perl-package` reference

As a policy, packages that provide Perl/CPAN modules should all be named `perl-<something>` in Buildroot.

This infrastructure handles various Perl build systems : `ExtUtils-MakeMaker` (EUMM), `Module-Build` (MB) and `Module-Build-Tiny`. `Build.PL` is preferred by default when a package provides a `Makefile.PL` and a `Build.PL`.

The main macro of the Perl/CPAN package infrastructure is `perl-package`. It is similar to the `generic-package` macro. The ability to have target and host packages is also available, with the `host-perl-package` macro.

Just like the generic infrastructure, the Perl/CPAN infrastructure works by defining a number of variables before calling the `perl-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the Perl/CPAN infrastructure.

Note that setting `PERL_FOO_INSTALL_STAGING` to `YES` has no effect unless a `PERL_FOO_INSTALL_STAGING_CMDS` variable is defined. The perl infrastructure doesn’t define these commands since Perl modules generally don’t need to be installed to the `staging` directory.

A few additional variables, specific to the Perl/CPAN infrastructure, can also be defined. Many of them are only useful in very specific cases, typical packages will therefore only use a few of them.

- `PERL_FOO_PREFER_INSTALLER`/`HOST_PERL_FOO_PREFER_INSTALLER`, specifies the preferred installation method. Possible values are `EUMM` (for `Makefile.PL` based installation using `ExtUtils-MakeMaker`) and `MB` (for `Build.PL` based installation using `Module-Build`). This variable is only used when the package provides both installation methods.
- `PERL_FOO_CONF_ENV`/`HOST_PERL_FOO_CONF_ENV`, to specify additional environment variables to pass to the `perl Makefile.PL` or `perl Build.PL`. By default, empty.
- `PERL_FOO_CONF_OPTS`/`HOST_PERL_FOO_CONF_OPTS`, to specify additional configure options to pass to the `perl Makefile.PL` or `perl Build.PL`. By default, empty.
- `PERL_FOO_BUILD_OPTS`/`HOST_PERL_FOO_BUILD_OPTS`, to specify additional options to pass to `make pure_all` or `perl Build build` in the build step. By default, empty.
- `PERL_FOO_INSTALL_TARGET_OPTS`, to specify additional options to pass to `make pure_install` or `perl Build install` in the install step. By default, empty.
- `HOST_PERL_FOO_INSTALL_OPTS`, to specify additional options to pass to `make pure_install` or `perl Build install` in the install step. By default, empty.
