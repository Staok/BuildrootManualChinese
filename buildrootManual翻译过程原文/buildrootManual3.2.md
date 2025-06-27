## 8.5. Building out-of-tree

As default, everything built by Buildroot is stored in the directory `output` in the Buildroot tree.

Buildroot also supports building out of tree with a syntax similar to the Linux kernel. To use it, add `O=<directory>` to the make command line:

```
 $ make O=/tmp/build menuconfig
```

Or:

```
 $ cd /tmp/build; make O=$PWD -C path/to/buildroot menuconfig
```

All the output files will be located under `/tmp/build`. If the `O` path does not exist, Buildroot will create it.

***\*Note:\**** the `O` path can be either an absolute or a relative path, but if it’s passed as a relative path, it is important to note that it is interpreted relative to the main Buildroot source directory, ***\*not\**** the current working directory.

When using out-of-tree builds, the Buildroot `.config` and temporary files are also stored in the output directory. This means that you can safely run multiple builds in parallel using the same source tree as long as they use unique output directories.

For ease of use, Buildroot generates a Makefile wrapper in the output directory - so after the first run, you no longer need to pass `O=<…>` and `-C <…>`, simply run (in the output directory):

```
 $ make <target>
```

## 8.6. Environment variables

Buildroot also honors some environment variables, when they are passed to `make` or set in the environment:

- `HOSTCXX`, the host C++ compiler to use
- `HOSTCC`, the host C compiler to use
- `UCLIBC_CONFIG_FILE=<path/to/.config>`, path to the uClibc configuration file, used to compile uClibc, if an internal toolchain is being built. Note that the uClibc configuration file can also be set from the configuration interface, so through the Buildroot `.config` file; this is the recommended way of setting it.
- `BUSYBOX_CONFIG_FILE=<path/to/.config>`, path to the BusyBox configuration file. Note that the BusyBox configuration file can also be set from the configuration interface, so through the Buildroot `.config` file; this is the recommended way of setting it.
- `BR2_CCACHE_DIR` to override the directory where Buildroot stores the cached files when using ccache.
- `BR2_DL_DIR` to override the directory in which Buildroot stores/retrieves downloaded files. Note that the Buildroot download directory can also be set from the configuration interface, so through the Buildroot `.config` file. See [Section 8.13.4, “Location of downloaded packages”](https://buildroot.org/downloads/manual/manual.html#download-location) for more details on how you can set the download directory.
- `BR2_GRAPH_ALT`, if set and non-empty, to use an alternate color-scheme in build-time graphs
- `BR2_GRAPH_OUT` to set the filetype of generated graphs, either `pdf` (the default), or `png`.
- `BR2_GRAPH_DEPS_OPTS` to pass extra options to the dependency graph; see [Section 8.9, “Graphing the dependencies between packages”](https://buildroot.org/downloads/manual/manual.html#graph-depends) for the accepted options
- `BR2_GRAPH_DOT_OPTS` is passed verbatim as options to the `dot` utility to draw the dependency graph.
- `BR2_GRAPH_SIZE_OPTS` to pass extra options to the size graph; see [Section 8.11, “Graphing the filesystem size contribution of packages”](https://buildroot.org/downloads/manual/manual.html#graph-size) for the acepted options

An example that uses config files located in the toplevel directory and in your $HOME:

```
 $ make UCLIBC_CONFIG_FILE=uClibc.config BUSYBOX_CONFIG_FILE=$HOME/bb.config
```

If you want to use a compiler other than the default `gcc` or `g`++ for building helper-binaries on your host, then do

```
 $ make HOSTCXX=g++-4.3-HEAD HOSTCC=gcc-4.3-HEAD
```

## 8.7. Dealing efficiently with filesystem images

Filesystem images can get pretty big, depending on the filesystem you choose, the number of packages, whether you provisioned free space… Yet, some locations in the filesystems images may just be *empty* (e.g. a long run of *zeroes*); such a file is called a *sparse* file.

Most tools can handle sparse files efficiently, and will only store or write those parts of a sparse file that are not empty.

For example:

- `tar` accepts the `-S` option to tell it to only store non-zero blocks of sparse files:
  - `tar cf archive.tar -S [files…]` will efficiently store sparse files in a tarball
  - `tar xf archive.tar -S` will efficiently store sparse files extracted from a tarball
- `cp` accepts the `--sparse=WHEN` option (`WHEN` is one of `auto`, `never` or `always`):
  - `cp --sparse=always source.file dest.file` will make `dest.file` a sparse file if `source.file` has long runs of zeroes

Other tools may have similar options. Please consult their respective man pages.

You can use sparse files if you need to store the filesystem images (e.g. to transfer from one machine to another), or if you need to send them (e.g. to the Q&A team).

Note however that flashing a filesystem image to a device while using the sparse mode of `dd` may result in a broken filesystem (e.g. the block bitmap of an ext2 filesystem may be corrupted; or, if you have sparse files in your filesystem, those parts may not be all-zeroes when read back). You should only use sparse files when handling files on the build machine, not when transferring them to an actual device that will be used on the target.