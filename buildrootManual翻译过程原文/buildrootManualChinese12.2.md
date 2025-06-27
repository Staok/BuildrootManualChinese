### 18.9.2. `python-package` 参考

作为政策，仅提供Python模块的软件包在Buildroot中应全部命名为 `python-<something>`。其他使用Python构建系统但不是Python模块的软件包可以自由命名（Buildroot中已有的例子有 `scons` 和 `supervisor`）。

Python软件包基础设施的主要宏是 `python-package`。它与 `generic-package` 宏类似。也可以通过 `host-python-package` 宏创建Python主机软件包。

与通用基础设施类似，Python基础设施通过在调用 `python-package` 或 `host-python-package` 宏前定义一系列变量来工作。

所有在[通用软件包基础设施](https://buildroot.org/downloads/manual/manual.html#generic-package-reference)中存在的软件包元数据信息变量，在Python基础设施中同样适用。

注意：

- 不需要在包的 `PYTHON_FOO_DEPENDENCIES` 变量中添加 `python` 或 `host-python`，因为这些基本依赖会被Python软件包基础设施自动添加。
- 同样，对于基于setuptools的软件包，也不需要在 `PYTHON_FOO_DEPENDENCIES` 中添加 `host-python-setuptools`，因为Python基础设施会根据需要自动添加。

有一个Python基础设施特有的变量是必须的：

- `PYTHON_FOO_SETUP_TYPE`，用于定义该包所使用的Python构建系统。支持的七种类型为 `flit`、`hatch`、`pep517`、`poetry`、`setuptools`、`setuptools-rust` 和 `maturin`。如果不确定包使用哪种类型，请查看包源码中的 `setup.py` 或 `pyproject.toml` 文件，看看是否有从 `flit` 或 `setuptools` 模块导入内容。如果包使用 `pyproject.toml` 文件且没有任何build-system requires，并且有本地in-tree backend-path，则应使用 `pep517`。

此外，还可以根据包的需要选择性地定义一些Python基础设施特有的变量。它们中的许多只在非常特定的情况下有用，典型的软件包通常只会用到其中的少数几个，甚至一个都不用。

- `PYTHON_FOO_SUBDIR` 可以包含包内包含主 `setup.py` 或 `pyproject.toml` 文件的子目录名称。如果主 `setup.py` 或 `pyproject.toml` 文件不在tarball解压后的根目录下，这会很有用。如果未指定 `HOST_PYTHON_FOO_SUBDIR`，则默认为 `PYTHON_FOO_SUBDIR`。
- `PYTHON_FOO_ENV`，用于指定传递给Python `setup.py` 脚本（对于setuptools包）或 `support/scripts/pyinstaller.py` 脚本（对于flit/pep517包）的额外环境变量，适用于构建和安装步骤。注意，基础设施会自动传递一些标准变量，这些变量在 `PKG_PYTHON_SETUPTOOLS_ENV`（针对setuptools目标包）、`HOST_PKG_PYTHON_SETUPTOOLS_ENV`（针对setuptools主机包）、`PKG_PYTHON_PEP517_ENV`（针对flit/pep517目标包）和 `HOST_PKG_PYTHON_PEP517_ENV`（针对flit/pep517主机包）中定义。
- `PYTHON_FOO_BUILD_OPTS`，用于指定传递给Python `python -m build` 调用的额外选项。要向构建后端传递额外选项，请使用 `build` 模块的 `--config-setting=`（`-C`）参数。
- `PYTHON_FOO_INSTALL_TARGET_OPTS`、`PYTHON_FOO_INSTALL_STAGING_OPTS`、`HOST_PYTHON_FOO_INSTALL_OPTS`，用于指定在目标安装、staging安装或主机安装步骤中传递给Python `setup.py` 脚本（对于setuptools包）或 `support/scripts/pyinstaller.py`（对于flit/pep517包）的额外选项。

使用Python基础设施，构建和安装软件包所需的所有步骤都已定义好，并且对于大多数基于Python的软件包来说都能很好地工作。然而，在需要时，仍然可以自定义某个特定步骤的行为：

- 通过添加后操作钩子（在提取、打补丁、配置、构建或安装之后）。详见[第18.23节，“各构建步骤可用的钩子”](https://buildroot.org/downloads/manual/manual.html#hooks)。
- 通过重写某个步骤。例如，即使使用了Python基础设施，如果包的 `.mk` 文件定义了自己的 `PYTHON_FOO_BUILD_CMDS` 变量，则会使用自定义的而不是默认的Python命令。但这种方法应仅限于非常特殊的情况。一般情况下不要使用。

### 18.9.3. 从PyPI仓库生成 `python-package`

如果你想为Buildroot创建的Python包在PyPI上有发布，可以使用 `utils/scanpypi` 工具（位于 `utils/` 目录下）来自动化该过程。

你可以在[这里](https://pypi.python.org/)找到现有PyPI包的列表。

`scanpypi` 需要你的主机上已安装Python的 `setuptools` 包。

在buildroot目录根下，直接执行：

```
utils/scanpypi foo bar -o package
```

如果 `foo` 和 `bar` 存在于 [https://pypi.python.org](https://pypi.python.org/) 上，这将会在package文件夹下生成 `python-foo` 和 `python-bar` 两个包。

找到 `external python modules` 菜单并将你的包插入其中。请注意，菜单中的条目应按字母顺序排列。

请记住，你很可能需要手动检查包是否有任何错误，因为有些内容生成器无法自动判断（例如对python核心模块如BR2_PACKAGE_PYTHON_ZLIB的依赖）。另外请注意，许可证和许可证文件是自动猜测的，必须手动检查。你还需要手动将包添加到 `package/Config.in` 文件中。

如果你的Buildroot包不在官方Buildroot树中，而是在br2-external树中，请使用如下的 -o 参数：

```
utils/scanpypi foo bar -o other_package_dir
```

这将在 `other_package_directory` 下生成 `python-foo` 和 `python-bar` 包，而不是在 `package` 目录下。

选项 `-h` 会列出可用选项：

```
utils/scanpypi -h
```

### 18.9.4. `python-package` CFFI后端

C Foreign Function Interface for Python（CFFI，C语言外部函数接口）为从Python调用用C编写的已编译代码提供了一种方便可靠的方式，接口声明采用C语言。依赖该后端的Python包可以通过其 `setup.py` 文件的 `install_requires` 字段中出现 `cffi` 依赖来识别。

此类包应：

- 添加 `python-cffi` 作为运行时依赖，以便在目标设备上安装已编译的C库包装器。可通过在包的 `Config.in` 文件中添加 `select BR2_PACKAGE_PYTHON_CFFI # runtime` 实现。

```
config BR2_PACKAGE_PYTHON_FOO
        bool "python-foo"
        select BR2_PACKAGE_PYTHON_CFFI # runtime
```

- 添加 `host-python-cffi` 作为构建时依赖，以便交叉编译C包装器。可通过在 `PYTHON_FOO_DEPENDENCIES` 变量中添加 `host-python-cffi` 实现。

```
################################################################################
#
# python-foo
#
################################################################################

...

PYTHON_FOO_DEPENDENCIES = host-python-cffi

$(eval $(python-package))
```
