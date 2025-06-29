## 9.2. Keeping customizations outside of Buildroot

As already briefly mentioned in [Section 9.1, “Recommended directory structure”](https://buildroot.org/downloads/manual/manual.html#customize-dir-structure), you can place project-specific customizations in two locations:

- directly within the Buildroot tree, typically maintaining them using branches in a version control system so that upgrading to a newer Buildroot release is easy.
- outside of the Buildroot tree, using the *br2-external* mechanism. This mechanism allows to keep package recipes, board support and configuration files outside of the Buildroot tree, while still having them nicely integrated in the build logic. We call this location a *br2-external tree*. This section explains how to use the br2-external mechanism and what to provide in a br2-external tree.

One can tell Buildroot to use one or more br2-external trees by setting the `BR2_EXTERNAL` make variable set to the path(s) of the br2-external tree(s) to use. It can be passed to any Buildroot `make` invocation. It is automatically saved in the hidden `.br2-external.mk` file in the output directory. Thanks to this, there is no need to pass `BR2_EXTERNAL` at every `make` invocation. It can however be changed at any time by passing a new value, and can be removed by passing an empty value.

**Note.** The path to a br2-external tree can be either absolute or relative. If it is passed as a relative path, it is important to note that it is interpreted relative to the main Buildroot source directory, ***\*not\**** to the Buildroot output directory.

**Note:** If using an br2-external tree from before Buildroot 2016.11, you need to convert it before you can use it with Buildroot 2016.11 onward. See [Section 27.2, “Migrating to 2016.11”](https://buildroot.org/downloads/manual/manual.html#br2-external-converting) for help on doing so.

Some examples:

```
buildroot/ $ make BR2_EXTERNAL=/path/to/foo menuconfig
```

From now on, definitions from the `/path/to/foo` br2-external tree will be used:

```
buildroot/ $ make
buildroot/ $ make legal-info
```

We can switch to another br2-external tree at any time:

```
buildroot/ $ make BR2_EXTERNAL=/where/we/have/bar xconfig
```

We can also use multiple br2-external trees:

```
buildroot/ $ make BR2_EXTERNAL=/path/to/foo:/where/we/have/bar menuconfig
```

Or disable the usage of any br2-external tree:

```
buildroot/ $ make BR2_EXTERNAL= xconfig
```

### 9.2.1. Layout of a br2-external tree

A br2-external tree must contain at least those three files, described in the following chapters:

- `external.desc`
- `external.mk`
- `Config.in`

Apart from those mandatory files, there may be additional and optional content that may be present in a br2-external tree, like the `configs/` or `provides/` directories. They are described in the following chapters as well.

A complete example br2-external tree layout is also described later.

#### The `external.desc` file

That file describes the br2-external tree: the *name* and *description* for that br2-external tree.

The format for this file is line based, with each line starting by a keyword, followed by a colon and one or more spaces, followed by the value assigned to that keyword. There are two keywords currently recognised:

- `name`, mandatory, defines the name for that br2-external tree. That name must only use ASCII characters in the set `[A-Za-z0-9_]`; any other character is forbidden. Buildroot sets the variable `BR2_EXTERNAL_$(NAME)_PATH` to the absolute path of the br2-external tree, so that you can use it to refer to your br2-external tree. This variable is available both in Kconfig, so you can use it to source your Kconfig files (see below) and in the Makefile, so that you can use it to include other Makefiles (see below) or refer to other files (like data files) from your br2-external tree.

  **Note:** Since it is possible to use multiple br2-external trees at once, this name is used by Buildroot to generate variables for each of those trees. That name is used to identify your br2-external tree, so try to come up with a name that really describes your br2-external tree, in order for it to be relatively unique, so that it does not clash with another name from another br2-external tree, especially if you are planning on somehow sharing your br2-external tree with third parties or using br2-external trees from third parties.

- `desc`, optional, provides a short description for that br2-external tree. It shall fit on a single line, is mostly free-form (see below), and is used when displaying information about a br2-external tree (e.g. above the list of defconfig files, or as the prompt in the menuconfig); as such, it should relatively brief (40 chars is probably a good upper limit). The description is available in the `BR2_EXTERNAL_$(NAME)_DESC` variable.

Examples of names and the corresponding `BR2_EXTERNAL_$(NAME)_PATH` variables:

- `FOO` → `BR2_EXTERNAL_FOO_PATH`
- `BAR_42` → `BR2_EXTERNAL_BAR_42_PATH`

In the following examples, it is assumed the name to be set to `BAR_42`.

**Note:** Both `BR2_EXTERNAL_$(NAME)_PATH` and `BR2_EXTERNAL_$(NAME)_DESC` are available in the Kconfig files and the Makefiles. They are also exported in the environment so are available in post-build, post-image and in-fakeroot scripts.

#### The `Config.in` and `external.mk` files

Those files (which may each be empty) can be used to define package recipes (i.e. `foo/Config.in` and `foo/foo.mk` like for packages bundled in Buildroot itself) or other custom configuration options or make logic.

Buildroot automatically includes the `Config.in` from each br2-external tree to make it appear in the top-level configuration menu, and includes the `external.mk` from each br2-external tree with the rest of the makefile logic.

The main usage of this is to store package recipes. The recommended way to do this is to write a `Config.in` file that looks like:

```
source "$BR2_EXTERNAL_BAR_42_PATH/package/package1/Config.in"
source "$BR2_EXTERNAL_BAR_42_PATH/package/package2/Config.in"
```

Then, have an `external.mk` file that looks like:

```
include $(sort $(wildcard $(BR2_EXTERNAL_BAR_42_PATH)/package/*/*.mk))
```

And then in `$(BR2_EXTERNAL_BAR_42_PATH)/package/package1` and `$(BR2_EXTERNAL_BAR_42_PATH)/package/package2` create normal Buildroot package recipes, as explained in [Chapter 18, *Adding new packages to Buildroot*](https://buildroot.org/downloads/manual/manual.html#adding-packages). If you prefer, you can also group the packages in subdirectories called <boardname> and adapt the above paths accordingly.

You can also define custom configuration options in `Config.in` and custom make logic in `external.mk`.

#### The `configs/` directory

One can store Buildroot defconfigs in the `configs` subdirectory of the br2-external tree. Buildroot will automatically show them in the output of `make list-defconfigs` and allow them to be loaded with the normal `make <name>_defconfig` command. They will be visible in the *make list-defconfigs* output, below an `External configs` label that contains the name of the br2-external tree they are defined in.

**Note:** If a defconfig file is present in more than one br2-external tree, then the one from the last br2-external tree is used. It is thus possible to override a defconfig bundled in Buildroot or another br2-external tree.



