---
layout: post
title:  "POSIX线程接口手册"
summary: POSIX（Portable Operating System Interface of UNIX）表示可移植操作系统接口，POSIX标准定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称，线程接口是标准的一部分。
featured-img: os
# comments: true
---
## pthread_create ##

  函数用于创建一个新线程,原型为:`int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *),void *arg);`
  
  在进程中创建线程时可以通过参数attr指定属性,如果attr为NULL时使用默认属性.函数调用后对属性的修改不会影响线程.当函数执行成功时,线程ID会存储到参数thread中.
  
  线程创建口会去执行start\_routine并附带一个参数.start\_routine返回时,就像在退出状态隐式调用了pthread\_exit,返回值就是pthread\_exit的参数.注意主线程的行为方式与此不同,当main函数返回时,就像在退出状态隐式调用了exit,返回值就是exit函数的参数.
  
  当线程退出时,线程的存储空间必须被另一个线程通过调用pthread\_join()来回收.或者,对线程调用pthread\_detach以指示系统在退出时自动回收线程的存储空间.函数pthread\_attr\_setdetachstate可以设置attr并传递给pthread\_create用于在创建线程时达到相同的效果.
  
  线程的信号状态会被设置为:
  
  * 信号掩码从主创线程继承.
  * 新线程的信号集合为空.
  
  返回值:成功时返回0,否则返回错误码.
  
  错误码:
  * [EAGAIN] 系统资源不足或达到进程拥有线程数的上线.
  * [EPERM]  没有权限设置调度策略和调度参数.
  * [EINVAL] 非法的attr值.

## pthread_exit ##

函数用于终止调用的线程,原型为:`void pthread\_exit(void *value_ptr);`

函数终止线程并准备value\_ptr指针所指内容给成功join的调用者使用.任何已入栈但还未出栈的终止清理程序将会以与入栈相反的顺序出栈和执行.在所有清理程序执行完后如果还有线程数据位释放,相关的线程析构函数会以无序的方式调用.线程终止不会释放任何应用程序可见的进程资源,包括但不限于互斥锁,文件描述符;也不会执行进程级别的清理函数,包括但不限于调用atexit例程.

当一个非main函数线程第一次从start\_routine(创建线程时传入的函数)返回时,会隐式地调用pthread_exit.函数的参数则指定为return语句返回的值.

如果在清理程序或者线程析构函数中调用pthread\_exit时,系统的行为未定义.清理程序和线程析构函数本身是在显式或隐式调用pthread\_exit时触发的.

在线程已经终止后,继续访问线程的局部变量的结果是未定义的.因此,不能使用局部变量的指针作为pthread\_exit的value\_ptr参数.

当最后一个线程终止后,进程以状态码0退出,行为与调用exit函数并以0作为参数一样.

pthread_exit函数无法返回到它的调用者,也不会产生错误码.

## pthread_cancel ##

函数用于取消线程的执行,原型为:`int pthread\_cancel(pthread\_t thread);`

pthread\_cancel请求取消参数thread的所指代线程.目标线程的状态和类型决定何时执行取消动作.当取消动作激活,取消清理函数被调用.当最后一个清理函数返回时,线程相关数据析构函数将被调用,当最后一个析构函数返回时,线程被终止.

目标线程中的线程取消处理过程相对于调用pthread\_cancel的线程是异步执行的.

任何线程join目标线程可以得到一个PTHREAD\_CANCELED状态.PTHREAD\_CANCELED是类型为void *的常量表达式,它的值既不是内存对象的指针,也不是NULL.

在调用成功时,函数返回0.否则返回一个错误码.

错误码:
  * [ESRCH] 找不到线程ID对应的线程.
  
## 线程属性相关函数 ##
```
int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
int pthread_attr_setstack(pthread_attr_t *attr, void *stackaddr, size_t stacksize);
int pthread_attr_getstack(const pthread_attr_t * restrict attr, void ** restrict stackaddr, size_t * restrict stacksize);
int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
int pthread_attr_getstacksize(const pthread_attr_t *attr, size_t *stacksize);
int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
int pthread_attr_getguardsize(const pthread_attr_t *attr, size_t *guardsize);
int pthread_attr_setstackaddr(pthread_attr_t *attr, void *stackaddr);
int pthread_attr_getstackaddr(const pthread_attr_t *attr, void **stackaddr);
int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);
int pthread_attr_setinheritsched(pthread_attr_t *attr, int inheritsched);
```

线程属性用于为pthread\_cteate指定参数.一个属性对象可以用在多个pthread\_create调用中,多次调用之间,可以修改属性对象.

init函数用默认值来初始化属性对象attr.
destory函数销毁属性对象attr.
set系列函数设置属性对象中的某个属性,具体属性体现在函数的名字中.
get系列函数拷贝属性对象中的某个属性到第二个参数所指内存中,具体属性体现在函数的名字中.

函数调用成功时返回0.否则返回错误码.

错误码:
  * [ENOMEM] 内存耗尽,init函数会出现这类错误
  * [EINVAL] attr非法,destory函数会出现这类错误

## phtread_join ##

函数等待线程终止,原型为:`int pthread_join(pthread_t thread, void **value_ptr);`

pthread\_join函数挂起调用线程直到目标线程终止,除非目标线程已经终止.

成功调用pthread\_join函数,并且参数value\_ptr不为空时,pthread\_exit调用时指定的参数被存储在value\_ptr所指位置. 此函数返回成功,说明目标线程已经被终止.对同一个线程同时调用此函数的结果是未定义的. 如果调用线程被终止,那么目标线程不会被detach.

函数调用成功时返回0.否则返回错误码.

错误码:
  * [EINVAL] 目标线程不是一个可join的线程
  * [ESRCH] 找不到线程ID对应的线程
  * [EDEADLK] 检测到死锁,或者目标线程ID是调用线程本身
