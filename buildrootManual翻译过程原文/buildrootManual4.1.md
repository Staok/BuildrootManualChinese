## 8.11. Graphing the filesystem size contribution of packages

When your target system grows, it is sometimes useful to understand how much each Buildroot package is contributing to the overall root filesystem size. To help with such an analysis, Buildroot collects data about files installed by each package and using this data, generates a graph and CSV files detailing the size contribution of the different packages.

To generate these data after a build, run:

```
make graph-size
```

This will generate:

- `output/graphs/graph-size.pdf`, a pie chart of the contribution of each package to the overall root filesystem size
- `output/graphs/package-size-stats.csv`, a CSV file giving the size contribution of each package to the overall root filesystem size
- `output/graphs/file-size-stats.csv`, a CSV file giving the size contribution of each installed file to the package it belongs, and to the overall filesystem size.

This `graph-size` target requires the Python Matplotlib library to be installed (`python-matplotlib` on most distributions), and also the `argparse` module if you’re using a Python version older than 2.7 (`python-argparse` on most distributions).

Just like for the duration graph, a `BR2_GRAPH_OUT` environment variable is supported to adjust the output file format. See [Section 8.9, “Graphing the dependencies between packages”](https://buildroot.org/downloads/manual/manual.html#graph-depends) for details about this environment variable.

Additionally, one may set the environment variable `BR2_GRAPH_SIZE_OPTS` to further control the generated graph. Accepted options are:

- `--size-limit X`, `-l X`, will group all packages which individual contribution is below `X` percent, to a single entry labelled *Others* in the graph. By default, `X=0.01`, which means packages each contributing less than 1% are grouped under *Others*. Accepted values are in the range `[0.0..1.0]`.
- `--iec`, `--binary`, `--si`, `--decimal`, to use IEC (binary, powers of 1024) or SI (decimal, powers of 1000; the default) prefixes.
- `--biggest-first`, to sort packages in decreasing size order, rather than in increasing size order.

**Note.** The collected filesystem size data is only meaningful after a complete clean rebuild. Be sure to run `make clean all` before using `make graph-size`.

To compare the root filesystem size of two different Buildroot compilations, for example after adjusting the configuration or when switching to another Buildroot release, use the `size-stats-compare` script. It takes two `file-size-stats.csv` files (produced by `make graph-size`) as input. Refer to the help text of this script for more details:

```
utils/size-stats-compare -h
```

## 8.12. Top-level parallel build

**Note.** This section deals with a very experimental feature, which is known to break even in some non-unusual situations. Use at your own risk.

Buildroot has always been capable of using parallel build on a per package basis: each package is built by Buildroot using `make -jN` (or the equivalent invocation for non-make-based build systems). The level of parallelism is by default number of CPUs + 1, but it can be adjusted using the `BR2_JLEVEL` configuration option.

Until 2020.02, Buildroot was however building packages in a serial fashion: each package was built one after the other, without parallelization of the build between packages. As of 2020.02, Buildroot has experimental support for ***\*top-level parallel build\****, which allows some signicant build time savings by building packages that have no dependency relationship in parallel. This feature is however marked as experimental and is known not to work in some cases.

In order to use top-level parallel build, one must:

1. Enable the option `BR2_PER_PACKAGE_DIRECTORIES` in the Buildroot configuration
2. Use `make -jN` when starting the Buildroot build

Internally, the `BR2_PER_PACKAGE_DIRECTORIES` will enable a mechanism called ***\*per-package directories\****, which will have the following effects:

- Instead of a global *target* directory and a global *host* directory common to all packages, per-package *target* and *host* directories will be used, in `$(O)/per-package/<pkg>/target/` and `$(O)/per-package/<pkg>/host/` respectively. Those folders will be populated from the corresponding folders of the package dependencies at the beginning of `<pkg>` build. The compiler and all other tools will therefore only be able to see and access files installed by dependencies explicitly listed by `<pkg>`.
- At the end of the build, the global *target* and *host* directories will be populated, located in `$(O)/target` and `$(O)/host` respectively. This means that during the build, those folders will be empty and it’s only at the very end of the build that they will be populated.
