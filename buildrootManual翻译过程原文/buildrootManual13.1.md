## 18.12. Infrastructure for virtual packages

In Buildroot, a virtual package is a package whose functionalities are provided by one or more packages, referred to as *providers*. The virtual package management is an extensible mechanism allowing the user to choose the provider used in the rootfs.

For example, *OpenGL ES* is an API for 2D and 3D graphics on embedded systems. The implementation of this API is different for the *Allwinner Tech Sunxi* and the *Texas Instruments OMAP35xx* platforms. So `libgles` will be a virtual package and `sunxi-mali-utgard` and `ti-gfx` will be the providers.

### 18.12.1. `virtual-package` tutorial

In the following example, we will explain how to add a new virtual package (*something-virtual*) and a provider for it (*some-provider*).

First, let’s create the virtual package.

### 18.12.2. Virtual package’s `Config.in` file

The `Config.in` file of virtual package *something-virtual* should contain:

```
01: config BR2_PACKAGE_HAS_SOMETHING_VIRTUAL
02:     bool
03:
04: config BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL
05:     depends on BR2_PACKAGE_HAS_SOMETHING_VIRTUAL
06:     string
```

In this file, we declare two options, `BR2_PACKAGE_HAS_SOMETHING_VIRTUAL` and `BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL`, whose values will be used by the providers.

### 18.12.3. Virtual package’s `.mk` file

The `.mk` for the virtual package should just evaluate the `virtual-package` macro:

```
01: ################################################################################
02: #
03: # something-virtual
04: #
05: ################################################################################
06:
07: $(eval $(virtual-package))
```

The ability to have target and host packages is also available, with the `host-virtual-package` macro.

### 18.12.4. Provider’s `Config.in` file

When adding a package as a provider, only the `Config.in` file requires some modifications.

The `Config.in` file of the package *some-provider*, which provides the functionalities of *something-virtual*, should contain:

```
01: config BR2_PACKAGE_SOME_PROVIDER
02:     bool "some-provider"
03:     select BR2_PACKAGE_HAS_SOMETHING_VIRTUAL
04:     help
05:       This is a comment that explains what some-provider is.
06:
07:       http://foosoftware.org/some-provider/
08:
09: if BR2_PACKAGE_SOME_PROVIDER
10: config BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL
11:     default "some-provider"
12: endif
```

On line 3, we select `BR2_PACKAGE_HAS_SOMETHING_VIRTUAL`, and on line 11, we set the value of `BR2_PACKAGE_PROVIDES_SOMETHING_VIRTUAL` to the name of the provider, but only if it is selected.

### 18.12.5. Provider’s `.mk` file

The `.mk` file should also declare an additional variable `SOME_PROVIDER_PROVIDES` to contain the names of all the virtual packages it is an implementation of:

```
01: SOME_PROVIDER_PROVIDES = something-virtual
```

Of course, do not forget to add the proper build and runtime dependencies for this package!

### 18.12.6. Notes on depending on a virtual package

When adding a package that requires a certain `FEATURE` provided by a virtual package, you have to use `depends on BR2_PACKAGE_HAS_FEATURE`, like so:

```
config BR2_PACKAGE_HAS_FEATURE
    bool

config BR2_PACKAGE_FOO
    bool "foo"
    depends on BR2_PACKAGE_HAS_FEATURE
```

### 18.12.7. Notes on depending on a specific provider

If your package really requires a specific provider, then you’ll have to make your package `depends on` this provider; you can *not* `select` a provider.

Let’s take an example with two providers for a `FEATURE`:

```
config BR2_PACKAGE_HAS_FEATURE
    bool

config BR2_PACKAGE_FOO
    bool "foo"
    select BR2_PACKAGE_HAS_FEATURE

config BR2_PACKAGE_BAR
    bool "bar"
    select BR2_PACKAGE_HAS_FEATURE
```

And you are adding a package that needs `FEATURE` as provided by `foo`, but not as provided by `bar`.

If you were to use `select BR2_PACKAGE_FOO`, then the user would still be able to select `BR2_PACKAGE_BAR` in the menuconfig. This would create a configuration inconsistency, whereby two providers of the same `FEATURE` would be enabled at once, one explicitly set by the user, the other implicitly by your `select`.

Instead, you have to use `depends on BR2_PACKAGE_FOO`, which avoids any implicit configuration inconsistency.

## 18.13. Infrastructure for packages using kconfig for configuration files

A popular way for a software package to handle user-specified configuration is `kconfig`. Among others, it is used by the Linux kernel, Busybox, and Buildroot itself. The presence of a .config file and a `menuconfig` target are two well-known symptoms of kconfig being used.

Buildroot features an infrastructure for packages that use kconfig for their configuration. This infrastructure provides the necessary logic to expose the package’s `menuconfig` target as `foo-menuconfig` in Buildroot, and to handle the copying back and forth of the configuration file in a correct way.

The main macro of the kconfig package infrastructure is `kconfig-package`. It is similar to the `generic-package` macro.

Just like the generic infrastructure, the kconfig infrastructure works by defining a number of variables before calling the `kconfig-package` macro.

All the package metadata information variables that exist in the [generic package infrastructure](https://buildroot.org/downloads/manual/manual.html#generic-package-reference) also exist in the kconfig infrastructure.

In order to use the `kconfig-package` infrastructure for a Buildroot package, the minimally required lines in the `.mk` file, in addition to the variables required by the `generic-package` infrastructure, are:

```
FOO_KCONFIG_FILE = reference-to-source-configuration-file

$(eval $(kconfig-package))
```

This snippet creates the following make targets:

- `foo-menuconfig`, which calls the package’s `menuconfig` target
- `foo-update-config`, which copies the configuration back to the source configuration file. It is not possible to use this target when fragment files are set.
- `foo-update-defconfig`, which copies the configuration back to the source configuration file. The configuration file will only list the options that differ from the default values. It is not possible to use this target when fragment files are set.
- `foo-diff-config`, which outputs the differences between the current configuration and the one defined in the Buildroot configuration for this kconfig package. The output is useful to identify the configuration changes that may have to be propagated to configuration fragments for example.

and ensures that the source configuration file is copied to the build directory at the right moment.

There are two options to specify a configuration file to use, either `FOO_KCONFIG_FILE` (as in the example, above) or `FOO_KCONFIG_DEFCONFIG`. It is mandatory to provide either, but not both:

- `FOO_KCONFIG_FILE` specifies the path to a defconfig or full-config file to be used to configure the package.
- `FOO_KCONFIG_DEFCONFIG` specifies the defconfig *make* rule to call to configure the package.

In addition to these minimally required lines, several optional variables can be set to suit the needs of the package under consideration:

- `FOO_KCONFIG_EDITORS`: a space-separated list of kconfig editors to support, for example *menuconfig xconfig*. By default, *menuconfig*.
- `FOO_KCONFIG_FRAGMENT_FILES`: a space-separated list of configuration fragment files that are merged to the main configuration file. Fragment files are typically used when there is a desire to stay in sync with an upstream (def)config file, with some minor modifications.
- `FOO_KCONFIG_OPTS`: extra options to pass when calling the kconfig editors. This may need to include *$(FOO_MAKE_OPTS)*, for example. By default, empty.
- `FOO_KCONFIG_FIXUP_CMDS`: a list of shell commands needed to fixup the configuration file after copying it or running a kconfig editor. Such commands may be needed to ensure a configuration consistent with other configuration of Buildroot, for example. By default, empty.
- `FOO_KCONFIG_DOTCONFIG`: path (with filename) of the `.config` file, relative to the package source tree. The default, `.config`, should be well suited for all packages that use the standard kconfig infrastructure as inherited from the Linux kernel; some packages use a derivative of kconfig that use a different location.
- `FOO_KCONFIG_DEPENDENCIES`: the list of packages (most probably, host packages) that need to be built before this package’s kconfig is interpreted. Seldom used. By default, empty.
- `FOO_KCONFIG_SUPPORTS_DEFCONFIG`: whether the package’s kconfig system supports using defconfig files; few packages do not. By default, *YES*.

