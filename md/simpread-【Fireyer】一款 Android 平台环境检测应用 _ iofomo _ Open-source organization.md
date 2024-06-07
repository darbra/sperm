> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.iofomo.com](https://www.iofomo.com/blog/fireyer/#1--%E8%AF%B4%E6%98%8E)

> Fireyer 是为了校验我们的虚拟化环境构建是否存在缺陷，可以保障我们的每次更新的产品质量，提升开发效率。

> Fireyer 是为了校验我们的虚拟化环境构建是否存在缺陷，可以保障我们的每次更新的产品质量，提升开发效率。

![](https://www.iofomo.com/assets/images/topic-b5db416a0366ebe71d4ff1580d2e62a5.jpg)

项目已开源：

[☞ Github：https://www.github.com/iofomo/fireyer ☜](https://www.github.com/iofomo/fireyer)　

**如果您也喜欢 Fireyer，别忘了给我们点个星。**

### 1. 说明[​](#1--说明 "1.  说明的直接链接")

`fire` + `eyer` = `Fireyer`（火眼），`Fireyer`项目是我们在做虚拟化沙箱产品过程中的内部副产品。目的是为了校验我们的虚拟化环境构建是否存在漏洞，在内部作为我们产品的黑白检测工具应用，可以保障我们的每次更新的产品质量，提升开发效率。对于开发沙箱，虚拟化等相关场景产品的伙伴也可以提升开发效率，快速验证功能稳定性。`Fireyer`的检测项还在不断完善中，后续会持续同步更新。

由于我们的虚拟化产品是普通主流机型，因此`Fireyer`主要用于在正常系统环境下，检测应用被重打包（或重签名），容器环境（免安装加载运行），虚拟机（将`Android`系统变成普通应用）的通用个人手机场景。`Fireyer`当前并不适用于定制`ROM`，或刷入`Magisk`，或`ROOT`的环境检测（当然由于技术的相关性，其中某些检测项可能生效，但并非针对性用例），但也在我们后续的迭代计划中。

### 2. 如何使用[​](#2--如何使用 "2.  如何使用的直接链接")

`Fireyer`项目的主要目的是为了提升我们产品的稳定性，并非为了应用的强对抗，只是为了保证正常的应用行为运行稳定。

我们自测的方法：

1.  在正常的应用环境中，点击`单元测试【原始环境】`，`Fireyer`会将运行完成的用例数据格式化保存在系统的剪切板中备用。
2.  在虚拟的测试环境中，点击`单元测试【虚拟环境】`，`Fireyer`会从系统的剪切板中获取测试数据，然后与当前运行用例结果进行对比，最终得到测试验证的目的。

![](https://www.iofomo.com/assets/images/02-0fd2131d04eb034f168de6461d579045.jpg)

### 3. 系统调用实现[​](#3--系统调用实现 "3.  系统调用实现的直接链接")

为了可以实现对`inline`和`got`表的拦截检测，我们需要实现一些基本函数的系统调用，如：

```
int open(const char *pathname, int flags, ...);
int close(int fd);
int stat(const char* path, struct stat* buf);
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
ssize_t readlink(const char *path, char *buf, size_t bufsiz);


```

系统调用的方式如何实现呢，有个简单的办法就是将手机里面的`libc.so`库导出来（这里导出的 64 位的库），然后用`ida`打开，查看对应函数的实现，如`open`的实现如下：

![](https://www.iofomo.com/assets/images/01-93ae45d538508ace2698ef948713b2e6.jpg)

这样我们得到`openat`在 64 位系统上的系统调用的实现方式：

```
__attribute__((__naked__)) int svc_openat() { 
  __asm__ volatile("mov x15, x8\n" 
    "ldr x8, =0x38\n"
    "svc #0\n"
    "mov x8, x15\n"
    "bx lr"
  );
}


```

**优势：**

通过自实现系统调用函数，可以在关键的地方和正常的函数调用进行对比，从而达到识别的目的，不管是基于`got`表还是`inline`的拦截。

**对抗：**

如何对抗该检测，则可以使用应用级`trace`拦截。

### 4. 代理拦截和检测[​](#4--代理拦截和检测 "4.  代理拦截和检测的直接链接")

拦截是利用`Java`的`Proxy`模块完成的，如：

```
package java.lang.reflect;

public class Proxy implements java.io.Serializable {
       public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h);
}


```

代理后，原对象实例被更换为代理后的对象，当应用使用调用接口方法后，即可回调。

普通的检测方法：

```
package java.lang.reflect;

public class Proxy implements java.io.Serializable {
    public static boolean isProxyClass(Class<?> cl) {
        return Proxy.class.isAssignableFrom(cl) && proxyClassCache.containsValue(cl);
    }
}


```

通常对方会自己调用`native`方法实现创建代理对象，而不使用`Proxy`类，如：

```
package java.lang.reflect;

public class Proxy implements java.io.Serializable {
       private static native Class<?> generateProxy(String name, Class<?>[] interfaces,
                                                 ClassLoader loader, Method[] methods,
                                                 Class<?>[][] exceptions);
}


```

那我们依然可以通过对比该对象的类名进行识别，如：

```
// 正常类
android.view.IWindowSession$Stub$Proxy
// 代理后的类
android.view.IWindowSession$Stub$Proxy$Proxy


```

### 5. Binder 拦截和检测[​](#5--binder拦截和检测 "5.  Binder拦截和检测的直接链接")

很多时候我们与`Service`的通信可能被劫持，而拦截`Binder`通信最简单的方法就是接口代理。由于`Android`服务的`Binder`通信框架的数据解析和序列化都是基于接口：

```
/**
 * /frameworks/base/core/java/android/app/IActivityManager.aidl
 */
interface IActivityManager {
  // ...
}

/**
 * /frameworks/base/core/java/android/content/pm/IPackageManager.aidl
 */
interface IPackageManager {
  // ...
}

public interface Parcelable {
       public interface Creator<T> {
        public T createFromParcel(Parcel source);
        public T[] newArray(int size);
    }
}


```

1、我们可以获取对应服务的`Binder`对象，检测是否已经被代理。

```
Object obj = ReflectUtils.getStaticFieldValue("android.app.ActivityManager", "IActivityManagerSingleton");
Object inst = ReflectUtils.getFieldValue(obj, "mInstance");
if (Proxy.isProxyClass(inst.getClass())) {
    // TODO
}


```

2、可能面临基于底层`Binder`拦截的方案，如之前分享的开源项目：[【Android】深入 Binder 底层拦截](https://www.iofomo.com/blog/Binder)。

则整个解析不经过`Java`层，上层无法检测，但是底层解析有个很大的弊端就是对于复杂的`Binder`通信，如参数或返回值为`Bundle`，`Intent`，`ApplicationInfo`，`PackageInfo`时，解析逻辑非常复杂，要做到兼容性好，通常会调用上层的代码进行解析。

### 6. 完整性检测[​](#6--完整性检测 "6.  完整性检测的直接链接")

#### 6.1 签名校验[​](#61-签名校验 "6.1 签名校验的直接链接")

1、通过系统的`PackageManagerService`提供的返回值（太简单，非小白略过）。

```
PackageInfo pi = getContext().getPackageManager().getPackageInfo(getPackageName(), PackageManager.GET_SIGNING_CERTIFICATES);
pi.signingInfo;// TODO


```

2、通过解析本地文件。（太简单，非小白略过）。

```
PackageInfo pi = getContext().getPackageManager().getPackageArchiveInfo(mPackageInfo.applicationInfo.sourceDir, PackageManager.GET_SIGNING_CERTIFICATES);
pi.signingInfo;// TODO


```

以上两种方法都可以通过接口代理方式替换`SigningInfo.CREATOR`，来完成`PackageInfo.signingInfo`的拦截和伪装。

```
// source code
public final class SigningInfo implements Parcelable {

    public static final @android.annotation.NonNull Parcelable.Creator<SigningInfo> CREATOR =
            new Parcelable.Creator<SigningInfo>() {
        @Override
        public SigningInfo createFromParcel(Parcel source) {
            return new SigningInfo(source);
        }

        @Override
        public SigningInfo[] newArray(int size) {
            return new SigningInfo[size];
        }
    };
}


```

#### 6.2 属性检测[​](#62-属性检测 "6.2 属性检测的直接链接")

1、校验`Application`完整性。

```
<application
    android:theme="@ref/0x7f120289" ----------------------------------------- 是否被替换
    android:label="@ref/0x7f0d0001" ----------------------------------------- 是否被替换
    android:icon="@ref/0x7f0d0001" ------------------------------------------ 是否被替换
    android: --------------------------------- 是否被替换
    android:persistent="false"
    android:allowBackup="false"
    android:debuggable="false" ---------------------------------------------- 是否被开启
    android:hardwareAccelerated="true"
    android:largeHeap="true"
    android:supportsRtl="false"
    android:extractNativeLibs="true"
    android:usesCleartextTraffic="true"
    android:networkSecurityConfig="@ref/0x7f150051"
    android:appComponentFactory="androidx.core.app.CoreComponentFactory" ---- 是否替换
    android:requestLegacyExternalStorage="true"
    android:allowNativeHeapPointerTagging="false"
    android:preserveLegacyExternalStorage="true"
    >
</application>


```

2、检测`permission`。

3、检测四大组件：`activity`、`activity-alias`、`service`、`provider`、`receiver`。

4、检测`meta-data`。

### 7. 运行环境[​](#7--运行环境 "7.  运行环境的直接链接")

#### 7.1 检测隐藏 API 权限[​](#71-检测隐藏api权限 "7.1 检测隐藏API权限的直接链接")

很多应用篡改目的是为了完成某些功能，时常涉及隐藏接口的调用（从`9.0`后），会将一些模块的保护权限解除，因此我们需要对一些常用的模块做检测。

```
if (classFind("android.app.ActivityThread")) break;
// /libcore/dalvik/src/main/java/dalvik/system/DexPathList.java
if (classFind("dalvik.system.DexPathList")) break;
// /frameworks/base/core/java/android/app/LoadedApk.java
if (classFind("android.app.LoadedApk")) break;
// /frameworks/base/core/java/android/app/IActivityManager.aidl
if (classFind("android.app.IActivityManager")) break;
// /frameworks/base/core/java/android/content/pm/IPackageManager.aidl
if (classFind("android.content.pm.IPackageManager")) break;


```

通过一些类的反射访问（该类在`Android`开发者网站上说明，源码有`@hide`标注），可以确认当前运行环境的隐藏`API`是否已经被解除。该方案很难被修复，如果完全无感知需要虚拟化框架在调用时设置隐藏`API`策略，提前缓存好目标`class`，`method`和`field`，然后再恢复，但如此则虚拟化环境内存消耗和初始化性能则会受到很大影响。

#### 7.2 检测目录[​](#72-检测目录 "7.2 检测目录的直接链接")

通过系统调用实现查看当前私有目录下是否存在未知文件和目录，某些虚拟化环境会在应用目录提前存放了一些数据文件。

#### 7.3 检测调用栈[​](#73-检测调用栈 "7.3 检测调用栈的直接链接")

在某些关键函数回调中进行调用栈的检测。

1.  如：`AppComopentFactory`的初始化回调。
2.  如：`Application`的初始化回调。
3.  如：`ActivityThread$H`的`callback`回调。

检测的方式：

1.  直接上层的`Thread.dumpStack`获取。虚拟化环境可以通过对`native`的函数拦截伪装。
2.  通过低层`libunwind`库获取对应的函数名和库信息。虚拟化环境可以通过对`getcontext`的拦截进行伪装。

#### 7.4 检测线程[​](#74-检测线程 "7.4 检测线程的直接链接")

`Java`层检测：

```
public static void getAllThreadsInfo() {
    Map<Thread, StackTraceElement[]> allThreads = Thread.getAllStackTraces();
    for (Map.Entry<Thread, StackTraceElement[]> entry : allThreads.entrySet()) {
        Thread thread = entry.getKey();
        StackTraceElement[] stackTrace = entry.getValue();
        // Got thread id and names
    }
}


```

但某些实现会拦截`native`层函数调用进行伪装，因此我们需要遍历线程目录（使用自实现的系统调用函数访问）

```
void getAllThreadsInfo() {
    char threadName[128];
    DIR* taskDir = opendir("/proc/self/task");
    if (taskDir != nullptr) {
        struct dirent* entry;
        while ((entry = svc_readdir(taskDir)) != nullptr) {
            if (entry->d_type == DT_DIR && strcmp(entry->d_name, ".") != 0 && strcmp(entry->d_name, "..") != 0) {
                pid_t threadId = atoi(entry->d_name);
                if (pthread_getname_np(pthread_t(threadId), threadName, sizeof(threadName)) == 0) {
                             
                }
            }
        }
        closedir(taskDir);
    }
}


```

#### 7.5 C 进程检测[​](#75-c进程检测 "7.5 C进程检测的直接链接")

增加采用`C`程序命令的方式采集信息。如：

1.  `ls ${dir}`。
2.  `cat ${file}`。
3.  自己实现`c`程序对主进程进行信息采集。

应对方案：

1.  拦截进程`execve`函数，对调用`c`程序命令的参数进行修正。
2.  拦截进程`execve`函数，对即将`fork`的子进程，向子进程的`envp`环境变量注入预加载库，从而实现对`C`程序内部函数调用的拦截。

#### 7.6 maps 检测[​](#76-maps检测 "7.6 maps检测的直接链接")

`maps`检测实现，使用系统调用函数对`/proc/self/maps`中的内容进行校验。

1.  校验`maps`是否有第三方库的加载痕迹。
2.  校验`base.apk`路径是否合法。
3.  校验`dex`库是否被篡改。

该检测可以被`Trace`方案拦截，并映射至修正的新的`maps`文件，达到虚拟化伪装的目的。

#### 7.7 注入库检测[​](#77-注入库检测 "7.7 注入库检测的直接链接")

当前进程可能被加载了执行代码（如：`dex`或`lib`），因此我们通过查找本进程的`maps`进行识别（使用自实现的系统调用函数访问）。

```
int fd = svc_open("proc/self/maps", "r");
if (0 <= fd) {
  char buffer[1024];
  svc_read(fd, buffer, sizeof(buffer);
  svc_close(fd);
}


```

而对方可能会直接采用内存方式加载`dex`或`apk`，如：

```
/**
 * /libcore/dalvik/src/main/java/dalvik/system/DexPathList.java
 **/
public final class DexPathList {
       public static Element[] makeInMemoryDexElements(ByteBuffer[] dexFiles,
            List<IOException> suppressedExceptions) {
        Element[] elements = new Element[dexFiles.length];
        int elementPos = 0;
        for (ByteBuffer buf : dexFiles) {
            try {
                DexFile dex = new DexFile(new ByteBuffer[] { buf }, /* classLoader */ null,
                        /* dexElements */ null);
                elements[elementPos++] = new Element(dex);
            } catch (IOException suppressed) {
                System.logE("Unable to load dex file: " + buf, suppressed);
                suppressedExceptions.add(suppressed);
            }
        }
        if (elementPos != elements.length) {
            elements = Arrays.copyOf(elements, elementPos);
        }
        return elements;
    }
}


```

同样也会通过先在将`lib`库加载到内存，然后通过从内存加载`lib`的方式实现，这样在`maps`中就不会留下的文件目录痕迹。

```
FILE* tempFile = tmpfile();

const char* tempFileName = fileno(tempFile);
void* libHandle = dlopen(tempFileName, RTLD_NOW);
if (libHandle != nullptr) {
    
    dlclose(libHandle);
}
unlink(tempFileName);


```

以上情况，我们需要对`maps`中的地址区间的内容进行进一步的识别。

#### 7.8 Trace 检测[​](#78-trace检测 "7.8 Trace检测的直接链接")

`Trace`检测实现，当前使用系统调用函数对`/proc/self/status`中的`TracerPid:`字段进行简单校验。后面会有单独的文章分享如何构建`Trace`进程互相检测实现。