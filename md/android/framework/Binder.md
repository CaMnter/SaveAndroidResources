Binder
==

> Binder 机制 则是 Android 系统实现进程间通信（IPC）的机制之一。


<br/>

## 1. 背景

Android 系统启动时候，**Zygote 进程 孵化出第一个子进程叫 SystemServer**。在 SystemServer 进程中，很多系统提供的服务，比如 ActivityManagerService,     PowerManagerService 等，都在此进程中的某一条线程上运行。  

用户开发的 App，由于 Android 系统安全的考虑，安装 App 到系统中，会被**分配一个 UID**，并且**运行在一个 独立的进程 里**。   

由于 **Linux 系统用户安全机制**，普通 App 是无法访问系统服务的。


<br/>

## 2. Binder 机制四大概念

**Binder 机制四大概念**：

- **Server：**提供服务。

- **Client：**获取服务。

- **ServiceManager：** 和 Zygote 进程一样，属于 Android系统的守护进程之一。**作用：** 辅助 Android 系统去维护许多 Service 列表。**形象化：** 就像一个电话本，想要找到某人（ Service ），就必须找到电话本。

- **Binder 驱动：** 一段运行在内核空间的代码，整个 Binder 机制的核心，Binder 才能实现进程间的通信。


<br/>

## 3. Binder 机制四大概念的 运行空间

**Linux 内核空间：** 只有 **Binder 驱动**。

**Linux 用户空间：**   

- **Server**

- **Client**

- **ServiceManager**


<br/>

## 4. Binder  驱动

**整个 Binder 机制的核心，Binder 才能实现进程间的通信。**  

> Binder 驱动跟硬件没有联系，是一段运行在 内核空间 的代码，通过 /dev/binder 的文件在 内核空间 和 用户空间 之间 读写数据。

Server, Client 和 ServiceManager 是运行在 **用户空间** 中的 **不同的进程**，彼此是 **不能互相访问** 的。所以，**不同进程之间的数据传递，只能通过内核空间来操作。由于只有 Binder 驱动在内核空间，只能由 Binder 驱动来进行，所以 Binder 驱动是 Binder 机制核心。**

**Binder  驱动主要工作职责：**   

- **Binder 节点的建立。**

- **Binder 进程间传递。**

- **Binder 引用的计数管理。**

- **进程间传输数据包。**

进程间通信的大部分工作都是通过 `open()`, `mmap()` 和 `ioctl()`等文件操作，在内核空间与用户空间中进进出出，来实现的。


<br/>

## 5. ServiceManager

> ServiceManager 是 Android 系统的一个守护进程。

**Android 启动 ServiceManager 进程的时候：**

- **1.** 打开 "/dev/binder" 设备文件，进行内存映射。  

- **2.** 该进程将 **BINDER_SET_CONTEXT_MGR** 命令传给 Binder 驱动，Binder 驱动 就会为其在内核空间中创建一个节点（binder_node），这个节点就是 `binder_context_mgr_node`，也就是ServiceManager的 Binder 实体，**将该进程变为上下文管理者（ServiceManager）** 。   

- **3.** 进入无限循环，等待 Client 请求。

在整个 Android 系统中，只会有一个 `binder_context_mgr_node` ，所以只会有一个 ServiceManager 的进程。  

### 5.1 扩展-句柄

> 句柄理解上就是一个区分不同服务的标识。在 Android 系统中，只有跨进程的通信，才会用到句柄这个术语，因为没有办法直接访问。

### 5.2 句柄与引用的区别

一个是远程访问（跨进程），一个是本地访问（ 相同进程 ）。

每个进程在系统中的都有定义好的句柄，**ServiceManager 的句柄为 0**。


<br/>

## 6. Binder 机制的实现

### 6.1 Binder Server

- **1.** Binder 驱动 在 **内核空间** 里创建一个 Binder 节点，属于 **Server 进程**。   

- **2.** Binder 驱动 为节点分配一个句柄，将 **句柄** 和 **Name** 发送给 **ServiceManager** 实现注册。

**Binder Server 创建后会启动一个 隐藏线程，同时会创建 Binder 驱动 中的 Binder 服务端 的 远程 mRemote 对象，提供给客户端调用。**


### 6.2 Binder Client   

- **1.** Binder Client 通过 transact 方法， Client 线程进入到 Binder 驱动 中。

- **2.** 将 **想要服务 + 句柄=0**，封装成数据包，打开 **dev/binder** 设备文件，发给 Binder 驱动。

- **3.** Binder 驱动 接收到句柄0，找到 ServiceManager。

- **4.** ServiceManager 根据 **服务名称->句柄的 mapping** 找到服务句柄。

- **5.** ServiceManager 将想要服务的句柄返回。

**然后调用 Binder 驱动 中对应 Binder Server 的 mRemote 对象去访问 Binder Server，Binder Server 向 Binder驱动 发送一个notify消息，从而 Client 线程从 Binder 驱动代码区返回到 Server 代码区。**
