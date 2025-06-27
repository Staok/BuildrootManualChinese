Buildroot 2025.05 manual generated on 2025-06-09 20:23:58 UTC from git revision fcde5363aa

The Buildroot manual is written by the Buildroot developers. It is licensed under the GNU General Public License, version 2. Refer to the [COPYING](http://git.buildroot.org/buildroot/tree/COPYING?id=fcde5363aa35220a1f201159a05de652ec6f811f) file in the Buildroot sources for the full text of this license.

Copyright © The Buildroot developers <[buildroot@buildroot.org](mailto:buildroot@buildroot.org)>

# Part I. Getting started

## Chapter 1. About Buildroot

Buildroot is a tool that simplifies and automates the process of building a complete Linux system for an embedded system, using cross-compilation.

In order to achieve this, Buildroot is able to generate a cross-compilation toolchain, a root filesystem, a Linux kernel image and a bootloader for your target. Buildroot can be used for any combination of these options, independently (you can for example use an existing cross-compilation toolchain, and build only your root filesystem with Buildroot).

Buildroot is useful mainly for people working with embedded systems. Embedded systems often use processors that are not the regular x86 processors everyone is used to having in his PC. They can be PowerPC processors, MIPS processors, ARM processors, etc.

Buildroot supports numerous processors and their variants; it also comes with default configurations for several boards available off-the-shelf. Besides this, a number of third-party projects are based on, or develop their BSP [[1\]](https://buildroot.org/downloads/manual/manual.html#ftn.idm25) or SDK [[2\]](https://buildroot.org/downloads/manual/manual.html#ftn.idm27) on top of Buildroot.

------

[[1\] ](https://buildroot.org/downloads/manual/manual.html#idm25)BSP: Board Support Package

[[2\] ](https://buildroot.org/downloads/manual/manual.html#idm27)SDK: Software Development Kit

## Chapter 2. System requirements

Buildroot is designed to run on Linux systems.

While Buildroot itself will build most host packages it needs for the compilation, certain standard Linux utilities are expected to be already installed on the host system. Below you will find an overview of the mandatory and optional packages (note that package names may vary between distributions).

## 2.1. Mandatory packages

- Build tools:
  - `which`
  - `sed`
  - `make` (version 3.81 or any later)
  - `binutils`
  - `build-essential` (only for Debian based systems)
  - `diffutils`
  - `gcc` (version 4.8 or any later)
  - `g++` (version 4.8 or any later)
  - `bash`
  - `patch`
  - `gzip`
  - `bzip2`
  - `perl` (version 5.8.7 or any later)
  - `tar`
  - `cpio`
  - `unzip`
  - `rsync`
  - `file` (must be in `/usr/bin/file`)
  - `bc`
  - `findutils`
  - `awk`
- Source fetching tools:
  - `wget`

## 2.2. Optional packages

- Recommended dependencies:

  Some features or utilities in Buildroot, like the legal-info, or the graph generation tools, have additional dependencies. Although they are not mandatory for a simple build, they are still highly recommended:

  - `python` (version 2.7 or any later)

- Configuration interface dependencies:

  For these libraries, you need to install both runtime and development data, which in many distributions are packaged separately. The development packages typically have a *-dev* or *-devel* suffix.

  - `ncurses5` to use the *menuconfig* interface
  - `qt5` to use the *xconfig* interface
  - `glib2`, `gtk2` and `glade2` to use the *gconfig* interface

- Source fetching tools:

  In the official tree, most of the package sources are retrieved using `wget` from *ftp*, *http* or *https* locations. A few packages are only available through a version control system. Moreover, Buildroot is capable of downloading sources via other tools, like `git` or `scp` (refer to [Chapter 20, *Download infrastructure*](https://buildroot.org/downloads/manual/manual.html#download-infra) for more details). If you enable packages using any of these methods, you will need to install the corresponding tool on the host system:

  - `bazaar`
  - `curl`
  - `cvs`
  - `git`
  - `mercurial`
  - `scp`
  - `sftp`
  - `subversion`

- Java-related packages, if the Java Classpath needs to be built for the target system:

  - The `javac` compiler
  - The `jar` tool

- Documentation generation tools:

  - `asciidoc`, version 8.6.3 or higher
  - `w3m`
  - `python` with the `argparse` module (automatically present in 2.7+ and 3.2+)
  - `dblatex` (required for the pdf manual only)

- Graph generation tools:

  - `graphviz` to use *graph-depends* and *<pkg>-graph-depends*
  - `python-matplotlib` to use *graph-build*

- Package statistics tools (*pkg-stats*):

  - `python-aiohttp`

## Chapter 3. Getting Buildroot

Buildroot releases are made every 3 months, in February, May, August and November. Release numbers are in the format YYYY.MM, so for example 2013.02, 2014.08.

Release tarballs are available at http://buildroot.org/downloads/.

For your convenience, a [Vagrantfile](https://www.vagrantup.com/) is available in `support/misc/Vagrantfile` in the Buildroot source tree to quickly set up a virtual machine with the needed dependencies to get started.

If you want to setup an isolated buildroot environment on Linux or Mac Os X, paste this line onto your terminal:

```
curl -O https://buildroot.org/downloads/Vagrantfile; vagrant up
```

If you are on Windows, paste this into your powershell:

```
(new-object System.Net.WebClient).DownloadFile(
"https://buildroot.org/downloads/Vagrantfile","Vagrantfile");
vagrant up
```

If you want to follow development, you can use the daily snapshots or make a clone of the Git repository. Refer to the [Download page](http://buildroot.org/download) of the Buildroot website for more details.
