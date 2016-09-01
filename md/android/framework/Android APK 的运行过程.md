Android APK 的运行过程
==

**1.** 首先，**ActivityThread** 从 `main()` 函数中开始进行，调用 `prepareMainLooper()` 为 **UI** 线程创建 **Looper** 的同时，也创建一个消息队列（ **MessageQueue** ）。


**2.** 然后，创建一个 **ActivityThread** 对象，在 **ActivityThread** 初始化代码中会创建一个 **H**（ **Handler** ） 对象和一个  **ApplicationThread**（ **Binder** ）对象。其中 **Binder** 负责接收远程 **AMS** 的 **IPC** 调用，接收到调用后，则通过 **Handler** 把消息发送到消息队列（ **MessageQueue** ），**UI** 主线程会异步地从消息队列 （ **MessageQueue** ） 中取出消息并执行相应操作，比如 `start()`、`stop()`、`pause()` 等。


**3.** 接着 **UI** 主线程调用 `Looper.loop()` 方法进入消息循环体，进入后就会不断从消息队列（ **MessageQueue** ）中读取并处理消息。


```java
public final class ActivityThread {

    ...

    final ApplicationThread mAppThread = new ApplicationThread();
    final Looper mLooper = Looper.myLooper();
    final H mH = new H();

    ...

    private class H extends Handler {

      ...

    }

    ...

    // ApplicationThreadNative 继承自 Binder
    private class ApplicationThread extends ApplicationThreadNative {

      ...

    }

    ...

    public static void main(String[] args) {

        ...

        Looper.prepareMainLooper();

        ...

        ActivityThread thread = new ActivityThread();

        ...

        Looper.loop();

        ...

    }

    ...

}  
```

<img src="http://ww2.sinaimg.cn/large/006lPEc9gw1f3p5hm294kj31kw16owjm.jpg" width="760x"/>


**4.** 当 **ActivityThread** 接收到 **AMS** 发送 **start 某个 Activity 的请求** 后，就会创建指定的 **Activity** 对象。**Activity** 又会创建 **PhoneWindow 类 -> DecorView 类 -> 创建相应的 View 或者 ViewGroup**。创建完成后，**Activity** 需要把创建好的界面显示到屏幕上，于是调用 **WindowManager** 类，**WindowManager** 创建一个 **ViewRoot** 对象，实际上创建了 **ViewRoot** 类和 **W** 类，创建 **ViewRoot** 对象后，**WindowManager** 再调用 **WMS** 提供的远程接口完成添加一个窗口并显示到屏幕上。

<img src="http://ww1.sinaimg.cn/large/006lPEc9gw1f3p5i92o82j31kw16ojya.jpg" width="760x"/>

**5.** 接下来，如果在程序界面上操作。**KeyQ 线程** 不断把用户消息存储到 **QueueEvent 队列** 中 **InputDispatcherThread 线程** 逐个取出消息，然后调用 **WMS** 中的相应函数处理该消息。当 **WMS** 发现该消息属于客户端某个窗口时，就会调用相应窗口的 **W** 接口。


**6.** **W** 类 是一个 **Binder**，负责接收 **WMS** 的 **IPC 调用**，并把调用消息传递给 **ViewRoot**，**ViewRoot** 再把消息传递给 **UI** 主线程 **ActivityThread**，**ActivityThread** 解析该消息并做相应的处理。在客户端程序中，首先处理消息的是 **DecorView**，如果是 **DecorView** 不想处理某个消息，则可以将该消息传递给其内部包含在 **子 View 或者 ViewGroup**，如果还没处理，则传递给 **PhoneWindow**，最后再传递给 **Activity**。      
