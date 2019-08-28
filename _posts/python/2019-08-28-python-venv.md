---
layout: post
title:  "Python虚拟环境"
summary: "VENV模块提供了对创建轻量虚拟环境的支持,虚拟环境有自己的site目录,与系统site目录隔离.每一个虚拟环境还有自己的Python二级制文件,并可在自己的site目录中有独立的Python包集合."
featured-img: python
# comments: true
---

VENV模块提供了对创建轻量虚拟环境的支持,虚拟环境有自己的site目录,与系统site目录隔离.每一个虚拟环境还有自己的Python二级制文件,并可在自己的site目录中有独立的Python包集合.

**注意:** `pyvenv` 脚本在Python3.6以后被废弃,请使用`python3 -m venv`代替

## 创建虚拟环境 ##

`python3 -m venv /path/to/new/virtual/environment`

运行这个命令会创建目标目录并且在这个目录下生产一个`pyvenv.cfg`文件,文件内容如下.

```
home = /usr/bin
include-system-site-packages = false
version = 3.7.2
```

同时,命令还创建了一些子目录,如`bin  include  lib  lib64`.

`bin`子目录中包含了Python二进制文件的拷贝或符号链接.在`lib`子目录中还有`pythonX.Y/site-packages`子目录,在初始化时是空的.

使用`-h`选项可以查看帮助信息,如下.

```
$ python3 -m venv -h
usage: venv [-h] [--system-site-packages] [--symlinks | --copies] [--clear]
            [--upgrade] [--without-pip] [--prompt PROMPT]
            ENV_DIR [ENV_DIR ...]

Creates virtual Python environments in one or more target directories.

positional arguments:
  ENV_DIR               A directory to create the environment in.

optional arguments:
  -h, --help            show this help message and exit
  --system-site-packages
                        Give the virtual environment access to the system
                        site-packages dir.
  --symlinks            Try to use symlinks rather than copies, when symlinks
                        are not the default for the platform.
  --copies              Try to use copies rather than symlinks, even when
                        symlinks are the default for the platform.
  --clear               Delete the contents of the environment directory if it
                        already exists, before environment creation.
  --upgrade             Upgrade the environment directory to use this version
                        of Python, assuming Python has been upgraded in-place.
  --without-pip         Skips installing or upgrading pip in the virtual
                        environment (pip is bootstrapped by default)
  --prompt PROMPT       Provides an alternative prompt prefix for this
                        environment.

Once an environment has been created, you may wish to activate it, e.g. by
sourcing an activate script in its bin directory.
```

3.4版本以后,虚拟环境会默认安装`pip`,可使用`--without-pip`选项禁止安装.`--copies`指定使用二进制文件的副本而不是符号链接.

3.4版本之前,如果目标目录已经存在,创建虚拟环境时会报错,除非指定了`--clear`或`--upgrade`选项.

注意:Windows平台支持但不推荐使用符号链接.特别值得注意的是,在文件资源管理器中双击`python.exe`将导致符号链接过早被解释,使得虚拟环境被忽略.

`pyvenv.cfg`文件中有个`include-system-site-packages`键,如果设置为`true`,venv将以`--system-site-packages`选项运行.

`venv`命令可以指定多个路径,每一个路径都会按照指定的选项创建独立的虚拟环境.

虚拟环境创建以后,可以使用虚拟环境`bin`目录下的激活脚本来激活.不同平台,激活脚本的形态不同.如下,其中`<venv>`使用虚拟环境的时间路径代替.

**bash/zsh** `$ source <venv>/bin/activate`

**fish** `$ . <venv>/bin/activate.fish`

**csh/tcsh** `$ source <venv>/bin/activate.csh`

**cmd.exe** `C:\> <venv>\Scripts\activate.bat`

**PowerShell** `PS C:\> <venv>\Scripts\Activate.ps1`

激活脚本实际上只是将虚拟环境中的二进制目录添加到`path`中,使得运行Python时使用虚拟环境中的Python解释器,并且运行一些已安装的脚本时不需要使用完整的路径.所有安装在虚拟环境的脚本在虚拟环境没有激活时也是可运行,在虚拟环境中时会被自动运行.

运行`deactivate`可以退出虚拟环境.实现机制特定于平台(会用到一些平台相关的脚本和函数).

注意:一个虚拟环境是由Python解释器,库,已安装脚本构成的与其他虚拟环境以及系统环境隔离的Python环境.一个虚拟环境是一个包含Python可执行文件和其他文件构成的目录树.

像`setuptools`和`pip`这样的安装工具可以在虚拟环境中正常运行.换句话说,当虚拟环境激活时,这些安装工具会将Python包默认安装到虚拟环境中.

当虚拟环境激活时,`sys.prefix`和`sys.exec_prefix`指向虚拟环境的基目录.而`sys.base_prefix`和`sys.base_exec_prefix`则指向系统环境中的Python安装目录.如果虚拟环境未激活,`sys.prefix`,`sys.exec_prefix`与`sys.base_prefix`,`sys.base_exec_prefix`的指向相同(系统Python安装目录).

当虚拟环境激活时,任何改变安装路径的选项会被忽略以阻止在虚拟环境外部安装Python包.

在命令行环境下,通过激活脚本激活虚拟环境,其他情况下则无需激活虚拟环境,在虚拟环境中安装的脚本会有一个`shebang`行,指向虚拟环境中的Python解释器.这也意味这脚本将不再一来`PATH`环境变量的内容.在Windows平台上,`shebang`行的处理需要安装Python启动器(Python Launcher).这样双击脚本时,脚本会自动使用正确的解释器执行,而不再依赖`path`环境变量中的内容.

## API ##

上面提到的一些列高层方法依赖于一个简单的底层API,这个API提供了为第三方虚拟环境创建者定制虚拟环境的机制,这个API就是类EnvBuilder.详细内容请参考[官方文档](https://docs.python.org/3/library/venv.html),本文不再做介绍.

