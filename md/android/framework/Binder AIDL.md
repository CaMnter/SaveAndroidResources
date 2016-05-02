Binder AIDL
==

## 1. 简介

> AIDL:Android Interface Definition Language,即 Android 接口定义语言。

>什么是AIDL?
>为了使其他的应用程序也可以访问本应用程序提供的服务，Android 系统采用了远程过程调用（Remote Procedure Call，RPC）方式来实现。与很多其他的基于RPC的解决方案一样，Android使用一种接口定义语言（Interface Definition Language，IDL）来公开服务的接口。我们知道4个 Android 应用程序组件中的3个（Activity、BroadcastReceiver 和 ContentProvider）都可以进行跨进程访问，另外一个 Android 应用程序组件 Service 同样可以。因此，可以将这种可以跨进程访问的服务称为AIDL（Android Interface Definition Language）服务。

## 2. 建立 AIDL 服务的步骤

- **1.** Java包目录中建立一个扩展名为 .aidl 的文件。该文件的语法类似于 Java 代码，但会稍有不同。  
 
- **2.** 如果 aidl 文件的内容是正确的，ADT 会自动生成一个Java接口文件（*.java）。  
- **3.** 建立一个服务类（Service的子类）。  
- **4.** 实现由 aidl 文件生成的 Java 接口。
- **5.** 在 AndroidManifest.xml 文件中配置 AIDL 服务，尤其要注意的是，`<action>` 标签中android:name的属性值就是客户端要引用该服务的ID，也就是Intent类的参数值。


## AIDL 原理

**为什么 .aidl 文件生成的 java 代码能够进行 IPC （跨进程通信） 呢？**   
**答：**因为 生成的 java 代码**定义了 Binder 机制中 作为 Server 的 Binder 对象和 Client 中要使用的 Proxy 对象**。 Binder Server 创建后会启动一个隐藏线程，同时会创建 Binder 驱动中的 Binder Server的 远程 mRemote 对象。**mRemote 其实就是一个Proxy 对象** 。Client 通过 这个 Proxy 对象去和 Server 通信。


## 3. AIDL 实例

定义一个 .aidl 文件：

```java
// IPushMessage.aidl
package com.camnter.newlife.aidl;

// Declare any non-default types here with import statements

interface IPushMessage {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);

    String onMessage();

}
```

生成对应的 .java 文件

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.camnter.newlife.aidl;
// Declare any non-default types here with import statements

public interface IPushMessage extends android.os.IInterface {
    /** Local-side IPC implementation stub class. */
    public static abstract class Stub extends android.os.Binder
            implements com.camnter.newlife.aidl.IPushMessage {
        private static final java.lang.String DESCRIPTOR = "com.camnter.newlife.aidl.IPushMessage";


        /** Construct the stub at attach it to the interface. */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }


        /**
         * Cast an IBinder object into an com.camnter.newlife.aidl.IPushMessage interface,
         * generating a proxy if needed.
         */
        public static com.camnter.newlife.aidl.IPushMessage asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.camnter.newlife.aidl.IPushMessage))) {
                return ((com.camnter.newlife.aidl.IPushMessage) iin);
            }
            return new com.camnter.newlife.aidl.IPushMessage.Stub.Proxy(obj);
        }


        @Override public android.os.IBinder asBinder() {
            return this;
        }


        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags)
                throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_basicTypes: {
                    data.enforceInterface(DESCRIPTOR);
                    int _arg0;
                    _arg0 = data.readInt();
                    long _arg1;
                    _arg1 = data.readLong();
                    boolean _arg2;
                    _arg2 = (0 != data.readInt());
                    float _arg3;
                    _arg3 = data.readFloat();
                    double _arg4;
                    _arg4 = data.readDouble();
                    java.lang.String _arg5;
                    _arg5 = data.readString();
                    this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
                    reply.writeNoException();
                    return true;
                }
                case TRANSACTION_onMessage: {
                    data.enforceInterface(DESCRIPTOR);
                    java.lang.String _result = this.onMessage();
                    reply.writeNoException();
                    reply.writeString(_result);
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }


        private static class Proxy implements com.camnter.newlife.aidl.IPushMessage {
            private android.os.IBinder mRemote;


            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }


            @Override public android.os.IBinder asBinder() {
                return mRemote;
            }


            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }


            /**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString)
                    throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(anInt);
                    _data.writeLong(aLong);
                    _data.writeInt(((aBoolean) ? (1) : (0)));
                    _data.writeFloat(aFloat);
                    _data.writeDouble(aDouble);
                    _data.writeString(aString);
                    mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }


            @Override public java.lang.String onMessage() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.lang.String _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_onMessage, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readString();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_onMessage = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString)
            throws android.os.RemoteException;

    public java.lang.String onMessage() throws android.os.RemoteException;
}

```

## 4. 解析 AIDL 生成的 Java 文件

生成的 Java 文件 是**一个接口**。

### 4.1 aidl 文件生成了三个模块
- **1.** 本 `Ixx` 接口，并且定义了 aidl 文件里描述的方法。
- **2.** 静态抽象类 Stub，继承了 `android.os.Binder` ，并且实现了 aidl 生成的 `Ixx` 接口。
- **3.** 静态内部类 Stub.Proxy，也实现了 aidl 生成的 `Ixx` 接口。  
     
### 4.2 Stub
继承了 Binder，实现了我们在 aidl 生成的接口，定义了需要跨进程通信的方法。作为 Server 存在着。   
                 
### 4.3 Stub.Proxy
也实现了 aidl 生成的接口，Cilent 可以与 Proxy 通信，间接地和 Server 端进行了通信。 `private android.os.IBinder mRemote;`  是 Proxy 类 里的唯一属性。Proxy 实现的 aidl 接口方法中，都调用了 `mRemote.transact` 去与 Server 进行通信。
       
### 4.3 Proxy 与 Stub 的 Code 对应
**以上面实例中生成的接口为例：**
Proxy 类定义了 两个 code 去区分接口方法：`TRANSACTION_basicTypes` 和 `TRANSACTION_onMessage`。
同时，Stub 类的 `onTransact` 方法里也有对应 `TRANSACTION_basicTypes` 和 `TRANSACTION_onMessage` code的分发处理。  
          
### 4.4 Stub.onTransact 的作用？
**答：**    
> onTransact 方法主要是在 用户空间 和 内核空间 中进行数据的交换，实现进程间数据的交互。

### 4.5 asInterface 方法将 IBinder 对象转换成对应接口
```java
/**
 * Cast an IBinder object into an com.camnter.newlife.aidl.IPushMessage interface,
 * generating a proxy if needed.
 */
public static com.camnter.newlife.aidl.IPushMessage asInterface(android.os.IBinder obj) {
    if ((obj == null)) {
        return null;
    }
    android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
    if (((iin != null) && (iin instanceof com.camnter.newlife.aidl.IPushMessage))) {
        return ((com.camnter.newlife.aidl.IPushMessage) iin);
    }
    return new Stub.Proxy(obj);
}
``` 
    
    
### 4.6 为什么要用 asInterface 方法将 IBinder 对象进行转换？
**答：** 因为通信的话，**假如当前进程处在 Server 的进程，那么就不需要 Proxy 对象进行通信；假如你当前进程不在 Server 进程，就需要 Proxy 对象进行 IPC 通信**，证明这句话的代码就是上述的，如果 `queryLocalInterface` 查询本地接口，存在的话，就返回该接口（此时就是处在 Server 进程的情况）；没找到，直接 `new` 一个 Proxy 提供 IPC 通信功能。








