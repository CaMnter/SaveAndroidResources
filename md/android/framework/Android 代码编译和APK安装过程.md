Android 代码编译和APK安装过程
==

## 1. Android 代码编译

用 Java 语言开发 Android 程序的话，Java 编译器会将源代码编译成 Java 字节码（ **.class** ）文件。**.class** 文件接着被 **Android SDK** 提供的 **dx 工具** 转化为 **dex** 字节码，最后打包在 APK里的 **classes.dex** 文件中。

### 1.1 Jack (Java Android Compiler Kit) 编译

如果用 Jack 编译的话，上面的 ` .java + 资源 -> 资源 + .class -> .dex` 流程会变为 ` .java + 资源 -> .jack -> .dex ` 流程。

## 2. Android APK 安装过程

**APK** 文件在安装的时候，**Java Runtime Framework** 内的 **PacakgeManagerService** 会对  **APK** 文件进行解析，并且通过 **Socket IPC** 通知 **C/C++ Runtime Framework** 内的 **installd 守护进程** 对 **APK** 里的 **classes.dex** 进行优化，得到另外一个 **classes.odex**  文件。
