Android Framework 概述
==

## 1. Framework 框架

> Framework 定义了 客户端组件 和 服务端组件 功能及接口。在 Android 中， "应用程序" 一般是指 ".apk" 程序。

**Android Framework 框架：**

- **服务端**
- **客户端**
- **Linux 驱动**

## 2. 服务端

服务端有经常提到的两个重要的服务：**WindowManagerService（WmS）** 和 **ActivityManagerService（AmS）**。


### 2.1 一些服务的作用

- **WMS 作用：** 为所用应用程序分配窗口，并且管理这些窗口。包括分配窗口的大小，调节各窗口的叠放次序，隐
藏或者显示窗口。

- **AMS 作用：** 管理所用应用程序中的 `Activity`。


### 2.2 一些服务端的消息处理类

- **KeyQ 类：** `WMS` 的 **内部类**，继承于 `KeyInputQueue` 类。`KeyQ` 对象一旦创建了，就立即启动一个线程，该
线程会不断地读取用户的 UI 操作消息，比如按键、触摸屏、`trackball`、鼠标等，**并把这些消息放在消息队列
 QueueEvent 类中**。

- **InputDispatcherThread 类：** InputDispatcherThread 类的对象一旦创建，也会立即启动一个线程，该线程会不断地从 QueueEvent
中取出用户的消息，并进行一定的过滤，过滤后，再将这些消息发送给当前活动的客户端程序中。


## 3. 客户端

**客户端一些重要的类：**

- **ActivityThread 类：** 应用程序的主线程类，**所有的 APK 程序都有且仅有一个 ActivityThread 类，程序的入口就是 ActivityThread 类中的 static main()函数**。

- **Activity 类：** APK 程序的 **最小运行单元**。一个 APK 程序中可以包含多个 Activity 对象，ActivityThread 主类会根据用户操作选择运行哪个 Activity 对象。

- **PhoneWindow 类：** 继承于 Window 类，同时，**PhoneWindow 类内部包含了一个 DecorView 对象 。PhoneWindow 是把一个 FrameLayout 进行了一定的包装，并提供了一组通用的的窗口操作接口。**

- **DecorView 类：** FrameLayout 的子类。并且是 PhoneWindow 中的一个内部类。Decor 的英文是 Decoration，即 "修饰" 的意思，DecorView 就是对普通的 FrameLayout 进行了一定的修饰，比如添加一个通用的 Title bar，并响应特定的按键消息等。

- **Window 类：** 提供了一组通用的窗口（ Window ）操作API，这里的窗口仅仅是程序层面上的，**WMS 所管理的窗口并不是 Window 类，而是一个 View 类或者 ViewGroup 类，一般就是指 DecorView 类，即一个 DecorView 就是 WMS 所管理的一个窗口。Window 是一个 abstract 类型。**

- **ViewRoot（ Handler ） 类：** WMS 管理客户端窗口时，需要通知客户端进行某种操作，这些都是通过异步消息完成的，实现方式就是 Handler，ViewRoot 就是继承于 Handler，其作用主要是接收 WMS 的通知。

- **W（ Binder ） 类：** 继承于 Binder，并且是 **ViewRoot 的一个内部类**。

- **WindowManager 类：** 客户端要申请创建一个窗口，而具体创建窗口的任务是由 WMS 完成的 WindowManager 类就像是一个部门经理，谁有什么需求就告诉它，由它和 WMS 进行交互，客户端不能直接和 WMS 交互。


## 4. Linux 驱动

Linux 驱动 和 Framework 相关的主要包含两个部分。分别是 **SurfaceFlingger（SF）和 Binder**。

**SurfaceFlingger 驱动作用：** 把各个 Surface 显示同一个屏幕上。

**Binder驱动作用：** 提供跨进程的消息传递。
