# Python 打包指南

## 1、install package

### 1.1、安装pip

```text
curl -o CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo

yum install epel-release

yum install python2-pip-8.1.2-10.el7.noarch
```

升级到最新

```text
pip install --upgrade pip setuptools wheel
```

### 1.2、pip命令使用

```bash
# pip --help

Usage:
  pip <command> [options]

Commands:
  install                     Install packages.  安装包
  download                    Download packages. 下载包
  uninstall                   Uninstall packages. 卸载包
  freeze                      Output installed packages in requirements format. 按着一定格式输出已安装包列表
  list                        List installed packages. 列出已安装包
  show                        Show information about installed packages. 显示包详细信息
  check                       Verify installed packages have compatible dependencies.检查包的依赖关系是否完整
  config                      Manage local and global configuration.管理配置
  search                      Search PyPI for packages.搜索包
  wheel                       Build wheels from your requirements.
  hash                        Compute hashes of package archives.计算包的hash值 
  completion                  A helper command used for command completion.
  help                        Show help for commands.

General Options:
  -h, --help                  Show help.
  --isolated                  Run pip in an isolated mode, ignoring environment variables and user configuration.
  -v, --verbose               Give more output. Option is additive, and can be used up to 3 times.
  -V, --version               Show version and exit.
  -q, --quiet                 Give less output. Option is additive, and can be used up to 3 times (corresponding to WARNING, ERROR, and CRITICAL logging levels).
  --log <path>                Path to a verbose appending log.
  --proxy <proxy>             Specify a proxy in the form [user:passwd@]proxy.server:port.
  --retries <retries>         Maximum number of retries each connection should attempt (default 5 times).
  --timeout <sec>             Set the socket timeout (default 15 seconds).
  --exists-action <action>    Default action when a path already exists: (s)witch, (i)gnore, (w)ipe, (b)ackup, (a)bort).
  --trusted-host <hostname>   Mark this host as trusted, even though it does not have valid or any HTTPS.
  --cert <path>               Path to alternate CA bundle.
  --client-cert <path>        Path to SSL client certificate, a single file containing the private key and the certificate in PEM format.
  --cache-dir <dir>           Store the cache data in <dir>.
  --no-cache-dir              Disable the cache.
  --disable-pip-version-check
                              Don't periodically check PyPI to determine whether a new version of pip is available for download. Implied with --no-index.
  --no-color                  Suppress colored output
```

### 1.3、安装包

```bash
# 安装
pip install "SomeProject"

# 安装指定版本
pip install "SomeProject==1.4"

# 安装版本间的包
pip install "SomeProject>=1,<2"

# 安装任何>=1.4.2 的 1.4.* 包
pip install "SomeProject~=1.4.2"

#从其他index安装
pip install --index-url http://my.package.repo/simple/ SomeProject

#除了PyPI，在安装过程中还搜索其他index
pip install --extra-index-url http://my.package.repo/simple SomeProject
```

### 1.4、安装源

pip 可以从[Source Distributions（sdist）](https://packaging.python.org/glossary/#term-source-distribution-or-sdist)或[Wheels](https://packaging.python.org/glossary/#term-wheel)中安装，但是如果PyPI上同时存在两者，则pip将更喜欢兼容的Wheel。

Wheel是一种预构建的发行格式，与Source Distributions（sdist）相比，它提供了更快的安装，尤其是在项目包含编译后的扩展名时。

如果pip找不到wheel用以安装，它将在本地构建一个wheel并将其缓存以备将来安装，而不是在将来重新构建source distribution。

### 1.5、Requirements files

[Requirements File Format](https://pip.readthedocs.io/en/latest/reference/pip_install/#requirements-file-format)

“Requirements files”是包含要使用pip install安装的包列表的文件，如下所示：

```bash
pip install -r requirements.txt
```

![](../.gitbook/assets/image%20%2810%29.png)

```bash
# 将当前环境的已安装包，输出到 requirments.txt中
pip freeze > requirements.txt
```

## 2、[管理 application 依赖](https://packaging.python.org/tutorials/managing-dependencies/)

官网讲述了如何使用Pipenv。暂忽略。

## 3、打包python项目

本教程将引导您如何打包一个简单的Python项目。 它将向您展示如何添加必要的文件和结构以创建软件包，如何构建软件包以及如何将其上传到Python软件包索引。

### 3.1、创建project

创建如下结构的目录，然后进入顶层文件夹`cd packaging_tutorial`下执行所有命令

```bash
packaging_tutorial/
  example_pkg/
    __init__.py
```

创建一些文件来打包该项目并为分发做准备

```text
packaging_tutorial/
  example_pkg/
    __init__.py
  tests/
  setup.py
  LICENSE
  README.md
```

### 3.2、tests 目录

tests/是单元测试文件的占位符。 现在将其留空

### 3.3、setup.py

`setup.py`是`setuptools`的构建脚本。 它告诉setuptools有关您的软件包（例如名称和版本）以及要包括的代码文件的信息。

打开`setup.py`并输入以下内容。 更新软件包名称以包含您的用户名（例如`example-pkg-theacodes`），以确保您具有唯一的软件包名称，并且该软件包与本教程中其他人上传的软件包没有冲突。

```text
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="example-pkg-YOUR-USERNAME-HERE", # Replace with your own username
    version="0.0.1",
    author="Example Author",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)
```

setup\(\) 需要几个参数。 此示例包使用相对最小的集合：

* `name` 是程序包的分发名称。 该名称可以是任何名称，只要仅包含字母，数字，`_`和`-`。
* `version` is the package version see [**PEP 440**](https://www.python.org/dev/peps/pep-0440) for more details on versions.
* `author` and `author_email` 用于标识软件包的作者.
* `description` 是该软件包的简短摘要.
* `long_description` 是程序包的详细说明。 这在Python Package Index的包详细信息包中显示。 在这种情况下，详细描述是从`README.md`加载的，这是一种常见模式。.
* `long_description_content_type` 告诉索引用于长描述的标记类型。此例是 Markdown.
* `url` 是项目主页的URL。 对于许多项目，这仅是指向GitHub，GitLab，Bitbucket或类似代码托管服务的链接。
* `packages` 是应该包含在分发软件包中的所有Python导入软件包的列表。 无需手动列出每个软件包，我们可以使用`find_packages（）`自动发现所有软件包和子软件包。 在这种情况下，软件包列表将为example\_pkg，因为这是唯一的软件包.
* `classifiers(分类器)` 提供索引，并为您的软件包提供一些其他元数据。 在此例中，该软件包仅与Python 3兼容，已获得MIT许可，并且与操作系统无关。 您应始终至少包括您的软件包所使用的Python版本，软件包所使用的许可证以及软件包所使用的操作系统。 有关分类器的完整列表，请参见 [https://pypi.org/classifiers/](https://pypi.org/classifiers/).

有关更多详细信息，请参见[Packaging and distributing projects](https://packaging.python.org/guides/distributing-packages-using-setuptools/)。

### 3.4、README.md

打开README.md并输入以下内容。 您可以根据需要对此进行自定义。

```text
# Example Package

This is a simple example package. You can use
[Github-flavored Markdown](https://guides.github.com/features/mastering-markdown/)
to write your content.
```

### 3.5、LICENSE

重要的是，每个上传到Python Package Index的软件包都必须包含许可证。 这会告诉安装软件包的用户使用条款。 有关选择许可证的帮助，请参阅[https://choosealicense.com/。](https://choosealicense.com/。) 选择LICENSE后，打开“LICENSE”并输入许可证文本。 例如，如果您选择了MIT许可证：

```text
Copyright (c) 2018 The Python Packaging Authority

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### 3.6、Generating distribution archives

下一步是为程序包生成分发程序包。 这些是已上传到软件包索引的归档文件，可以通过pip安装。

确保已安装最新版本的setuptools和wheel：

```text
python3 -m pip install --user --upgrade setuptools wheel
```

现在，在`setup.py`所在的目录运行命令：

```text
python3 setup.py sdist bdist_wheel
```

此命令应输出大量文本，完成后应在dist目录中生成两个文件：

```text
dist/
  example_pkg_YOUR_USERNAME_HERE-0.0.1-py3-none-any.whl
  example_pkg_YOUR_USERNAME_HERE-0.0.1.tar.gz
```

`tar.gz`文件是`source archive`，而`.whl`文件是`built distribution`。 较新的pip版本会优先安装`built distribution`，但如果需要，将回找到源归档文件。 您应该始终上传源归档，并为项目兼容的平台提供内置包。 在这种情况下，我们的示例包在任何平台上都与Python兼容，因此仅需要一个内置包。

### 3.7、上传包

最后，是将您的软件包上传到Python软件包索引。

您需要做的第一件事是在Test PyPI上注册一个帐户。 测试PyPI是用于测试和实验的包索引的单独实例。 对于本教程这样的事情非常有用，因为我们不一定要上传到真实索引。 要注册帐户，请访问[https://test.pypi.org/account/register/并完成该页面上的步骤。](https://test.pypi.org/account/register/并完成该页面上的步骤。) 您还需要先验证电子邮件地址，然后才能上传任何软件包。 有关测试PyPI的更多详细信息，请参阅[Using TestPyPI](https://packaging.python.org/guides/using-testpypi/)。

现在，您将创建一个PyPI [API token](https://test.pypi.org/help/#apitoken)，以便能够安全地上传您的项目。

转到[https://test.pypi.org/manage/account/\#api-tokens并创建一个新的API令牌；](https://test.pypi.org/manage/account/#api-tokens并创建一个新的API令牌；) 不要将其范围限制为特定项目，因为您正在创建一个新项目

**在您复制并保存令牌之前，请不要关闭页面-您不会再看到该令牌。**

现在您已经注册，可以使用**`twine`**上载分发包。 您需要安装Twine：

```text
python3 -m pip install --user --upgrade twine
```

安装完成后，运行Twine将所有归档上传到dist下：

```text
python3 -m twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```

系统将提示您输入用户名和密码。 对于用户名，请使用`__token__` 对于口令，请使用令牌值，包括`pypi-`前缀。

命令完成后，您应该看到类似于以下的输出：

![](../.gitbook/assets/image%20%2823%29.png)

上传后，您的软件包应该可以在TestPyPI上查看，例如[https://test.pypi.org/project/example-pkg-YOUR-USERNAME-HERE](https://test.pypi.org/project/example-pkg-YOUR-USERNAME-HERE)

### 3.8、安装测试

创建virtualenv，安装package

```text
python3 -m pip install --index-url https://test.pypi.org/simple/ --no-deps example-pkg-YOUR-USERNAME-HERE
```

### 3.9、深入链接

* Read more about using [setuptools](https://packaging.python.org/key_projects/#setuptools) to package libraries in [Packaging and distributing projects](https://packaging.python.org/guides/distributing-packages-using-setuptools/).
* Read about [Packaging binary extensions](https://packaging.python.org/guides/packaging-binary-extensions/).
* Consider alternatives to [setuptools](https://packaging.python.org/key_projects/#setuptools) such as [flit](https://packaging.python.org/key_projects/#flit), [hatch](https://github.com/ofek/hatch), and [poetry](https://github.com/sdispater/poetry).

## 参考文档

[Python Packaging User Guide](https://packaging.python.org/)



