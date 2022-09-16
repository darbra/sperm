> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [lief-project.github.io](https://lief-project.github.io/doc/latest/tutorials/09_frida_lief.html)

> LIEF Documentation

In this tutorial we will see how to use Frida gadget on a non-rooted device.

Scripts and materials are available here: [materials](https://github.com/lief-project/tutorials/tree/master/09_Frida_LIEF)

By Romain Thomas - [@rh0main](https://twitter.com/rh0main)

* * *

From the last few years, Frida became the tool of the trade to perform hooking. It supports various platforms and enables to write hooks quickly and dynamically.

Most of the time there are no constraints to use Frida on a rooted device but in some scenario the application to analyze could check its environment.

A technique based on modifying the Dalvik bytecode has been well described by [@ikoz](https://twitter.com/ikoz) in the post “[Using Frida on Android without root](https://koz.io/using-frida-on-android-without-root/)”. In this tutorial we propose a new technique without modifying the Dalvik Bytecode (i.e. `classes.dex`).

Frida Gadget[¶](#frida-gadget "Permalink to this headline")
-----------------------------------------------------------

In the default mode, Frida needs in a first step to inject an _agent_ in the targeted application so that it is in the memory space of the process.

On Android and Linux such injection is done with `ptrace` by attaching or spawning a process and then injecting the agent. Once the agent is injected, it communicates with its server through a pipe.

Some kind of injection require privileges. For example, we can’t use `ptrace` as a _normal_ user. To address this constraint, Frida provides another mode of operation called “embedded”. In this mode the user is responsible to inject the _frida-gadget_ library.

Such injection could be done with:

*   Environment variables: `LD_PRELOAD`, `DYLD_INSERT_LIBRARIES` …
    
*   Using `dlopen`
    
*   In an open-source target, use the linker to link with frida-gadget.
    
*   …
    

For more information about Frida gadget, here is the documentation: [frida-gadget](https://frida.re/docs/gadget/)

Frida & LIEF[¶](#frida-lief "Permalink to this headline")
---------------------------------------------------------

One less known injection technique but quite old is based on modifying the ELF format. It has been well explained by Mayhem in [Phrack](http://phrack.org/issues/61/8.html) [1](#id6) and LIEF provides a user-friendly API [2](#id7) to do it.

To summarize, executable formats include libraries that are linked with executable. We can have list of linked libraries with `ldd` or `readelf` (Unix) or with [elf_reader.py](https://github.com/lief-project/LIEF/blob/master/examples/python/elf_reader.py) (Linux, Windows, OSX):

```
$ python ./elf_reader.py -d /bin/ls

== Dynamic entries ==

|Tag    | Value | Info        |
|NEEDED | 0x1   | libcap.so.2 |
|NEEDED | 0x80  | libc.so.6   |


```

Here `/bin/ls` has two dependencies:

*   `libcap.so.2`
    
*   `libc.so.6`
    

In the loading phase of the executable, the loader iterates over these libraries and map them in the memory space of the process. Once mapped it calls its constructor [3](#id8).

The idea is to add `frida-agent.so` as a dependency of native libraries embedded in the APK.

Adding such dependencies is as simple as:

```
import lief

libnative = lief.parse("libnative.so")
libnative.add_library("libgadget.so") # Injection!
libnative.write("libnative.so")


```

Telegram[¶](#telegram "Permalink to this headline")
---------------------------------------------------

To explain the process, we will inject frida gadget in the Telegram application. It’s an interesting target because:

*   It contains only one native library so the library should be loaded early.
    
*   It shows the reliability of LIEF to modify ELF files
    
*   It’s a real app
    

Regarding the environment, we will use the version `4.8.4-12207` of Telegram (February 18, 2018) on an Android 6.0.1 device with an AArch64 architecture (Samsung Galaxy S6)

### Injection with LIEF[¶](#injection-with-lief "Permalink to this headline")

As explained above, the injection is just a call to the [`lief.ELF.Binary.add_library()`](https://lief-project.github.io/doc/latest/api/python/elf.html#lief.ELF.Binary.add_library "lief.ELF.Binary.add_library") on the `libtmessages.28.so` library.

Prior to the injection `libtmessages.28.so` is linked against the following libraries

```
$ readelf -d ./libtmessages.28.so|grep NEEDED
  0x0000000000000001 (NEEDED) Shared library: [libjnigraphics.so]
  0x0000000000000001 (NEEDED) Shared library: [liblog.so]
  0x0000000000000001 (NEEDED) Shared library: [libz.so]
  0x0000000000000001 (NEEDED) Shared library: [libOpenSLES.so]
  0x0000000000000001 (NEEDED) Shared library: [libEGL.so]
  0x0000000000000001 (NEEDED) Shared library: [libGLESv2.so]
  0x0000000000000001 (NEEDED) Shared library: [libdl.so]
  0x0000000000000001 (NEEDED) Shared library: [libstdc++.so]
  0x0000000000000001 (NEEDED) Shared library: [libm.so]
  0x0000000000000001 (NEEDED) Shared library: [libc.so]


```

After `telegram.add_library("libgadget.so")` we have the new dependency at the first position:

```
$ readelf -d ./libtmessages.28.so|grep NEEDED
  0x0000000000000001 (NEEDED) Shared library: [libgadget.so]
  0x0000000000000001 (NEEDED) Shared library: [libjnigraphics.so]
  0x0000000000000001 (NEEDED) Shared library: [liblog.so]
  0x0000000000000001 (NEEDED) Shared library: [libz.so]
  0x0000000000000001 (NEEDED) Shared library: [libOpenSLES.so]
  0x0000000000000001 (NEEDED) Shared library: [libEGL.so]
  0x0000000000000001 (NEEDED) Shared library: [libGLESv2.so]
  0x0000000000000001 (NEEDED) Shared library: [libdl.so]
  0x0000000000000001 (NEEDED) Shared library: [libstdc++.so]
  0x0000000000000001 (NEEDED) Shared library: [libm.so]
  0x0000000000000001 (NEEDED) Shared library: [libc.so]


```

### Configuration of Frida Gadget[¶](#configuration-of-frida-gadget "Permalink to this headline")

From the documentation, Frida gadget enables to use a configuration file to parametrize the interaction:

*   **Listing**: Interaction is the same as frida-server
    
*   **Script**: Direct interaction with a JS script for which the path is specified in the configuration
    
*   **ScriptDirectory**: Same as _Script_ but for multiple applications and multiple scripts
    

_Listing_ interaction would require `android.permission.INTERNET` permission. We can add such permission by modifying the manifest. Instead, we will use the _Script_ interaction which does not require permission.

The Frida payload will be located in `/data/local/tmp/myscript.js` file. The gadget configuration associated with context is given below

```
{
  "interaction": {
    "type": "script",
    "path": "/data/local/tmp/myscript.js",
    "on_change": "reload"
  }
}


```

Use of configuration file must follow two requirements:

1.  File must have the same name as the gadget library name (e.g. `libgadget.so` and `libgadget.conf`)
    
2.  The configuration file must be located in the **same** directory as the gadget library
    

The second requirement means that after the installation on the device, the gadget library will look for the config file in the `/data/app/org.telegram.messenger-1/lib` directory.

When installing an application, the Android package manager will copy files from the `lib/` directory of the APK only if [4](#id9):

*   It starts with the prefix `lib`
    
*   It ends with the suffix `.so`
    
*   It’s `gdbserver`
    

Frida is aware of these requirements as illustrated in listing below. Hence we can simply add the suffix `.so` to `libgadget.conf`

```
#if ANDROID
  if (!FileUtils.test (config_path, FileTest.EXISTS)) {
    var ext_index = config_path.last_index_of_char ('.');
    if (ext_index != -1) {
      config_path = config_path[0:ext_index] + ".config.so";
    } else {
      config_path = config_path + ".config.so";
    }
  }
#endif


```

[lib/gadget/gadget.vala](https://github.com/frida/frida-core/blob/289a08b237eeab1fb8ec3e2f41ed726de44b5d66/lib/gadget/gadget.vala#L500-L509)

Finally, the `lib` directory of the new Telegram `.apk` has the following structure:

```
$ tree lib
.
└── arm64-v8a
    ├── libgadget.config.so
    ├── libgadget.so
    └── libtmessages.28.so


```

With `libtmessages.28.so` linked with `libgadget.so`

```
$ readelf -d ./arm64-v8a/libtmessages.28.so
  0x0000000000000001 (NEEDED) Shared library: [libgadget.so]
  ...


```

### Run[¶](#run "Permalink to this headline")

Once:

1.  The injection done in `libtmessages.28.so`
    
2.  The gadget library and its configuration placed in the `/lib/ABI` directory
    
3.  The application resigned
    

We can install the repackaged APK `new.apk` and push `myscript.js` in `/data/local/tmp`:

```
$ adb shell install new.apk
$ adb push myscript.js /data/local/tmp
$ adb shell chmod 777 /data/local/tmp/myscript.js


```

The Frida script `myscript.js` used in this tutorial is just a call to the Android log function:

```
'use strict';

console.log("Waiting for Java..");

Java.perform(function () {
  var Log = Java.use("android.util.Log");
  Log.v("frida-lief", "Have fun!");
});


```

myscript.js

Lastly, we can run the telegram application and observe the Android logs:

[![](https://lief-project.github.io/doc/latest/_images/telegram.png)](https://lief-project.github.io/doc/latest/_images/telegram.png)

```
$ adb logcat -s "frida-lief:V"
--------- beginning of system
--------- beginning of main
03-24 17:23:51.908 10243 10243 V frida-lief: Have Fun!


```

Conclusion[¶](#conclusion "Permalink to this headline")
-------------------------------------------------------

With this tutorial we demonstrated how format instrumentation and dynamic instrumentation can be combined.

Here is a quick summary of advantages/disadvantages of this technique

Advantages

*   Doesn’t require rooted device
    
*   Doesn’t depend of frida-server
    
*   Can be used to bypass some anti-frida
    
*   Doesn’t modify `AndroidManifest.xml` and DEX file(s)
    

Disadvantages

*   Require to add files in the APK
    
*   Require that the application have at least one native library
    
*   Hope that the library is loaded early in the application
    

Notes

API

*   [`lief.ELF.Binary.add_library()`](https://lief-project.github.io/doc/latest/api/python/elf.html#lief.ELF.Binary.add_library "lief.ELF.Binary.add_library")