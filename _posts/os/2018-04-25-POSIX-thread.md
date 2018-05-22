---
layout: post
title:  "POSIX线程编程手册"
summary: POSIX（Portable Operating System Interface of UNIX）表示可移植操作系统接口，POSIX标准定义了操作系统应该为应用程序提供的接口标准，是IEEE为要在各种UNIX操作系统上运行的软件而定义的一系列API标准的总称，线程接口是标准的一部分。
featured-img: os
# comments: true
---
POSIX线程(pthread)是IEEE制定的标准C语言线程编程接口.

从技术层面讲,线程是一个独立的指令流,可以被操作系统调度和运行.从开发者角度讲,线程是独立于主程序运行的一个"过程".更直白的说,我们可以想象主程序拥有一些过程,这些过程可以被操作系统调度,并同时地,独立地运行.

使用多线程可以提高用户体验,充分利用多核CPU资源.而pthread则有利开发可移植的多线程应用程序.

类UNIX系统都支持pthread,而windows系统有自己的一套线程编程接口,但也可以通过MinGW得到对pthread的支持.本文基于freeBSD系统帮助手册具体介绍pthread中各个函数的用法.

## 线程基本操作 ##
### pthread_create ###
**原型:**
```
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine)(void *),void *arg);
```
  
**说明:** 
  * 函数用于创建线程,在进程中创建线程时可以通过参数`attr`指定属性.如果`attr`为NULL时,使用默认属性.函数调用后对属性的修改不会影响线程.当函数执行成功时,线程ID会存储到参数`thread`中.
  * 线程创建时会去执行`start_routine`并附带一个参数`arg`.`start_routine`返回时的效果就像是在退出状态隐式调用了`pthread_exit`,返回值就是`pthread_exit`的参数.注意主线程的行为方式与此不同,`main`函数返回时效果就像在退出状态隐式调用了`exit`,返回值就是`exit`函数的参数.
  * 当线程退出时,线程的存储空间必须被另一个线程通过调用`pthread_join`来回收.或者,对线程调用`pthread_detach`以指示操作系统在退出时自动回收线程的存储空间.函数`pthread_attr_setdetachstate`可以设置`attr`并传递给`pthread_create`用于在创建线程时达到相同的效果.
  
**返回值:**
  * 成功时返回0,否则返回错误码.
  
**错误码:**
  * [EAGAIN] 系统资源不足或达到进程拥有线程数的上线.
  * [EPERM]  没有权限设置调度策略和调度参数.
  * [EINVAL] 参数`attr`的值非法.

### pthread_exit ###

**原型:**
```
void pthread_exit(void *value_ptr);
```

**说明:** 
  * 函数终止线程并将`value_ptr`所指内容传递给成功调用`pthread_join`的线程.任何已入栈但还未出栈的终止清理程序将会以与入栈相反的顺序出栈和执行.在所有清理程序执行完后如果还有线程数据未释放,相关的线程析构函数会以无序的方式调用.线程终止不会释放任何应用程序可见的进程资源,包括但不限于互斥锁,文件描述符;也不会执行进程级别的清理函数,包括但不限于调用`atexit`例程.
  * 当一个非main函数线程从`start_routine`返回时,会隐式地调用`pthread_exit`.函数的参数会指定为`return`语句返回的值.
  * 如果在清理程序或者线程析构函数中调用`pthread_exit`,系统的行为未定义.清理程序和线程析构函数本身是在显式或隐式调用`pthread_exit`时触发的.
  * 在线程已经终止后,继续访问线程的局部变量的结果是未定义的.因此,不能使用局部变量的指针作为``value_ptr`的值.
  * 当最后一个线程终止后,进程以状态码0退出,行为与调用`exit`函数并以0作为参数一样.
  * `pthread_exit`函数无法返回到它的调用者,也不会产生错误码.

### pthread_cancel ###

**原型:**
```
int pthread_cancel(pthread_t thread);
```

**说明:**
  * 函数向操作系统请求取消参数`thread`的所指代线程.目标线程的状态和类型决定何时执行取消动作.当取消动作激活,清理函数被调用.当最后一个清理函数返回时,线程相关数据析构函数将被调用,当最后一个析构函数返回时,线程被终止.
  * 目标线程中的取消处理过程相对于`pthread_cancel`的调用线程是异步执行的.
  * 任何线程对目标线程调用`pthread_join`可以得到一个`PTHREAD_CANCELED`状态.`PTHREAD_CANCELED`是类型为`void *`的常量表达式,它的值既不是内存对象的指针,也不是`NULL`.

**返回值:**
  * 成功返回0,否则返回一个错误码.

**错误码:** 
  * [ESRCH] 找不到线程ID对应的线程.
  
### 线程属性相关函数 ###

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

**说明:**
  * 线程属性用于为`pthread_cteate`指定参数.一个属性对象可以用在多个`pthread_create`调用中,多次调用之间,可以修改属性对象.
  * `init`函数用默认值来初始化属性对象`attr`.
  * `destory`函数销毁属性对象`attr`.
  * `set`系列函数设置属性对象中的某个属性,具体属性体现在函数的名字中.
  * `get`系列函数拷贝属性对象中的某个属性到第二个参数所指内存中,具体属性体现在函数的名字中.

**返回值:**
  * 成功时返回0.否则返回错误码.

**错误码:**
  * [ENOMEM] 内存耗尽
  * [EINVAL] 参数attr的值非法

### phtread_join ###

**原型:**
```
int pthread_join(pthread_t thread, void **value_ptr);
```

**说明:**
  * 函数挂起调用线程直到目标线程终止.如果成功调用`pthread_join`函数,并且参数`value_ptr`不为空,在`pthread_exit`调用时指定的参数被存储在`value_ptr`中. 此函数返回成功,说明目标线程已经被终止.对同一个线程同时调用此函数的结果是未定义的. 如果调用线程被终止,那么目标线程不会被`detach`(见下节).

**返回值:**
  * 成功时返回0.否则返回错误码.

**错误码:**
  * [EINVAL] 目标线程不是一个可join的线程
  * [ESRCH] 找不到线程ID对应的线程
  * [EDEADLK] 检测到死锁,或者目标线程ID是调用线程本身

### pthread_detach ###

**原型:**
```
int pthread_detach(pthread_t thread);
```

**说明:**
  * 函数用来指示线程的存储空间可以在线程终止时被回收.如果线程还没有被终止,函数不会触发线程终止.同一个线程对象多次调用`pthread_detach`的结果是未定义的.

**返回值:**
  * 成功函数返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 目标线程不是一个可连接的线程.
  * [ESRCH] 找不到线程ID对应的线程对象.

### pthread_self ###

**原型:**
```
pthread_t pthread_self(void);
```

**说明:**
  * 函数获取调用线程的ID.

### pthread_equal ###

**原型:**
```
int pthread_equal(pthread_t t1, pthread_t t2);
```

**说明:**
  * 函数比较两个线程ID,如果两个ID指示同一个线程对象则返回这个ID(非零值),否则返回0.

### pthread_once ###

**原型:**
```
int pthread_once(pthread_once_t *once_control, void (*init_routine)(void));
```

**说明:**
  * 进程中任意线程第一个使用给定`once_control`作为参数调用`pthread_once`,将会调用不带参数的`init_routine`.而后续线程的对`pthread_once`的调用将不会触发`init_routine`.从`pthread_once`返回后,能保证`init_routine`已经完成.参数`once_control`用来决定相关联的初始化例程是否已经被调用.
  * `pthread_once`不是取消点.如果`init_routine`是一个取消点,并且线程处于取消状态,此时的效果就好像`pthread_once`从未调用一样.
  * 如果`once_control`有自动存储期限或者没有用`PTHREAD_ONCE_INIT`初始化,那么`pthrad_once`的调用结果是不可预期的.

**返回值:**
  * 函数成功时返回0,否则返回错误码.

### pthread_mutex_init ###

**原型:**
```
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
```

**说明:**
  * 函数创建一个新的互斥量,并通过`attr`参数指定其属性,如果`attr`为`NULL`则使用默认属性.

**返回值:**
  * 成功返回0,并将互斥量的ID填到参数`mutex`中,否则返回错误码.

**错误码:**
  * [EINVAL] 参数`attr`的值非法
  * [ENOMEM] 进程无法为创建互斥量申请足够的内存  
  
## 互斥量相关函数 ##
### pthread_mutex_destory ###

**原型:**
```
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

**说明:**
  * 销毁为互斥量分配的空间.

**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 参数mutex的值非法.
  * [EBUSY] 参数mutex处于锁定状态.
  
### 互斥量属性相关函数 ###

**原型:**
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

**说明:**
  * 互斥量属性用来为`pthread_mutex_init`函数提供参数.与线程属性类似,一个属性对象可以用于多个`pthread_mutex_init`函数,在多个调用之间可以修改属性对象.
  * `init`函数使用默认值初始化参数其参数`attr`;
  * `destory`销毁`attr`所占内存资源;
  * `settype`设置属性的类型字段,合法的值包括:
    + `PTHREAD_MUTEX_NORMAL` 这类互斥量不会检查用法上的错误,重复上锁可能导致死锁,锁定的互斥量被其他线程释放时系统行为是未定义的,尝试解锁已经解锁的互斥量也会导致未定义的行为.
    + `PTHREAD_MUTEX_ERRORCHECK` 这类互斥量会检查用法上的错误,并返回错误码.
    + `PTHREAD_MUTEX_RECURSIVE` 这类互斥量允许锁的嵌套.接触全部嵌套的锁才能唤醒其他等待中的线程,如果线程视图解开有另一个线程上锁的互斥量则返回错误码,对已经解开的锁继续解锁也会返回错误码.强烈建议这类锁不要和条件变量一起使用,原因是`pthread_cond_wait`和`pthread_cond_timedwait`隐含解锁行为.
    + `PTHREAD_MUTEX_DEFAULT`与`PTHREAD_MUTEX_NORMAL`相似,是`init`函数使用的默认类型.
  * `gettype`函数拷贝属性值到第二个参数指定的位置.
  * `set`系列函数设置在函数名中体现的属性字段.
  * `get`系列函数获取在函数名中体现的属性字段.

**返回值:**
  * 成功时返回0,否则返回错误码.

**错误码:**
  * [ENOMEM] 内存耗尽.
  * [EINVAL] 参数attr的值非法.

### pthread_mutex_lock ###

**原型:**
```
int pthread_mutex_lock(pthread_mutex_t *mutex);
```

**说明:**
  * 函数用来锁定互斥量,如果互斥量已经锁定,调用线程会阻塞指导互斥量再次可用.

**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 参数mutex的值非法
  * [EDEADLK] 检测到死锁
  
### pthread_mutex_trylock ###

**原型:**
```
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```

**说明:**
  * 函数尝试锁定互斥量,如果互斥量已经锁定,函数不会阻塞但是会返回错误码.

**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 参数mutex的值非法
  * [EBUSY] 互斥量已经锁定

### pthread_mutex_unlock ###

**原型:**
```
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

**说明:**
 * 函数解锁互斥量,如果当前线程持有参数`mutex`的锁,调用函数则会释放这个锁,如果试图释放当前线程未持有的锁,结果是为定义的.

**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 参数mutex的值非法
  * [EPERM] 当前线程没有持有mutex的锁

## 条件量 ##
### pthread_cond_init ###

**原型:**
```
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr);
```

**说明:**
  * 函数根据属性参数`attr`创建一个新的条件量,如果`attr`为`NULL`,则使用默认属性来创建.

**返回值:**
  * 成功返回0,并将新的条件量id存入参数`cond`所指位置,否则返回错误码.

**错误码:**
  * [EINVAL] 参数`attr`的值非法
  * [ENOMEM] 进程无法分配足够的内存来创建条件量
  * [EAGAIN] 系统暂时没有足够的资源创建条件量
  
### pthread_cond_destroy ###

**原型:**
```
int pthread_cond_destroy(pthread_cond_t *cond);
```

**说明:**
  * 函数释放创建条件量`cond`时分配的资源.

**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 参数`cond`的值非法
  * [EBUSY] 条件量被其他线程锁定
  
## 条件量属性相关函数 ##

**原型:**
```
int pthread_condattr_init(pthread_condattr_t *attr);
int pthread_condattr_destroy(pthread_condattr_t *attr);
```

**说明:**
  * 条件量属性用来为`pthread_cond_init`指定参数.FreeBSD的条件量实现不支持任何非默认的属性,所以这些函数不是很有用,但POSIX要求提供这些函数.

**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:**
  * [EINVAL] 参数`attr`的值非法.
  * [ENOMEM] 内存耗尽.

## 读写锁 ##
读写锁把对共享资源的访问者划分成读者和写者，读者只对共享资源进行读访问，写者则需要对共享资源进行写操作。多处理器环境中,读写锁能提高并发性能，它允许同时有多个读者来访问共享资源。写者是排他性的，一个读写锁同时只能有一个写者或多个读者（与CPU数相关），但不能同时既有读者又有写者。

### pthread_rwlock_init ###

**原型:**
```
int pthread_rwlock_init(pthread_rwlock_t *lock, const pthread_rwlockattr_t *attr);
```

**说明:**
  * 函数用参数`attr`来初始化一个读写锁.如果`attr`为NULL,使用默认读写锁属性来创建.
  * 对已经初始化过的读写锁再次调用初始化的结果是未定义的.

**返回值:** 成功时返回0,否则返回错误码.

**错误码:**
  * [EAGAIN] 系统缺少必要的资源(除内存以外的)
  * [ENOMEM] 内存不足
  * [EPERM] 调用者没有足够的权限进行此操作
  * [EBUSY] 重复初始化且前一次的初始化的锁未释放
  * [EINVAL] 参数`attr`的值非法

## pthread_rwlock_destory ##

**原型:** `int pthread_rwlock_destroy(pthread_rwlock_t *lock);`

**说明:** 函数用来销毁创建的读写锁.

**返回值:** 成功返回0,否则返回错误码.

**错误码:**
  * [EPERM] 调用者没有权限进行此操作 
  * [EBUSY] 系统检测到参数`lock`处于是锁定状态
  * [EINVAL] 参数`lock`的值非法

## 获取读锁 ##
**原型:**
```
int pthread_rwlock_rdlock(pthread_rwlock_t *lock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *lock);
```
**说明:**
  * `pthread_rwlock_rdlock`函数获取一个读锁,在没有线程正持有写锁,并且没有线程阻塞在写锁上时,调用线程立即获得读锁,否则线程阻塞直到读锁可用.
  * `pthread_rwlock_tryrdlock`函数的行为类似,但是在锁无法获得时,不会阻塞线程.
  * 一个线程可能同时持有多个相同的读锁,解锁时应当对每个持有的读锁调用`pthread_rwlock_unlock`.
  * 调用线程持有写锁的同时再请求读锁的结果是未定义的.
  * 为了防止写进程"饿死",写线程优先于读线程.
  
**返回值:**
  * 成功返回0,否则返回错误码.

**错误码:** 
  * [EBUSY] 由于写锁被持有而无法获得读锁
  * [EAGAIN] 由于达到了最大读锁个数无法获得读锁
  
## 获取写锁 ##

**原型:**
```
int pthread_rwlock_wrlock(pthread_rwlock_t *lock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *lock);
```
**说明:**
  * `pthread_rwlock_wrlock`函数阻塞线程直到写锁可用.
  * `pthread_rwlock_trywrlock`函数在写锁不可获得时不会阻塞线程.
  * 在调用线程已经持有锁时再次调用函数的结果是未定义的.

**返回值:**
    * 成功返回0,失败返回错误码.

**错误码:**
  * [EBUSY] 调用线程无法获得写锁
  * [EDEADLK] 调用线程已经持有了读锁或写锁.
  * [EINVAL] 参数`lock`的值非法
  * [ENOMEM] 内存不足
  
## pthread_rwlock_unlock ##
**原型:**
```
int pthread_rwlock_unlock(pthread_rwlock_t *lock);
```

**说明:**
  * 释放已获取的读锁或写锁.

**返回值:**
  * 成功返回0,失败返回错误码.

**错误码:**
  * [EINVAL] 参数`lock`的值非法.
  * [EPERM] 当前线程没有持有任何读锁或写锁.

## 线程数据 ##
每一个线程都可以独立存储和使用线程数据,就好像各个线程拥有自己的全局变量一样.

## pthread_key_create ##

**原型:**
```
int pthread_key_create(pthread_key_t *key, void (*destructor)(void *));
```

**说明:** 
  * 函数创建所有线程都可见的线程数据关键字.关键字是一个不透明的对象,用来指示线程数据的位置.尽管关键字对所有进程都一样,但是绑定到关键字的值由各个线程负责维护并且在整个调用线程的生命周期中存在.
  * 创建关键字时,所有活动线程中该关键字的值都为NULL.创建线程时,所有已创建的关键字的值都为NULL.
  * 每个关键字都可以指定一个可选的析构函数.在线程退出时,如果关键字有一个对应的非空析构函数指针,并且线程有一个非空的值关联到关键字,那么析构函数以该值为唯一参数被调用.当线程中存在不止一个析构函数时,析构函数的调用时无序的.
  * 在所有析构函数调用完成后,仍然存在一些非空的值关联了析构函数,那么系统重复调用过程.重复次数达到`PTHREAD_DESTRUCTOR_ITERATIONS`规定的次数时停止调用析构函数.
  
**返回值:**
  * 成功返回0,并将新创建的关键字保存到key所指内存.否则返回错误码.

**错误码:**
  * [EAGAIN] 系统缺少足够的资源来创建关键字,或者关键字个数达到`PTHREAD_KEYS_MAX`规定的上限.
  * [ENOMEM] 没有足够的内存来创建关键字.

## pthread_key_delete ##

**原型:**
```
int pthread_key_delete(pthread_key_t key);
```

**说明:** 
  * 函数删除一个已经创建的线程数据关键字,关联到关键字线程数据不必为NULL.由应用程序负责释放所有存储空间并执行与已删除关键字有关的数据结构和线程数据的清理动作.清理动作可以在`pthread_key_delete`调用之前执行.试图再次使用已经删除的关键字会产生无法预期的结果.
  * `pthread_key_delete`函数可以在析构函数中调用,析构函数不是由`pthread_key_delete`触发的.线程退出时,与关键字关联的析构函数将不会被调用.
  
**返回值:**
  * 成功返回0,失败返回错误码.

**错误码:**
  * [EINVAL] 参数key的值非法
  
## pthread_setspecific ##  

**原型:**
```
int pthread_setspecific(pthread_key_t key, const void *value);
```

**说明:** 
  * 将线程数据关联的关键字.不同的线程可以绑定不同的值到相同关键字.值通常是指向动态分配的内存块,其中保存了调用线程会使用的数据.
  * 函数对未初始化的关键字或者已经释放的关键字进行操作的结果是未定义的.
  * 函数可以在线程数据析构函数中调用,但这可能导致数据丢失或死循环.
  
**返回值:**
  * 成功返回0,失败返回错误码.

**错误码:** 
  * [ENOMEM] 内存不足
  * [EINVAL] 参数`key`的值非法
  
## pthread_getspecific ##

**原型:**
```
void * pthread_getspecific(pthread_key_t key);
```

**说明:**
  * 函数返回当前线程绑定到关键字的值.
  * 函数对未初始化或已经释放的关键字进行操作会产生未定义的结果.
  * 函数可以在线程数据的析构函数中调用.
  
**返回值:**
  * 成功返回线程数据的指针,失败返回NULL.
