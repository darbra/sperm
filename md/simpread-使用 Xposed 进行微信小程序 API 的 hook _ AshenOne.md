> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [ashenone66.cn](https://ashenone66.cn/2022/03/24/shi-yong-xposed-jin-xing-wei-xin-xiao-cheng-xu-api-de-hook/)

> 大 A 的个人博客，用于记录学习过程

[](#前言 "前言")前言
--------------

  上一篇文章讲了安卓的虚拟定位相关的内容，最后编写了一个 frida 脚本来对 Framework 层的 API 进行 hook 实现虚拟定位。但是有几点局限性：

> 1.  强制 disable WIFI 和基站定位使用 GPS 定位在某些情况下无法 work
> 2.  使用 frida 进行 hook 意味着必须搭配 PC 使用，难以完成持久化的 hook

frida 虽然确实调试起来相当方便，但是 Xposed 由于能够安装在用户手机上实现持久化的 hook，至今受到很多人的青睐，特别是类似虚拟定位的功能，还是使用 Xposed 作为最终实现比较方便。另外，对于微信小程序的`wx.getLocation` API，使用上篇文章中的虚拟定位方法是无法成功的，原因是这个 API 在关闭基站和 WIFI 定位后就不能正常工作。因此，本文将以该 API 作为用例，介绍如何使用 Xposed 来对微信小程序的 js API 进行 hook。

[](#背景知识 "背景知识")背景知识
--------------------

  众所周知，Xposed 主要用于安卓 Java 层的 Hook，而微信小程序则是由 JS 编写的，显然无法直接进行 hook。安卓的有一个 WebView 的组件能够用于网页的解析和 js 的执行，并且提供了 JSBridge 可以支持 js 代码调用安卓的 java 代码，微信小程序正是以此为基础开发了它的微信小程序框架，微信小程序特有的 API 则是由这个框架中的 WxJsApiBridge 提供的，因此以`wx.`开头的 API 都能在这个框架中找到对应的 Java 代码，所以我们虽然不能直接 hook js 代码，但是我们可以通过 hook 这些 js api 对应的 Java 代码来实现微信小程序 api 的 hook。

[](#Frida调试 "Frida调试")Frida 调试
------------------------------

  在编写 Xposed 插件前，首先先使用 Frida 进行逆向分析以及 hook 调试，确保功能能够实现后在用 Xposed 编写插件，毕竟 Xposed 插件调试起来还是不如 Frida 方便。

  首先我们要知道，js 代码中的`getLocation`字符串一定会在 java 层中出现，因此在 jeb 反编译完微信以后，直接搜索该字符串即可。当然，得到的结果肯定不止一个，一个一个调试排除后，确定真正主要的相关类是`com.tencent.mm.plugin.appbrand.jsapi.m.n`（8.0.19 版本，由于混淆不同版本可能名称不一样）。其实在`com.tencent.mm.plugin.appbrand.jsapi`路径下，我们可以看到很多以`JsApixxxxx`的命名格式的命名的类，这些都是微信小程序 API 对应的 Java 类，他们都有一个`NANE`字段，这个值就是`wx.xxx`中的`xxx`，比如`wx.getLocation`对应的 java 类的`NAME`值就是`getLocation`，smali 代码为`.field public static final NAME:String = "getLocation"`。所以以后我们要找哪个 API 直接在 jeb 的 bytecode 里搜索`NAME:String = "xxxx"`就行了。

  定位到具体的类以后，我们可以用 Objection 来 hook 整个类来观察这个类中函数的调用情况，以此发现主要的函数。不过在 hook 之前，需要注意的是，微信小程序一般会以新进程的方式启动，其进程名为`com.tencent.mm:appbrand0`（不确定这个编号 0 是否固定）。因此，如果直接用`frida -U com.tencent.mm -l xxx`或者`objection -g com.tencent.mm explore`来 hook 的话，是无法看到函数调用的，因为你 hook 的进程不是微信小程序的进程而是微信本体的进程。所以我们要指定 pid 来进行 hook，可以使用`dumpsys activity top | grep ACTIVITY`来得到；也可以使用`frida -UF -l xxx`来 hook 当前最顶层的 Activity。对于 Xposed 则没有这个问题，只需指定微信的包名就会自动 hook 上所有的子进程。

结合动态测试的函数调用结果，随便浏览一下被调用的函数的代码，看到了一个主要函数代码如下：

java

```
@Override  
public final void d(f arg10, JSONObject arg11, int arg12) {
    AppMethodBeat.i(0x23110);
    String v3 = Util.nullAsNil(arg11.optString("type", "wgs84")).trim();
    if(Util.isNullOrNil(v3)) {
        v3 = "wgs84";
    }

    boolean v4 = arg11.optBoolean("altitude", false);
    Log.i("MicroMsg.JsApiGetLocation", "getLocation data:%s", new Object[]{arg11});
    if(!"wgs84".equals(v3) && !"gcj02".equals(v3)) {
        Log.e("MicroMsg.JsApiGetLocation", "doGeoLocation fail, unsupported type = %s", new Object[]{v3});
        HashMap v0 = new HashMap(1);
        v0.put("errCode", Integer.valueOf(-1));
        arg10.callback(arg12, this.m("fail:invalid data", v0));
        AppMethodBeat.o(0x23110);
        return;
    }

    if(!n.u(arg10)) {
        HashMap v0_1 = new HashMap(1);
        v0_1.put("errCode", Integer.valueOf(-2));
        arg10.callback(arg12, this.m("fail:system permission denied", v0_1));
        AppMethodBeat.o(0x23110);
        return;
    }

    this.w(arg10);
    Bundle v7 = this.f(arg10, arg11);
    com.tencent.mm.plugin.appbrand.utils.b.a v6 = (com.tencent.mm.plugin.appbrand.utils.b.a)arg10.T(com.tencent.mm.plugin.appbrand.utils.b.a.class);
    if(v6 != null) {
        v6.a(v3, this.a(arg10, new b() {
            @Override  
            public final void a(int arg8, String arg9, com.tencent.mm.plugin.appbrand.utils.b.a.a arg10) {
                AppMethodBeat.i(0x2310F);
                Log.i("MicroMsg.JsApiGetLocation", "errCode:%d, errStr:%s, location:%s", new Object[]{((int)arg8), arg9, arg10});
                n.this.x(arg10);
                if(arg8 == 0) {
                    HashMap v0 = new HashMap(4);
                    v0.put("type", v3);
                    v0.put("latitude", Double.valueOf(arg10.latitude));
                    v0.put("longitude", Double.valueOf(arg10.longitude));
                    v0.put("speed", Double.valueOf(arg10.huN));
                    v0.put("accuracy", Double.valueOf(arg10.usi));
                    if(v4) {
                        v0.put("altitude", Double.valueOf(arg10.altitude));
                    }

                    v0.put("provider", arg10.provider);
                    v0.put("verticalAccuracy", Integer.valueOf(0));
                    v0.put("horizontalAccuracy", Double.valueOf(arg10.usi));
                    if(!Util.isNullOrNil(arg10.buildingId)) {
                        v0.put("buildingId", arg10.buildingId);
                        v0.put("floorName", arg10.floorName);
                    }

                    v0.put("indoorLocationType", Integer.valueOf(arg10.usj));
                    v0.put("direction", Float.valueOf(arg10.usk));
                    v0.put("steps", Double.valueOf(arg10.usl));
                    arg10.callback(arg12, n.this.m("ok", v0));
                    AppMethodBeat.o(0x2310F);
                    return;
                }

                HashMap v0_1 = new HashMap(1);
                v0_1.put("errCode", Integer.valueOf(arg8));
                arg10.callback(arg12, n.this.m("fail:".concat(String.valueOf(arg9)), v0_1));
                AppMethodBeat.o(0x2310F);
            }
        }), v7);
    }

    AppMethodBeat.o(0x23110);
}

```

  可以看到经度纬度的值是从`com.tencent.mm.plugin.appbrand.utils.b.a$a arg10`这个对象中返回来的，因此我们可以直接 hook 相关的函数。但是这样有一个问题就是，由于目标类的类名被混淆了，所以如果 app 版本更新，hook 的代码可能失效，我们要想办法 hook 一个更加底层的不被混淆的类，具体的分析结果就省略了，最终我定位到了一个比较好的类`com.tencent.map.geolocation.sapp.TencentLocationManager`，其中的`requestSingleFreshLocation`函数在每次调用`wx.getLocation`函数时会被调用

java

```
public final int requestSingleFreshLocation(TencentLocationRequest arg8, TencentLocationListener arg9, Looper arg10, boolean arg11) {
       AppMethodBeat.i(0x337F6);
       if(arg9 != null) {
           if(arg10 != null) {
               int v0 = this.mInitStatus;
               if(v0 > 0) {
                   AppMethodBeat.o(0x337F6);
                   return v0;
               }
               ......

```

传参时传了个`com.tencent.map.geolocation.sapp.TencentLocationListener`对象

java

```
package com.tencent.map.geolocation.sapp;

public interface TencentLocationListener {
    public static final String CELL = "cell";
    public static final String GPS = "gps";
    @Deprecated
    public static final String RADIO = "radio";
    public static final int STATUS_DENIED = 2;
    public static final int STATUS_DISABLED = 0;
    public static final int STATUS_ENABLED = 1;
    public static final int STATUS_GPS_AVAILABLE = 3;
    public static final int STATUS_GPS_UNAVAILABLE = 4;
    public static final int STATUS_LOCATION_SWITCH_OFF = 5;
    public static final int STATUS_UNKNOWN = -1;
    public static final String WIFI = "wifi";

    void onLocationChanged(TencentLocation arg1, int arg2, String arg3);

    void onStatusUpdate(String arg1, int arg2, String arg3);
}

```

`TencentLocationListener`的回调函数`onLocationChanged`的第一个参数为`com.tencent.map.geolocation.sapp.TencentLocation`这个接口中就有`getLatitude()`和`getLongitude()`，当然我们不能直接 hook 接口，这是没有意义的，我们要 hook 这个接口的具体的实现类中的对应函数才行。思路就是先 hook `requestSingleFreshLocation`，在调用之前通过`getClass()`获取其第二参数的对象类型，然后 hook 这个类的`onLocationChanged`函数，同样在其调用之前得到其第一参数的对象类型，最后 hook 这个类的`getLatitude()`和`getLongitude()`即可。这里就不给 Frida 版本的代码了，直接上 Xposed 的代码

[](#Xposed模块编写 "Xposed模块编写")Xposed 模块编写
---------------------------------------

  Xposed 模块的架子怎么搭就不介绍了，网上很多教程，这里主要讲下模块编写时踩的坑。直接使用`lpparam.classloader`来 hook 的话，发现对于安卓自带的函数能够成功 hook，但是对于微信自己特有的函数却没法 hook 成功，表现为没有报错找不到类或者方法，但是就是没有函数调用。这个问题我尝试过很多方法来解决，更换 xposed 版本、使用 lsposed 和 edxposed、换个函数 hook、排除子进程 hook 的问题等，都失败了，最后参考网上其他的微信 hook 模块的代码，先 hook `Application`的`attach`函数来获取`Context`，在从`Context`中获取`Classloader`，然后使用这个`Classloader`来完成 hook 终于成功了。上代码：

Application hook：

java

```
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
    if(!lpparam.packageName.equals("com.tencent.mm"))return;

    try{
        XposedHelpers.findAndHookMethod(Application.class,
                "attach",
                Context.class,
                new XC_MethodHook() {
                    @Override
                    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                        super.afterHookedMethod(param);
                        Context context = (Context)param.args[0];
                        ClassLoader classLoader = context.getClassLoader();
                        HookLocation(classLoader);
                    }
                });
    }catch (Throwable e){
        XposedBridge.log(e);
    }

}

```

Location hook:

java

```
private static void HookLocation(ClassLoader classLoader) throws ClassNotFoundException {
    double latitude = 0.0
    double longitude = 0.0

    Class clazz = classLoader.loadClass(
            "com.tencent.map.geolocation.sapp.TencentLocationManager");
    XposedHelpers.findAndHookMethod(clazz, "requestSingleFreshLocation",
            "com.tencent.map.geolocation.sapp.TencentLocationRequest",
            "com.tencent.map.geolocation.sapp.TencentLocationListener",
            "android.os.Looper",boolean.class, new XC_MethodHook() {
                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    XposedBridge.log("requestSingleFreshLocation");
                    Class tencentLocationListenerClass = param.args[1].getClass();
                    XposedHelpers.findAndHookMethod(tencentLocationListenerClass,
                            "onLocationChanged",
                            "com.tencent.map.geolocation.sapp.TencentLocation",
                            int.class, String.class, new XC_MethodHook() {
                                @Override
                                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                                    Class tencentLocation = param.args[0].getClass();
                                    XposedHelpers.findAndHookMethod(tencentLocation,
                                            "getLatitude",
                                            new XC_MethodHook() {
                                                @Override
                                                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                                                    param.setResult(latitude);
                                                }
                                            });
                                    XposedHelpers.findAndHookMethod(tencentLocation,
                                            "getLongitude",
                                            new XC_MethodHook() {
                                                @Override
                                                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                                                    param.setResult(longitude);
                                                }
                                            });


                                }
                            });
                }
            }
    );
}

```