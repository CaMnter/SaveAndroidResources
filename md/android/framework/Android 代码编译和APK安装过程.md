Android 代码编译和APK安装过程
==

## 1. Android 代码编译

用 Java 语言开发 Android 程序的话，Java 编译器会将源代码编译成 Java 字节码（ **.class** ）文件。**.class** 文件接着被 **Android SDK** 提供的 **dx 工具** 转化为 **dex** 字节码，最后打包在 APK里的 **classes.dex** 文件中。

### 1.1 Jack ( Java Android Compiler Kit ) 编译

如果用 Jack 编译的话，上面的 ` .java + 资源 -> 资源 + .class -> .dex` 流程会变为 ` .java + 资源 -> .jack -> .dex ` 流程。

## 2. Android APK 安装过程

**APK** 文件在安装的时候，**Java Runtime Framework** 内的 **PacakgeManagerService** 会对  **APK** 文件进行解析，并且通过 **Socket IPC** 通知 **C/C++ Runtime Framework** 内的 **installd 守护进程** 对 **APK** 里的 **classes.dex** 进行优化，得到另外一个 **classes.odex**  文件。


### 2.1 APK 从桌面 Launcher 启动的过程

**APK 从桌面 Launcher 启动的过程：** 从 **Launcher** 点击应用图标的时候，**Launcher** 向 **Java Runtime Framework** 内的 **AMS** 发送一个启动应用的请求。**AMS** 通过 **Socket IPC** 向 **C/C++ Runtime Framework** 内的 **zygote** 守护进程 请求创建一个应用进程。应用进程包含都包含一个虚拟机 （ **Dalvik or art** ），并且应用进程创建启动后，就通过这个虚拟机 （ **Dalvik or art** ）去加载 **classes.odex**  文件，这才运行起来。


Android APK 运行过程依赖于虚拟机（ **Dalvik or art** ）。将 **classes.odex** 内的字节码解释成本地机器指令执行。在 APK 运行的过程中，如果通过 `FileInputStream` 或者 `FileOutputStream` 打开一个文件的时候，虚拟机（ **Dalvik or art** ）会找到 **C/C++ Runtime Framework** 里的 C 库 **bionic** 提供的系统接口 `open`，打开指定文件。


### 2.2 应用程序界面的绘制和渲染过程

**应用程序界面的绘制和渲染过程：**

- **1.** 应用程序通过 **SDK** 提供的 **UI** 类向 **Java Runtime Framework** 内的 **WMS** 申请分配一块图形缓冲区。**WMS** 又通过 **Binder IPC** 向 **C/C++ Runtime Framework** 内的 **SurfaceFlinger** 申请分配图形缓冲区。但是，图形缓冲区不是由 **SurfaceFlinger** 分配的，而是由 显示系统 分配的。可能在显存中，也可能在 **GPU** 中。**SurfaceFlinger** 通过 **HAL** （ **C/C++ Runtime Framework** ） 层面的 **Gralloc** 模块向 **Kernel** 内的显卡或者 **GPU** 驱动申请分配真正的图形缓冲区。

- **2.** 获得的 **绘制 UI 所需的 图形缓冲区** 之后，就可以绘制用户指定的 **UI** 了。在应用程序使用的是硬件绘制方式，则通过 **C/C++ Runtime Framework** 内的 **OpenGL** 进行绘制。此时，**SDK** 的 **UI** 类的绘制相关方法就通过 虚拟机（ **Dalvik or art** ）转换成了 **C/C++ Runtime Framework** 内的 **OpenGL** 操作。

- **3.** 应用程序的 UI 绘制完成之后，绘制结果保存在徒刑缓冲区之中。此时，应该 **将该图形缓冲区渲染到手机屏幕上**，还需要 **Binder IPC** 将该图形缓冲区发送给 **C/C++ Runtime Framework** 内的 **SurfaceFlinger**。**SurfaceFlinger** 通过使用 OpenGL 或者 HWComposer 将需要渲染到屏幕上的图形缓冲区进行合成后，得到一个主图形缓冲区。最后这个主图形缓冲区又会被 **SurfaceFlinger** 提交给 **Kernel** 显卡驱动，并且在屏幕上进行显示。
