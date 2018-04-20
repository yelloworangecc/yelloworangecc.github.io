---
layout: post
title:  "QT4: 资源编译器"
summary: QT资源系统是独立于平台的,存储二进制文件到应用的可执行程序的体系.资源系统提供的rrc工具可以在编译过程中嵌资源到qt应用程序.
featured-img: sleek
# categories: jekyll update
---
## 用法 ##

rrc工具用于在编译过程中嵌入资源到qt应用程序,通过生成一个包含qt资源文件中指定的数据的C++源文件来实现.

`rcc [options] <inputs>`

命令行参数如下:

  * `-o file` 将输出写入文件而不是到标准输出
  * `-name name` 使用指定名称创建一个外部初始化函数
  * `-threshold level` 指定用于判断是否需要压缩文件的阈值.默认值为70%.
  * `-compress level` 压缩输入文件到指定压缩级别,1最快,9最节省空间.默认为-1,使用zlib的默认级别.
  * `-root path` 访问资源的根路径
  * `-no-compress` 不压缩
  * `-binary` 输出到二进制文件用做动态资源
  * `-version` 显示版本信息
  * `-help` 显示帮助信息

## QT资源系统 ##

QT资源系统是独立于平台的,存储二进制文件到应用的可执行程序的体系.当你的应用总是需要一些文件集时非常有用,你不必担心丢失这些文件.资源系统建立在qmake,rrc和QFile的紧密合作之上.他淘汰了qt3的qembed工具和图片集合机制.

### 资源集合文件 ###

和一个应用相关联的资源在`.qrc`后缀的文件中指定,这是一个xml格式的文件列表,在其中给列表中的文件分配资源名称,应用程序必须通过资源名称访问资源.下面是一个例子.

```
<!DOCTYPE RCC><RCC version="1.0">
<qresource>
    <file>images/copy.png</file>
    <file>images/cut.png</file>
    <file>images/new.png</file>
    <file>images/open.png</file>
    <file>images/paste.png</file>
    <file>images/save.png</file>
</qresource>
</RCC>
```

`.qrc`文件中列出的资源是应用程序资源树上的一部分.使用相对于`.qrc`文件所在目录的相对路径.`.qrc`文件中列出的资源文件必须与其在同一目录或者在同目录的子目录中.资源数据可以编译到二进制中,这样应用程序代码可以立即访问,或者可以创建一个二进制资源并在稍后应用程序将其注册到资源系统.默认情况下,使用带`:/`前缀的名称访问资源,也可以使用qrc制定的URL访问.例如,`:/images/cut.png`和`qrc:///images/cut.png`可以访问到cut.png,此外通过xml表情的属性定义别名来访问:

```
<file alias="cut-img.png">images/cut.png</file>
```

现在,可以在应用程序中使用`:/cut-img.png`访问上面的资源了.

有些资源需要根据用户的位置改变,比如翻译文件或者图标.这时可以通过添加`lang`属性来制定一个合适的位置,例如:

```
<qresource>
    <file>cut.jpg</file>
</qresource>
<qresource lang="fr">
    <file alias="cut.jpg">cut_fr.jpg</file>
</qresource>
```

如果用户位于法国,`:/cut.jpg`变成对图片cut_fr.jpg的引用,其他位置仍然使用cut.jpg.

### 外部二进制资源 ###

对于创建外部二进制资源,你必须在rrc创建资源数据时传入`-binary`选项.一旦二进制资源创建好厚,你可以使用QResoure API进行注册.例如,`.qrc`文件中指定的资源集合可以这样编译`rcc -binary myresource.qrc -o myresource.rcc`,在应用程序中资源可以这样注册`QResource::registerResource("/path/to/myresource.rcc");`.

### 内置资源 ###

对于编译到二进制的资源,`.qrc`文件必须在应用程序的`.pro`文件中登记,这样qmake才能感知文件的存在.例如,`RESOURCES     = application.qrc`.qmake将会生成规则来创建名为qrc_application.cpp的文件,文件最终将编译和链接到应用程序.这个文件中包含了所有图片和其他资源的数据,数据压缩存储在静态C++数组中.当`.qrc`文件变化或者列表中的任意文件变化时,qrc_application.cpp会自动重新生成.如果不是用`.pro`文件,你也可以手动触发rrc,并把规则添加到你自己的编译系统中去.

当前,QT总是将数据直接存储到可执行程序,即使是在原生支持资源文件的Windows和Mac OS X上.这个问题未来可能会改进.

### 压缩 ###

资源默认以ZIP格式压缩,也可以关闭压缩功能,因为有些资源文件本身包含压缩格式,比如`.png`.不需要压缩是使用`-no-compress`参数即可,例如,`rcc -no-compress myresources.qrc`.`.rrc`也提供了几个可以控制压缩的参数,你可以指定压缩级别已经阈值级别,例如,`rcc -compress 2 -threshold 3 myresources.qrc`

### 在应用程序中使用资源 ###

在应用程序中,大多数情况下都可以使用资源路径代替原始的文件系统路径.在特殊的情况下,你可以传一个资源路径代替文件的名称到QIcon,QImage或者QPixmap的构造函数中:`cutAct = new QAction(QIcon(":/images/cut.png"), tr("Cu&t"), this);`

在内存中,资源由树中的资源对象来表示,树在启动程序时自动创建并使用QFile来处理路径找到对应资源,你可以使用初始化为`:/`的QDir在资源树中从根节点开始导航.

QT的资源支持查找路径列表的概念,如果你使用`:`代替`:/`作为前缀引用资源,QT会使用查找路径列表来查找资源.查找路径列表在启动时时空的,调用`QDir::addSearchPath()`将路径加入其中.

如果静态库中包含资源,你需要调用`Q_INIT_RESOURCE()`来强制初始化资源,参数为`.qrc`文件不带后缀的名称,例如:

```
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
    Q_INIT_RESOURCE(graphlib);
    ...
    return app.exec();
}
```

类似地,如果必须显示地卸载资源(比如卸载插件,资源不在合法),你可以调用`Q_CLEANUP_RESOURCE()`来卸载,参数和上面相同.
