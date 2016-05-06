Android 系统特点
==

## 1.Android 内核系统

Android 是一个 **Linux** 系统，是基于 **Linux** 内核开发的系统。

## 2. Android 系统与 Linux 系统的区别

**Android 系统使用的 Linux 内核，其实与传统的 Linux 内核并无多大区别。**

**区别：**

- **2.** Android 在传统的 Linux 内核中以模块的形式加入了一些专用的驱动，例如日志驱动 Logger、匿名共享内存驱动 Ashmem、进程间通信驱动 Binder。

- **3.** **Android 系统自己有一套独特的用户空间运行时，也就是应用程序框架。** Android 系统将在传统的 Linux 内核实现的硬件驱动程序划分成了两个模块，一块是**内核实现**，另一块是**用户空间实现**，也就是**硬件抽象层 HAL**。这样划分，**是出于商业考虑，而不是技术考虑**。因为 Linux 内核使用的 GPL 许可协议，驱动全部放在内核实现就意味着需要全部开源代码，用户空间使用的是 Apache License，可以不开源代码。通过这样的方式，就可以保护厂家的商业利益，因为这些代码通常都会包含有硬件的相关参数。

## 3. Android 开源工程

Android 应用程序框架中，包含了大量开源工程。浏览器用的内核 WebKit、管理 Wi-Fi 网络的 wap_supplicant、播放音乐视频的StageFright、Android系统专用的渲染UI的 SurfaceFlinger、播放声音的 AudioFlinger，**实现了最基本的功能，都是用 C/C++ 写的**。


## 4. Android 守护进程服务

上面的 Android 开源工程实现了最基本的功能，这些功能被 Java 层的一些守护进程服务用到。比如：组件管理服务 ActivityManagerService、应用程序安装服务 PackageManagerService、网络连接服务 ConnectivityService 等。


### 5. Android 系统层次结构

>Android 系统 = Android = Linux Kernel + C/C++ Runtime Framework + Java Runtime Framework + Davik Virtual Machine

>调用 Android SDK 提供的一个API后，这个 API 交给 Java Runtime Framework 处理，Java Runtime Framework 又会将这个 API 交给 C/C++ Runtime Framework 处理，最后 C/C++ Runtime Framework 又有可能接着将这个 API 调用交给 Linux 内核来处理。

Java 层只是位于 Android 最上面的一层编程接口。**除了 Android 自带的 SDK 外，还能使用 NDK 绕开 Java Runtime Framework，将 API 交给 C/C++Runtime Framework 处理。**
