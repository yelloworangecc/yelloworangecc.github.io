---
layout: post
title:  "Windows COM库简介"
summary: "组件对象模型（Component Object Model，缩写COM）是微软的一套软件组件的二进制接口标准。这使得跨编程语言的进程间通信、动态对象创建成为可能。"
featured-img: os
---

COM的核心是一组组件对象间交互的规范，定义了组件对象如何与其用户通过二进制接口标准进行交互，COM的接口是组件的类型纽带。此外，COM还是一个称为“COM库”的实现，包括若干API函数，用于COM程序的创建与使用。

任何使用COM的进程必须初始化和取消初始化COM库.除了协议中规定的内容之外,COM也实现了一些重要的服务.这些服务通过一组DLL和EXE文件提供(如主要的Ole32.dll和Rpcss.exe),内容包括:

* 一些用来创建COM应用的基础函数,包括客户端和服务端应用.对于客户端,COM提供了用来创建对象的基础函数.对于服务端,COM提供了开放内部对象的手段.
* 实现定位器服务使得COM能够使用唯一类标识符(CLSID)确定哪一个服务实现了特定的类以及这个服务在何处.该服务在一个对象类标识和实现的封装之间提供了一个间接层,通常是系统注册表,这样客户端可以独立于封装,易于在未来进行变动.
* 针对在本地或远程服务中运行的对象提供透明的远程过程调用.
* 提供一个标准的机制来允许应用在进程内部控制内存的分配,特别是那些需要在协作对象见传递的内存,使得它们能够被正确的释放.

为了访问COM基础服务,客户端中运行的所有COM线程和进程外(out-of-process)运行的服务端,在调用除了内存分配外的其他任何COM函数都需要先调用CoInitialize或者CoInitializeEx.CoInitializeEx增加了一个允许你指定线程模型的参数,线程可以是套间线程(apartment-threaded)或者自由线程(free-threaded).而CoInitialize简单的将线程模型设置为套间线程.

OLE(Object Linking and Embedding)复合式文档应用需要调用OleInitialize函数,这个函数内部会调用CoInitializeEx并做一些复合式文档相关的初始化.因此,调用OleInitialize的线程不能是自由线程.

进程内(in-process)服务端不需要调用初始化函数,因为加载它们的进程已经做过初始化了.进程内服务端必须在注册表的InprocServer32键中设置线程模型.

此外,取消初始化也是非常重要的.对于CoInitialize或CoInitializeEx的每一次调用都必须有对应的CoUninitialize调用.OleInitialize对应的则是OleUninitialize.
