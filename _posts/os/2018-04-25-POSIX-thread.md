---
layout: post
title:  "POSIX线程编程手册"
summary: POSIX（Portable Operating System Interface of UNIX）表示可移植操作系统接口，POSIX标准定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称，线程接口是标准的一部分。
featured-img: os
# comments: true
---

POSIX线程(pthread)是IEEE制定的标准C语言线程编程接口.从技术层面讲,线程是一个独立的指令流,可以被操作系统调度和运行.从开发者角度讲,线程是独立于主程序运行的一个"过程".更直白的说,我们可以想象主程序拥有一些过程,这些过程可以被操作系统调度,并同时地,独立地运行.使用多线程可以提高用户体验,充分利用多核CPU资源.而pthread则有利开发可移植的多线程应用程序.类UNIX系统都支持pthread,而windows系统有自己的一套线程编程接口,但也可以通过MinGW得到对pthread的支持.下面具体介绍了pthread中各个函数.

## pthread_create ##

**原型:**`int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *),void *arg);`
  
  用于创建线程,在进程中创建线程时可以通过参数`attr`指定属性.如果`attr`为NULL时,使用默认属性.函数调用后对属性的修改不会影响线程.当函数执行成功时,线程ID会存储到参数`thread`中.
  
  线程创建时会去执行`start_routine`并附带一个参数`arg`.`start_routine`返回时的效果果就像是在退出状态隐式调用了`pthread_exit`,返回值就是`pthread_exit`的参数.注意主线程的行为方式与此不同,`main`函数返回时效果就像在退出状态隐式调用了`exit`,返回值就是`exit`函数的参数.
  
当线程退出时,线程的存储空间必须被另一个线程通过调用`pthread_join`来回收.或者,对线程调用`pthread_detach`以指示操作系统在退出时自动回收线程的存储空间.函数`pthread_attr_setdetachstate`可以设置`attr`并传递给`pthread_create`用于在创建线程时达到相同的效果.
  
线程的信号状态会被设置为:
  
  * 信号掩码从主创线程继承
  * 新线程的信号集合为空
  
**返回值:**成功时返回0,否则返回错误码.
  
**错误码:**

  * [EAGAIN] 系统资源不足或达到进程拥有线程数的上线
  * [EPERM]  没有权限设置调度策略和调度参数
  * [EINVAL] 参数`attr`的值非法

## pthread_exit ##

**原型:**`void pthread_exit(void *value_ptr);`

函数终止线程并将`value_ptr`所指内容传递给成功调用`pthread_join`的线程.任何已入栈但还未出栈的终止清理程序将会以与入栈相反的顺序出栈和执行.在所有清理程序执行完后如果还有线程数据未释放,相关的线程析构函数会以无序的方式调用.线程终止不会释放任何应用程序可见的进程资源,包括但不限于互斥锁,文件描述符;也不会执行进程级别的清理函数,包括但不限于调用`atexit`例程.

当一个非main函数线程从`start_routine`返回时,会隐式地调用`pthread_exit`.函数的参数会指定为`return`语句返回的值.

如果在清理程序或者线程析构函数中调用`pthread_exit`,系统的行为未定义.清理程序和线程析构函数本身是在显式或隐式调用`pthread_exit`时触发的.

在线程已经终止后,继续访问线程的局部变量的结果是未定义的.因此,不能使用局部变量的指针作为``value_ptr`的值.

当最后一个线程终止后,进程以状态码0退出,行为与调用`exit`函数并以0作为参数一样.

`pthread_exit`函数无法返回到它的调用者,也不会产生错误码.

## pthread_cancel ##

**原型:**`int pthread_cancel(pthread_t thread);`

函数向操作系统请求取消参数`thread`的所指代线程.目标线程的状态和类型决定何时执行取消动作.当取消动作激活,清理函数被调用.当最后一个清理函数返回时,线程相关数据析构函数将被调用,当最后一个析构函数返回时,线程被终止.

目标线程中的取消处理过程相对于`pthread_cancel`的调用线程是异步执行的..

任何线程对目标线程调用`pthread_join`可以得到一个`PTHREAD_CANCELED`状态.`PTHREAD_CANCELED`是类型为`void *`的常量表达式,它的值既不是内存对象的指针,也不是`NULL`.

**返回值:**成功返回0,否则返回一个错误码.

**错误码:** 
  * [ESRCH] 找不到线程ID对应的线程.
  
## 线程属性相关函数 ##

**原型:**
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

线程属性用于为`pthread_cteate`指定参数.一个属性对象可以用在多个`pthread_create`调用中,多次调用之间,可以修改属性对象.

`init`函数用默认值来初始化属性对象`attr`.
`destory`函数销毁属性对象`attr`.
`set`系列函数设置属性对象中的某个属性,具体属性体现在函数的名字中.
`get`系列函数拷贝属性对象中的某个属性到第二个参数所指内存中,具体属性体现在函数的名字中.

**返回值:**成功时返回0.否则返回错误码.

**错误码:**
  
  * [ENOMEM] 内存耗尽
  * [EINVAL] 参数attr的值非法

## phtread_join ##

**原型:**`int pthread_join(pthread_t thread, void **value_ptr);`

函数挂起调用线程直到目标线程终止.

如果成功调用`pthread_join`函数,并且参数`value_ptr`不为空,在`pthread_exit`调用时指定的参数被存储在`value_ptr`中. 此函数返回成功,说明目标线程已经被终止.对同一个线程同时调用此函数的结果是未定义的. 如果调用线程被终止,那么目标线程不会被`detach`(见下节).

**返回值:**成功时返回0.否则返回错误码.

**错误码:**
  
  * [EINVAL] 目标线程不是一个可join的线程
  * [ESRCH] 找不到线程ID对应的线程
  * [EDEADLK] 检测到死锁,或者目标线程ID是调用线程本身

## pthread_detach ##

**原型:**`int pthread_detach(pthread_t thread);`

函数用来指示线程的存储空间可以在线程终止时被回收.如果线程还没有被终止,函数不会触发线程终止.同一个线程对象多次调用`pthread_detach`的结果是未定义的.

**返回值:**成功函数返回0,否则返回错误码.

**错误码:**
  
  * [EINVAL] 目标线程不是一个可连接的线程.
  * [ESRCH] 找不到线程ID对应的线程对象.

## pthread_self ##

**原型:**`pthread_t pthread_self(void);`

函数获取调用线程的ID.

## pthread_equal ##

**原型:**`int pthread_equal(pthread_t t1, pthread_t t2);`

函数比较两个线程ID,如果两个ID指示同一个线程对象则返回这个ID(非零值),否则返回0.

## pthread_once ##

**原型:**`int pthread_once(pthread_once_t *once_control, void (*init_routine)(void));`

进程中任意线程第一个使用给定`once_control`作为参数调用`pthread_once`,将会调用不带参数的`init_routine`.而后续线程的对`pthread_once`的调用将不会触发`init_routine`.从`pthread_once`返回后,能保证`init_routine`已经完成.参数`once_control`用来决定相关联的初始化例程是否已经被调用.

`pthread_once`不是取消点.如果`init_routine`是一个取消点,并且线程处于取消状态,此时的效果就好像`pthread_once`从未调用一样.

如果`once_control`有自动存储期限或者没有用`PTHREAD_ONCE_INIT`初始化,那么`pthrad_once`的调用结果是不可预期的.

**返回值:**函数成功时返回0,否则返回错误码.

## pthread_mutex_init ##

**原型:**`int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);`

函数创建一个新的互斥量,并通过`attr`参数指定其属性,如果`attr`为`NULL`则使用默认属性.

**返回值:**成功返回0,并将互斥量的ID填到参数`mutex`中,否则返回错误码.

**错误码:**

  * [EINVAL] 参数`attr`的值非法
  * [ENOMEM] 进程无法为创建互斥量申请足够的内存
  
## pthread_mutex_destory ##

**原型:**`int pthread_mutex_destroy(pthread_mutex_t *mutex);`

销毁为互斥量分配的空间.

**返回值:**成功返回0,否则返回错误码.

**错误码:**

  * [EINVAL] 参数mutex的值非法
  * [EBUSY] 参数mutex处于锁定状态
  
## 互斥量属性相关函数 ##

```
int pthread_mutexattr_init(pthread_mutexattr_t *attr);
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
int pthread_mutexattr_setprioceiling(pthread_mutexattr_t *attr, int prioceiling);
int pthread_mutexattr_getprioceiling(pthread_mutexattr_t *attr, int *prioceiling);
int pthread_mutexattr_setprotocol(pthread_mutexattr_t *attr, int protocol);
int pthread_mutexattr_getprotocol(pthread_mutexattr_t *attr, int *protocol);
int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
int pthread_mutexattr_gettype(pthread_mutexattr_t *attr, int *type);
```

互斥量属性用来为`pthread_mutex_init`函数提供参数.与线程属性类似,一个属性对象可以用于多个`pthread_mutex_init`函数,在多个调用之间可以修改属性对象.

`init`函数使用默认值初始化参数其参数`attr`;
`destory`销毁`attr`所占内存资源;
`settype`设置属性的类型字段,合法的值包括:
  * `PTHREAD_MUTEX_NORMAL` 这类互斥量不会检查用法上的错误,重复上锁可能导致死锁,锁定的互斥量被其他线程释放时系统行为是未定义的,尝试解锁已经解锁的互斥量也会导致未定义的行为.
  * `PTHREAD_MUTEX_ERRORCHECK` 这类互斥量会检查用法上的错误,并返回错误码.
  * `PTHREAD_MUTEX_RECURSIVE` 这类互斥量允许锁的嵌套.接触全部嵌套的锁才能唤醒其他等待中的线程,如果线程视图解开有另一个线程上锁的互斥量则返回错误码,对已经解开的锁继续解锁也会返回错误码.强烈建议这类锁不要和条件变量一起使用,原因是`pthread_cond_wait`和`pthread_cond_timedwait`隐含解锁行为.
  * `PTHREAD_MUTEX_DEFAULT`与`PTHREAD_MUTEX_NORMAL`相似,是`init`函数使用的默认类型.
`gettype`函数拷贝属性值到第二个参数指定的位置.
`set`系列函数设置在函数名中体现的属性字段.
`get`系列函数获取在函数名中体现的属性字段.

**返回值:**成功时返回0,否则返回错误码.

**错误码:**

  * [ENOMEM] 内存耗尽.
  * [EINVAL] 参数attr的值非法.

## pthread_mutex_lock ##

**原型:**`int pthread_mutex_lock(pthread_mutex_t *mutex);`

函数用来锁定互斥量,如果互斥量已经锁定,调用线程会阻塞指导互斥量再次可用.

**返回值:** 成功返回0,否则返回错误码.

**错误码:**

  * [EINVAL] 参数mutex的值非法
  * [EDEADLK] 检测到死锁
  
## pthread_mutex_trylock ##

**原型:**`int pthread_mutex_trylock(pthread_mutex_t *mutex);`

函数尝试锁定互斥量,如果互斥量已经锁定,函数不会阻塞但是会返回错误码.

**返回值:**成功返回0,否则返回错误码.

**错误码:**

  * [EINVAL] 参数mutex的值非法
  * [EBUSY] 互斥量已经锁定

## pthread_mutex_unlock ##

**原型:**`int pthread_mutex_unlock(pthread_mutex_t *mutex);`

解锁互斥量,如果当前线程持有参数`mutex`的锁,调用函数则会释放这个锁,如果试图释放当前线程未持有的锁,结果是为定义的.

**返回值:**成功返回0,否则返回错误码.

**错误码:**
  
  * [EINVAL] 参数mutex的值非法
  * [EPERM] 当前线程没有持有mutex的锁
  
