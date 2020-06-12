---
layout: post
title:  "利用QT网络通信获取上证指数"
summary: 最近关注股市,于是在自己的番茄工作小应用中增加上证指数数据,用到QT的网络通信模块从东方财富网站上爬取数据.
featured-img: qt
# categories: jekyll update
---
## 东方财富上证指数数据获取 ##
### 分析HTML代码 ###
找到上证指数数据所在标签ID`price9`和引用的js文件`../newstatic/js/zs.js`
```
<div id="arrowud" xid="0">
  <span class="ar a"><strong id="price9" class="xp1" style="">2745.62</strong><i id="arrow-find" class="xp2 down-arrow"></i></span>
  <span class="ar b"><b id="km1" class="xp3" style="">0.00</b><b id="km2" class="xp4" style="">0.00%</b></span> 
</div> 
...
<script src="../newstatic/js/zs.js"></script> 
```
### 分析JS代码 ###
下载js文件,查找`price9`,找到请求过程
```
jQuery.ajax({
  url: baseUrl + "/api/qt/stock/get?secid=" + _this._Market + "." + _this._Code + '&ut=' + ut +'&fields=f118,f107,f57,f58,f59,f152,f43,f169,f170,f46,f60,f44,f45,f168,f50,f47,f48,f49,f46,f169,f161,f117,f85,f47,f48,f163,f171,f113,f114,f115,f86,f117,f85,f119,f120,f121,f122&invt=2',
  dataType: 'jsonp',
  jsonp: 'cb',
  scriptCharset: "utf-8",
  success: function (res) {
  // console.log("基本数据>>", res)
  var data = res.data
  getHqStat(data.f118)

  var zxj = data.f43 == '-' ? data.f43 : (data.f43 / Math.pow(10, data.f59)).toFixed(data.f59)
  var zd = (data.f169 == "-" ? "-" : (data.f169 / Math.pow(10, data.f59)).toFixed(data.f59))
  var zdf = data.f43 == '-' ? '-' : (data.f170 / 100).toFixed(2)
  var time = dateFtt("yyyy-MM-dd hh:mm:ss", new Date(data.f86 * 1000))
  var jkj = data.f46 == '-' ? data.f46 : (data.f46 / Math.pow(10, data.f59)).toFixed(data.f59)
  var zgj = data.f44 == '-' ? data.f44 : (data.f44 / Math.pow(10, data.f59)).toFixed(data.f59)
  var zdj = data.f45 == '-' ? data.f45 : (data.f45 / Math.pow(10, data.f59)).toFixed(data.f59)
  var zsj = data.f60 == '-' ? data.f60 : (data.f60 / Math.pow(10, data.f59)).toFixed(data.f59)
  var kpj = data.f46 == '-' ? data.f46 : (data.f46 / Math.pow(10, data.f59)).toFixed(data.f59)

  document.title = (data.f58 + " " + zxj + " " + zd + "(" + zdf + "%) _ 股票行情 _ 东方财富网");
  $("#price9").text(zxj).addClass(rendercolor(data.f43, data.f60)) //最新价
```
查找相关变量的值,提取出完整的URL:`curl http://push2.eastmoney.com/api/qt/stock/get?secid=1.000001&ut=bd1d9ddb04089700cf9c27f6f7426281&fields=f43,f60`,其中`f43`,`f60`分别为最近价和昨收价.

## QT网络通信介绍 ##

QT网络通信模块提供了实现TCP/IP客户端和服务端的类.它即针对低层次的网络概念(TCP/UDP)提供了低层次的类,如`QTcpSocket`,`QTcpSocket`, `QTcpServer`和`QUdpSocket`,也针对常用网络协议(HTTP/FTP)提供了高层次的类,如`QNetworkRequest`, `QNetworkReply`和`QNetworkAccessManager`,甚至也提供了通信载体管理相关的类,如`QNetworkConfiguration`,`QNetworkConfigurationManager`和`QNetworkSession`.本文主要介绍HTTP相关的类.

网络访问接口是一系列用来执行常用网络操作的类.API接口在不同网络操作和协议(如http的get和post操作)之上提供了一个抽象层,只对用户暴露通用的,高层概念相关的类,函数,信号.

类`QNetworkRequest`用来表示网络请求,也可以看做是请求信息的一个通用容器,这些信息可以是任何头信息和使用的加密选项.URL在网络请求对象创建以后指定,决定了网络请求最终用到的协议.目前HTTP,FTP和本地文件URL可以支持上传和下载.

类`QNetworkAccessManager`管理网络操作的执行.当一个网络请求创建后,这个类被用于发送网络请求,并触发QT信号来报告处理过程.这个管理类还被用来利用cookies在客户端存储数据,认证请求和用于代理.每个应用程序或库可以创建多个类`QNetworkAccessManager`的实例来处理网络通信.

类`QNetworkReply`用来表示网络请求的响应,它是由管理类`QNetworkAccessManager`在发出请求后创建.这个类的QT信号可以用于监控每一个响应和丢弃它们,或者开发这也可以选择使用管理类的QT信号来实现同样的目的.由于这个类是类`QIODevice`的子类,响应可以,就像阻塞和非阻塞操作一样,以同步或异步的方式被处理.

### 类QNetworkRequest ###
类`QNetworkRequest`是网络访问API的一部分,这个类持有用于在网络中发送请求的必要信息,包括一个URL和一些辅助信息.

### 类QNetworkAccessManager ###
网络访问API的构造是围绕类`QNetworkAccessManager`进行的,这个类里有一些用于发送请求的通用配置信息,包括代理和缓存,以及相关的QT信号,还有可以用于监控网络操作过程的响应QT信号.一个应用程序只需要一个`QNetworkAccessManager`实例就足够了,并且这个实例只能在当前线程中使用.

当`QNetworkAccessManager`对象创建完成后,应用程序可以用它在网络上发送请求.这个类提供了一组标准函数来发送请求和相关数据,并返回一个`QNetworkReply object`对象.返回的对象用来提取响应中的数据.

一个简单的从网络下载内容的例子如下:
```
QNetworkAccessManager *manager = new QNetworkAccessManager(this);
connect(manager, &QNetworkAccessManager::finished,this, &MyClass::replyFinished);
manager->get(QNetworkRequest(QUrl("http://qt-project.org")));
```
类`QNetworkAccessManager`还有一个异步的API.在上述例子中,当槽`replyFinished`被调用时,他的参数是一个`QNetworkReply `对象,包含了下载的数据和元数据(http头等).注意在请求完成后,用户负责在适当的时机释放`QNetworkReply`对象.不要直接在槽函数中释放.可以使用`deleteLater()`释放(释放过程由QT调度).另外,`QNetworkAccessManager`接受到的请求会进入队列,请求的并发执行数量取决于协议.目前,对于桌面平台的HTTP协议,可在一个主机端口上并行处理6个请求.

一个更加完善的例子如下:
```
QNetworkRequest request;
request.setUrl(QUrl("http://qt-project.org"));
request.setRawHeader("User-Agent", "MyOwnBrowser 1.0");

QNetworkReply *reply = manager->get(request);
connect(reply, &QIODevice::readyRead, this, &MyClass::slotReadyRead);
connect(reply, QOverload<QNetworkReply::NetworkError>::of(&QNetworkReply::error), this, &MyClass::slotError);
connect(reply, &QNetworkReply::sslErrors,this, &MyClass::slotSslErrors);
```
### 类QNetworkReply ###
类`QNetworkReply`包含了被`QNetworkAccessManager`发送出的请求的响应数据和元数据.与`QNetworkRequest`类似,它包含了一个URL,头数据,状态信息和响应内容本身.

类`QNetworkReply`是一个按顺序访问的`QIODevice`,也就是说,一旦数据读取过,设备中就不在有这些数据了.因此,需要应用程序自己保存需要保存的数据.当有更多数据从网络到达时,QT信号`readyRead()`会被释放.

QT信号`downloadProgress()`也会在收到数据的时候释放,但是,如果数据内容做过一些转换,那么数据的字节数可能不是实际收到的字节数,比如压缩处理和移除协议头.

尽管`QNetworkReply`是一个连接到响应数据内容的`QIODevice`,但它也能释放QT信号`uploadProgress()`,用来指示处理上传数据内容的过程.
