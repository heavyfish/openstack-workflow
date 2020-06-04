---
description: 陆续更新，目前只更新遇见的
---

# Python打包指南-术语表

## 1、[通用术语表](https://packaging.python.org/glossary/#term-built-distribution)

### Binary Distribution

一种特定类型的 [Built Distribution](https://packaging.python.org/glossary/#term-built-distribution) ，包含已编译的扩展 extensions.

### Built Distribution

包含文件和元数据的[Distribution](https://packaging.python.org/glossary/#term-distribution-package)格式，只需将其移动到要安装的目标系统上的正确位置即可。 Wheel是这种格式，而distutil的Source Distribution不是，因为它需要一个构建步骤才能安装。 此格式并不意味着必须预先编译Python文件（Wheel故意不包括已编译的Python文件）

可以直接理解为安装包

### Distribution Package

版本化的归档文件，其中包含用于安装对应版本的Python packages，modules 和其他资源文件。 归档文件是最终用户将从Internet下载并安装的文件。

distribution package 通常用“ package”或“ distribution”一词来表示，通常使用单个术语“distribution”来指代。

### Egg

`setuptools` 引入的 Built Distribution格式，已由Wheel取代。 有关详细信息，请参见 [The Internal Structure of Python Eggs](https://setuptools.readthedocs.io/en/latest/formats.html) and [Python Eggs](http://peak.telecommunity.com/DevCenter/PythonEggs)。

### Extension Module

低级语言编写的模块用于给Python使用：如C/C++ 于 Python，Java 于 Jython。 通常包含在单个可动态加载的预编译文件中，例如 Unix上用于Python扩展的共享对象（.so）文件，Windows上用于Python扩展的DLL（给定的.pyd扩展）或Jython扩展的Java类文件

### Known Good Set \(KGS\)

A set of distributions at specified versions which are compatible with each other. Typically a test suite will be run which passes all tests before a specific set of packages is declared a known good set. This term is commonly used by frameworks and toolkits which are comprised of multiple individual distributions.

### Import Package

一个Python模块可以包含其他模块或递归地包含其他包。

import package 通常用单个词“package”来指代，但是注意避免与Distribution Package混淆

### Module

Python中代码可重用性的基本单位，以两种类型之一存在：Pure Module或Extension Module。

### Package Index

具有Web界面的分发仓库。

### Per Project Index

私有或其他非规范Package Index。被特定项目指明为首选索引 或 为解决该项目的依赖关系所需

### Project

旨在打包到 Distribution\(发行版\) 中的库\(library\)，框架，脚本，插件，application 或其他资源的数据集合或它们的某种组合。

由于大多数项目都是使用distutils或setuptools创建 发行版 的，因此当前定义项目的另一种实用方法是在项目的根目录中包含setup.py，其中“ setup.py”是distutils和setuptools使用的项目规范文件名。

Python项目必须具有在PyPI上注册的唯一名称。 每个项目将包含一个或多个版本，每个版本可包含一个或多个发行版。

### Pure Module

用Python编写并包含在单个.py文件中的模块。

### Python Packaging Authority \(PyPA\)

PyPA是一个工作组，负责维护Python打包中的许多相关项目。

### Python Package Index \(PyPI\)

PyPI是Python社区的默认Package Index。 它对所有Python开发人员开放以使用和分发其发行版。

### pypi.org

pypi.org是Python软件包索引（PyPI）的域名。 它在2017年取代了旧索引索引域名pypi.python.org

### Release

项目在特定时间点的快照，由版本标识符表示。

制作一个release可能需要发布多个Distributions。 例如，如果发布了项目的1.0版，则可能以source distribution 格式和Windows安装程序格式提供。

### Requirement

要安装的软件包的规范。 PYPA推荐的安装程序pip允许采用各种形式的规范，这些规范都可以被视为“Requirement”。

### Requirement Specifier

pip用于从“Package Index”安装程序包的格式。 有关格式的EBNF图，请参阅setuptools文档中的[pkg\_resources.Requirement](https://setuptools.readthedocs.io/en/latest/pkg_resources.html#requirement-objects)条目。 例如，“ foo&gt; = 1.3”是需求说明符，其中“ foo”是项目名称，而“&gt; = 1.3”部分是版本说明符

### Requirements File

包含可以使用pip安装的需求列表的文件。 [Requirements Files](https://pip.pypa.io/en/latest/user_guide/#requirements-files).

### setup.py

distutils和setuptools的项目规范文件。

### Source Archive

在Source Distribution 或 Built Distribution.之前，包含Release的原始源代码的归档。

### Source Distribution \(or “sdist”\)

一种distribution格式（通常使用`python setup.py sdist`生成），提供metadata 和 必要的source files 给 pip 或 用以生成Built Distribution

### System Package

以操作系统固有格式提供的软件包，例如 rpm或dpkg文件

### Version Specifier

Requirement Specifier\(说明符\) 的 version 部分。 例如，“ foo&gt; = 1.3”的“&gt; = 1.3”部分

### Virtual Environment

一个隔离的Python环境，允许安装软件包以供特定应用程序使用，而不是在系统范围内安装。

### Wheel

[PEP 427](https://www.python.org/dev/peps/pep-0427)引入的Built Distribution 格式，旨在代替Egg格式。 pip目前支持Wheel。.

### Working Set

可供导入的发行版集合。 这些路径分布在sys.path变量中。 在工作集中最多只能有一个版本的发行版

## 2、[项目术语表](https://packaging.python.org/key_projects/#pip)

### distlib

`distlib`是一个库，用于实现与Python软件的打包和分发有关的低级功能。 `distlib`实现了几个相关的PEP（Python增强建议标准），对于第三方打包工具的开发人员来说非常有用，它们可以制作和上传二进制和源代码发行版，实现互操作性，解决依赖关系，管理程序包资源以及执行其他类似功能。

不同于更严格的`packaging`项目，`packaging`专门实现了现代Python打包互操作性标准）。`distlib`在被要求处理早于现代互操作性标准并且属于不兼容的软件包子集的旧版软件包和元数据时，也会尝试提供合理的兼容行为。

### packaging

`pip`和`setuptools`使用的Python核心打包工具。

`packaging` 库中的核心实用程序处理 `version handling, specifiers, markers, requirements, tags`以及`Python packages` 的类似属性和任务。 大多数Python用户都依赖此库，而无需显式调用它。 开发人员经常使用此处列出的Python打包，分发和安装工具，其功能来解析，发现和处理依赖项属性。

此项目专门致力于实现[PyPA规范](https://packaging.python.org/specifications/#packaging-specifications)中定义的现代Python打包互操作性标准，并将报告与这些标准不兼容的足够旧的旧版软件包的错误。 相比之下，`distlib`项目是一个允许范围更大的库，在`packaging`将报告错误的情况下，该库试图提供对不明确的元数据的合理读取。

### pip

安装Python软件包的最流行的工具，现代版本的Python附带的工具。

它提供了从PyPI和其他Python软件包索引中查找，下载和安装软件包的基本核心功能，并且可以通过其命令行界面（CLI）集成到各种开发工作流程中。

### pipenv

Pipenv是一个旨在将所有 打包\(packaging\) 的精华带入Python世界的项目。 它将`Pipfile`，`pip`和`virtualenv`整合到一个工具链中。 它具有非常漂亮的终端颜色。

Pipenv旨在帮助用户在命令行上管理环境，依赖项和导入的程序包。 它也可以在Windows（其他工具通常不常用）上很好地工作，制作和检查文件hashes，以确保符合hash-locked的依赖项说明符，并简化了程序包和依赖项的卸载。 Python用户和系统管理员使用它，但是自2018年末以来维护较少。

### Pipfile

`Pipfile`及其姐妹`Pipfile.lock`是高级别的application-centric，其代理了pip的低级别`requirements.txt`文件

### readme\_renderer

`readme_renderer`是一个库，程序包开发人员可使用该库将其用户文档（README）文件从Markdown或reStructuredText等标记语言转换为HTML。

开发人员在发布管理过程中会单独或通过[twine](https://packaging.python.org/key_projects/#twine)调用它，以检查其软件包描述是否正确显示在PyPI上。

### twine

Twine是开发人员用来将软件包上传到Python软件包索引或其他Python软件包索引的主要工具。

它是一个命令行程序，将程序文件和元数据传递到Web API。

开发人员之所以使用它，是因为它是官方的PyPI上传工具，它既快速又安全，得到维护并且可靠地工作。

### setuptools

setuptools（包括`easy_install`）是Python distutils的增强功能的集合，使您可以更轻松地构建和分发Python发行版，尤其是依赖于其他软件包的发行版。

[distribute](https://pypi.org/project/distribute)是setuptools的一个分支，它被合并回`setuptools`（在v0.7中），从而使`setuptools`成为Python打包的主要选择。

### wheel

最初，wheel项目提供了`bdist_wheel` setuptools扩展名，用于创建wheel 发行版。 此外，它提供了自己的命令行实用程序来创建和安装wheels。

另请参阅[auditwheel](https://github.com/pypa/auditwheel)，这是程序包开发人员用来检查和修复其以binary wheel格式编写的Python程序包的工具。 它提供了以下功能：发现依赖关系，检查元数据是否符合要求，并修复wheel和元数据以正确链接，并将外部共享库包含在包中。

### distutils

原始的Python打包系统，已添加到Python 2.0的标准库中。

由于维护打包系统的挑战，在打包系统中，功能更新与language runtime更新紧密耦合，因此现在不鼓励直接使用distutils，而setuptools是首选。 setuptools不仅提供普通distutils不提供的功能（例如依赖项声明和入口点声明），而且还提供所有受支持的Python版本之间一致的构建接口和功能集。

## 3、补充

tar.gz格式：这个就是标准压缩格式，里面包含了项目元数据和代码，使用python setup.py sdist命令生成。 

.egg格式：本质上一个压缩文件，扩展名换了，里面也包含了项目元数据以及源代码。可以通过命令python setup.py bdist\_egg命令生成。 

.whl格式：这个是Wheel包，也是一个压缩文件，只是扩展名换了，里面也包含了项目元数据和代码。可以通过命令python setup.py bdist\_wheel生成.

