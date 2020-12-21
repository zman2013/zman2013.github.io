---
title: Android apps share FileDescriptor
date: 2020-11-29 21:41:24
tags: 'share fd'
---

Android进程间传递文件的FD，进而实现一个App访问另一个App私有目录下的文件，该方式不能跳过SE拦截。访问非本进程的文件，官网推荐的做法是使用[Content Uri](https://developer.android.com/training/secure-file-sharing)授权，无论设计还是开发都要比share fd的方式麻烦。

## Java-Client + Java-Service
```java
package example;

interface IShareFdService {

    void shareFd(in ParcelFileDescriptor fd);

}
```
如果是Java应用，整个故事到此结束了。

## Java-Client + C-Service
直接手写c binder service，写法如下:
```c++
  status_t IFdService::onTransact(uint32_t code, const Parcel& data,
    Parcel* reply, uint32_t flags){

      switch(code){
        case FD: {
          // 校验接口
          data.checkInterface(this);
          // 读取标志位
          int ff = data.readInt32();
          fprintf(stdout,"ff %d\n", ff);
          // 读取javaClient传来的fd
          int fd = data.readParcelFileDescriptor();
          // 复制fd，binder传来的fd会在方法结束时自动释放
          int newFd = dup(fd);
          
          processFd(newFd);

          break;
        }
      }
      return NO_ERROR;
    }
```

## 根据aidl生成c代码
1. 在Android源码目录下，创建目录vendor/service/share-fd
2. 在share-fd目录下，创建文件Android.mk，内容如下：
```shell
#Android.mk
LOCAL_PATH := $(my-dir)

$(info 'exec - share-fd-service')
include $(CLEAR_VARS)
LOCAL_MODULE := share-fd-service
LOCAL_MULTILIB:= 64
LOCAL_SRC_FILES := ./IShareFdService.aidl 
LOCAL_STATIC_LIBRARIES := libhwbinder libbase libbinder libutils liblog libcutils libvndksupport
LOCAL_AIDL_INCLUDES := $(LOCAL_PATH)
include $(BUILD_SHARED_LIBRARY)
```
3. 在share-fd目录下，创建目录example，并加入aidl文件IShareFdService.aidl
```java
// 此aidl与生成java代码的文件不同，主要是因为未找到办法处理ParcelFileDescriptor，从aidl-cpp源码分析像是不支持
package example;

interface IShareFdService {

    void shareFd(int fd);

}
```
4. 在android源码目录，执行命令:
```shell
. build/envsetup.sh
lunch 9
```
5. 在share-fd目录下，执行: `mm`
6. 生成的源文件
```
.h: out/target/product/xxx/obj/SHARED_LIBRARIES/share-fd-service_intermediates/aidl-generated/include
.cpp: out/target/product/xxx/obj/SHARED_LIBRARIES/share-fd-service_intermediates/aidl-generated/src/dotdot/example/IShareFdService.cpp
```
7. 修改文件IShareFdService.cpp
```cpp
// 找到函数onTransact，修改其中读取fd的代码如下:
_aidl_ret_status = _aidl_data.readInt32(&in_fd);
+ in_fd = _aidl_data.readParcelFileDescriptor();
+ in_fd = dup(in_fd);
```

## C端代码
```cpp
  ShareFdService service = new ShareFdService();

  sp<ProcessState> proc(ProcessState::self());
  sp<IServiceManager> sm = defaultServiceManager();
  // add service
  sm->addService(String16("com.example.aidl.IShareFdService"), service);

  // 查看ServiceManager注册的所有service
  Vector<String16> listServices = sm->listServices();
  Vector<String16>::iterator it = listServices.begin();
  for (; it != listServices.end(); ++it)
  {
    String8 name = String8((*it));
    fprintf(stdout, "StartNodeService1,Name=%s\n", name.string());
  }
```

## Java端代码
如何绑定c端注册binder service？
```java
try {
    Class localClass = Class.forName("android.os.ServiceManager");
    Method getService = localClass.getMethod("getService", new Class[] { String.class });
    if (getService != null) {
        Log.i(TAG, "try to bind IShareFdService");
        Object result = getService.invoke(localClass, new Object[] { "com.example.aidl.IShareFdService" };
        
        if (result != null) {
            
            IBinder binder = (IBinder) result;
            Log.i(TAG, "connected, isBinder alive: " + binder.isBinderAlive());

            IShareFdService service = IShareFdService.Stub.asInterface(binder);

            ParcelFileDescriptor fd = ParcelFileDescriptor.open(file, MODE_WRITE_ONLY);
            service.shareFd(fd);
        }
    }
} catch (Exception e) {
    Log.e(TAG, "error", e);
}
```

## 注意点
### 1. 环境配置
Java App访问C Service需要配置SE策略或者直接关闭SE，Java App需要设置为android.uid.system

### 2. Android关闭、启用SE
```shell
adb shell setenforce 0 # 设置成permissive 模式
adb shell setenforce 1 # 设置成enforce 模式
adb shell getenforce   # 查看当前状态
```

### 3. Android App签名
platform.x509.pem,platform.pk8存放在: android/build/target/product/security/
signapk.jar存放在: android/out/host/linux-x86/framework/
```shell
java -Djava.library.path="android/out/host/linux-x86/lib64" -jar signapk.jar platform.x509.pem platform.pk8 share-fd-unsigned.apk share-fd-signed.apk
```

## Links
[aidl-cpp](https://android.googlesource.com/platform/system/tools/aidl/+/brillo-m10-dev/docs/aidl-cpp.md)

