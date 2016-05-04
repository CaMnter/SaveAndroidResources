Looper Handler MessageQueue
==

## 1. 异步消息处理线程


### 1.1 区别

**普通线程：** run() 方法内的代码后线程就结束了。  
**异步消息处理线程：** 线程会进入一个无限循环体，**循环一次就从内部的消息队列中取出一个消息，并回调该消息对应的处理方法**。然后继续循环下去，如果消息队列没有消息了，线程会停止；否则只有有新的消息，就继续循环。  


### 1.2 需求场景      

>  异步消息处理线程其本质上也是一个线程，只不过是被设计成了在一个无限循环体下，不断读取消息的场景。并且有消息则循环，无消息暂停的模式。

**需求场景：**      
- **1.** 任务需要常驻。比如说用于处理用户交互的任务。
- **2.** 任务需要根据外部传递的消息而执行不同的操作。


### 1.3 简单实现

<img src="http://ww2.sinaimg.cn/large/006lPEc9jw1f3il291ls9j31kw16c78d.jpg" width="640x"/>  

- **1.** 每个异步线程内部包含一个消息队列（MessageQueue)，队列中的消息一般采用排队机制
即先到达的消息会先得到处理。

- **2.** 线程的执行体中使用 while ( true ) 进行无限循环，循环体中从消息队列中取出消息，并且根
据消息的来源，回调其对应的消息处理函数。

- **3.** 其他外部线程可以向本线程的消息队列中发送消息，消息队列内部的读/写操作必须进行加锁
即消息队列不能同时进行读/ 写操作。

## 2. Android 中的异步消息处理线程

<img src="http://ww4.sinaimg.cn/large/006lPEc9jw1f3il7024jdj31kw16odm7.jpg" width="640x"/>   

在线程内部有一个或多个 Handler 对象，外部程序通过该 Handler 对象向线程发送异步消息，消
息经由 Handler 传递到 MessageQueue 对象中。线程内部只能包含一个 MessageQueue 对象，线
程主执行函数中从 MessageQueue 中读取消息，并回调 Handler 对象中的回调函数 handleMessage()。   

## 3. 线程局部存储

**变量的常见作用域：**

- **方法内部变量：** 作用域就是该方法内，每次调用该方法时，变量都会重新回到初始值。

- **类内部的变量：** 其作用域是该类所产生的对象，只要对象没有销毁，则对象内部的变量值
一直保持。

- **类内部静态变量：** 其作用域是整个过程，只要在该进程中，则该变量的值就一直保持，无
论使用该类构造过多少个对象，该变量只有一个赋值，并一直保持。进程中哪个线程引用该变量，其值总是相同的，因为在编译器内部为静态变量分配了单独的内存空间。

**线程局部存储：**
> 同一个线程引用变量值相同，不同线程引用则变量值不相同。



## 4. Looper

**Looper 的作用：**

- **1.** 调用静态方法 `prepare()` 为该线程创建一个 `消息队列` 。

- **2.** 调用 `loop()`， 线程进入无限循环的异步消息处理流程，从消息队列内读取消息，并处理消息。


### 4.1 prepare()

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
...
...
...
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

创建一个 Looper 并将，Looper 存储在 ThreadLocal 内。所以，**相同线程 访问的 Looper 是一样的，不同线程访问的 Looper 是不一样的。**

### 4.2 Looper 构造方法

由上面的 `new Looper(quitAllowed)` 方法。可以看出，调用了 Looper 的构造方法。

```java
private Looper(boolean quitAllowed) {  
    mQueue = new MessageQueue(quitAllowed);  
    mThread = Thread.currentThread();  
}
```

创建一个 `MessageQueue` 。

### 4.3 loop()

```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

**主要操作：**

- **1.** `myLooper()` 拿到当前线程存储的 `Looper` 对象。

- **2.** 拿到 `MessageQueue` 后，进入到 `for (;;)` 无限循环内。

- **3.** 再逐个通过 `queue.next()` 取出 `Message`。

- **4.** 通过 `msg.target.dispatchMessage(msg)` 不难发现 `msg.target` 是一个Handler，执行了该 `msg` 的处理。

- **5.** 最后，`msg.recycleUnchecked();` 对消息的资源进行回收。对象占用的系统资源。因为 `Message` 类内部使用了一个数据池去保存 `Message` 对象，从而避免不停地创建和删除  `Message` 类对象。因此，每次处理完该消息后，需要将该 `Message` 对象表明为空闲，以便 `Message` 对象可以被重用。
