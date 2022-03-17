> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.pediy.com](https://bbs.pediy.com/thread-267940.htm)

> [原创]android 抓包学习的整理和归纳

最近学习了各种抓包。为了防止忘记。学到的东西必须是整理一波啊。向大佬们看齐。如果有啥地方写的不对，希望大家多多指点

 

抓包主要是针对网络通讯数据，客户端向服务端上报的数据拦截下来。一般都是抓取 http、https、tcp、udp。想要获取到数据包。有多种方式，下面简单列一下。

抓包方式
----

1、hook app 业务层，根据业务代码逻辑找到触发请求的函数，比如按钮触发，或者触发数据上报时的提示框等方式。分析后找到发送数据的地方 hook 打印。

 

**优点：不受 app 的防抓包手段的影响，只要能 hook 到就能抓到包。**

 

**缺点：必须分析 app 找关键点，并且每个不同的请求都要找对应的触发函数，效率太慢。**

 

2、系统框架层的 hook。直接 hook 系统源码发送和接受数据的地方。

 

**优点：可以直接省略掉业务层的分析。因为业务层不论逻辑怎么样最终都是调用系统的或者是第三方的库来进行数据传输。并且通用性更好。基本不用修改就可以抓很多 app 的包。并且可以在这里直接打印堆栈回溯请求触发的函数，提高分析的效率。同样不受防抓包手段影响。**

 

**缺点：hook 出来的抓包数据不便于我们分析和筛选。只能在日志中查找对应的数据，分析数据包会比较繁琐。简单的需求或者是溯源时使用比较好**

 

3、中间人抓包，使用 charles、burp 等抓包工具进行拦截，中间人抓包在 https 请求时，抓包软件的证书在中间冒充服务端接收客服端的请求。然后又冒充客户端，发送请求给服务端。在中间拿到了客户端和服务端的数据。

 

**优点：专业的抓包分析软件，更加友好的分析页面。更加强大的功能，例如可以拦截请求进行改写替换，安装证书可以解析抓到的 https 数据包。支持 vpn 抓包**

 

**缺点：防抓包手段针对的主要目标，例如服务端验证客户端证书，不正确就拒绝访问，我们需要把客户端的证书给 dump 出来，然后让中间人抓包使用指定证书。或者是客户端验证服务端证书。我们需要找到并 hook 去掉验证的代码。无法抓 tcp 和 udp 的包**

 

4、网卡抓包、路由抓包。如 wireshark，科来之类的。

 

**优点：这种方式不受任何限制，并且通杀，绝对能抓到。**

 

**缺点：对于加密的数据没有办法。需要自己进行解密，对于 http 和 https 包的展示不太友好**

 

其中比较通用一点的是系统层的 hook。所以我这里主要针对这个记录下系统层 hook 的方式抓取 http、https、tcp、udp 的数据。

系统层 hook 抓包
-----------

### [](#1、java层的抓包)1、java 层的抓包

首先贴一个网上找的经典的网络模型图。可以看出来。http 包是处于应用层的一种封装，所以我们抓 tcp 包的时候就可以抓到 http 包。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_SN747677AEB6NWH.png)

 

最常听到别人说的一句话就是想要逆向先会正向。我们得先知道如何在 android 中发包。下面我直接封装两种不同库的 http 请求方式和一个 tcp 的请求。然后再顺着代码去分析。看是否能直接在一个地方 hook。将这三种情况的包都能抓到。

 

先贴上用来测试当 tcp 服务端的代码。我在网上随便搜的：https://github.com/fschr/simpletcp.git

 

下面是我拿来测试的 tcp server 代码

```
import queue
from simpletcp.tcpserver import TCPServer
 
def echo(ip, queue, data):
    queue.put(bytes("server recv ","UTF-8")+data)
    print("echo "+data.decode("utf-8"))
 
server = TCPServer("192.168.3.8", 5000, echo)
server.run()

```

然后下面贴上 java 的代码。三种请求的例子

```
public static void GetByHttpURL(final String url) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String resultData="";
                URL connUrl=new URL (url);
                HttpURLConnection urlConn= (HttpURLConnection)connUrl.openConnection ();
                InputStreamReader in=new InputStreamReader (urlConn.getInputStream());
                BufferedReader buffer=new BufferedReader (in);
                String inputLine=null;
                while (((inputLine=buffer.readLine()) !=null)) {
                    resultData+=inputLine+"\n";
                }
                in.close();
                urlConn.disconnect();
                Log.d(TAG, "GetByHttpURL "+resultData);
            } catch (Exception e) {
                Log.d(TAG, "GetByHttpURL error "+e.getMessage());
                e.printStackTrace();
            }
        }
    }).start();
}
 
public static void GetByOkHttp(String url) {
    try {
        OkHttpClient okHttpClient = new OkHttpClient();
        final Request request = new Request.Builder()
                .url(url)
                .get()//默认就是GET请求，可以不写
                .build();
        Call call = okHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Log.d(TAG, "onFailure: ");
            }
 
            @Override
            public void onResponse(Call call, Response response) throws IOException {
                Log.d(TAG, "GetByOkHttp onResponse: " + response.body().string());
            }
        });
    } catch (Exception e) {
        Log.d(TAG, "GetByOkHttp error "+e.getMessage());
        e.printStackTrace();
    }
}
 
 
public static void DoTcp() throws IOException, InterruptedException {
    Log.d(TAG, "DoTcp bind tcp");
    Socket socket = new Socket("192.168.3.8", 5000);
    socket.setSoTimeout(10000);
    //发送数据给服务端
    OutputStream outputStream = socket.getOutputStream();
    outputStream.write("hello,server".getBytes("UTF-8"));
    Thread.sleep(2000);
    socket.shutdownOutput();
    //读取数据
    BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    String line = br.readLine();
    //打印读取到的数据
    Log.d(TAG, "tcp server recv:" + line);
    br.close();
    socket.close();
 
}
 
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    TextView tv = findViewById(R.id.sample_text);
    tv.setText(stringFromJNI());
    new Thread(new Runnable() {
        @Override
        public void run() {
            while(true){
                try {
                    Log.d(TAG, "start send http");
                    GetByOkHttp("http://missking.cc/");
                    GetByHttpURL("http://10.ip138.com");
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }).start();
 
    new Thread(new Runnable() {
        @Override
        public void run() {
 
            try {
                while(true){
                    DoTcp();
                    Thread.sleep(3000);
                }
            } catch (InterruptedException e) {
                Log.d(TAG, "DoTcp error: " + e.getMessage());
                e.printStackTrace();
            } catch (IOException e) {
                Log.d(TAG, "DoTcp error: " + e.getMessage());
                e.printStackTrace();
            }
        }
    }).start();
}

```

上面列出了 HttpURLConnection 请求。OkHttp3 的库来请求。以及一个 tcp 的请求。我们知道 http 最终也是调用的 tcp 连接来传输的。所以我们直接分析 tcp 的调用流程即可。在调用链中。任意一个含有我们传递参数的位置都可以打印出想要的明文信息。但是我们要尽量的找一个更通用一些。能获取到数据更完整的点来 hook。所以我们找到最后调用的 native 的位置。下面列出我们接下来的目标

 

1、找到 tcp 请求发送数据的 native 函数处

 

2、hook 函数打印发送的数据以及目标服务器地址、端口。

 

3、找到 tcp 请求接受数据的 native 函数处

 

4、hook 函数打印接收的数据以及目标服务器地址、端口。

 

目标服务器地址和端口我们直接 hook 了 Socket 的构造函数即可拿到。所以直接分析发送数据部分。

```
//public abstract class OutputStream   //这个OutputStream是一个抽象类，我们需要找到这个对象的真实类型
OutputStream outputStream = socket.getOutputStream();
//找到对应的真实类型后，再找到对应的write函数
outputStream.write("hello,server".getBytes("UTF-8"));
//下面打印下input和output的真实类型
Log.d(TAG,"output class:"+outputStream.getClass());
Log.d(TAG,"input class:"+socket.getInputStream().getClass());

```

然后结果如下。

```
output class:class java.net.SocketOutputStream
input class:class java.net.SocketInputStream

```

我们先看看 write 的调用链。整理后如下

```
//SocketOutputStream.java
//第一步调用到这里
public void write(byte b[]) throws IOException {
      socketWrite(b, 0, b.length);
}
//第二步调用到这个
private void socketWrite(byte b[], int off, int len) throws IOException {
        if (len <= 0 || off < 0 || len > b.length - off) {
            if (len == 0) {
                return;
            }
            throw new ArrayIndexOutOfBoundsException("len == " + len
                    + " off == " + off + " buffer length == " + b.length);
        }
 
        FileDescriptor fd = impl.acquireFD();
        try {
            BlockGuard.getThreadPolicy().onNetwork();
            socketWrite0(fd, b, off, len);
        } catch (SocketException se) {
            if (se instanceof sun.net.ConnectionResetException) {
                impl.setConnectionResetPending();
                se = new SocketException("Connection reset");
            }
            if (impl.isClosedOrPending()) {
                throw new SocketException("Socket closed");
            } else {
                throw se;
            }
        } finally {
            impl.releaseFD();
        }
    }
 
//第三部调用到这里。java部分的就走完了
private native void socketWrite0(FileDescriptor fd, byte[] b, int off,
                                     int len) throws IOException;

```

所以我们发送数据抓包可以通过 hook 函数 socketWrite0 来获取

 

接下来是接受数据 readline 的流程，这里比上面稍微复杂一些。我会列出完整的来龙去脉

```
//先贴是怎么调用的
BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
String line = br.readLine();
 
//所以直接看BufferedReader.java的readLine()
public String readLine() throws IOException {
      return readLine(false);
}
//然后调用了另外一个重载，这个函数比较大。省略掉非关键位置
String readLine(boolean ignoreLF) throws IOException {
        ...
    fill(); 
      ...
}
//然后看看fill的处理
private void fill() throws IOException {
      ...
    in.read(cb, dst, cb.length - dst);
      ...
}
//上面的in是InputStreamReader这个类型所以InputStreamReader.java中看看read
public int read(char cbuf[], int offset, int length) throws IOException {
      return sd.read(cbuf, offset, length);
}
//这个sd是StreamDecoder这个类。继续到里面看看read函数
public int read(char cbuf[], int offset, int length) throws IOException {
      ...
    return n + implRead(cbuf, off, off + len);
    ...
}
//继续看implRead
int implRead(char[] cbuf, int off, int end) throws IOException {
      ...
    int n = readBytes();
    ...
}
//继续看readBytes。这里的in就是我们的SocketInputStream。绕了一圈终于到这里了。继续往后看
private int readBytes() throws IOException {
      ...
    int n = in.read(bb.array(), bb.arrayOffset() + pos, rem);
    ...
}
//下面是SocketInputStream.java中的read函数
public int read(byte b[], int off, int length) throws IOException {
      return read(b, off, length, impl.getTimeout());
}
//继续看调用的另外一个重载
int read(byte b[], int off, int length, int timeout) throws IOException {
        ...
    n = socketRead(fd, b, off, length, timeout);
    ...
}
//继续看socketRead。
private int socketRead(FileDescriptor fd,
                           byte b[], int off, int len,
                           int timeout)
    throws IOException {
    return socketRead0(fd, b, off, len, timeout);
}
//终于到达了最后的native函数。java部分走完了
private native int socketRead0(FileDescriptor fd,
                                   byte b[], int off, int len,
                                   int timeout)throws IOException;

```

根据上面我们的线索。下面开始写代码 hook 一下。最后看看是不是能通抓三个发包的数据。

```
//将数组转换成c++的byte[]。并且hexdump打印结果
function print_bytes(bytes) {
    var buf  = Memory.alloc(bytes.length);
    Memory.writeByteArray(buf, byte_to_ArrayBuffer(bytes));
    console.log(hexdump(buf, {offset: 0, length: bytes.length, header: false, ansi: true}));
}
//将java的数组转换成js的数组
function byte_to_ArrayBuffer(bytes) {
    var size=bytes.length;
    var tmparray = [];
    for (var i = 0; i < size; i++) {
        var val = bytes[i];
        if(val < 0){
            val += 256;
        }
        tmparray[i] = val
    }
    return tmparray;
}
// java.net.Socket$init(ip,port) 获取ip和端口
// socketWrite0(FileDescriptor fd, byte[] b, int off,int len) 获取发送的数据
// socketRead0(FileDescriptor fd,byte b[], int off, int len,int timeout) 获取接受的数据
function hook_tcp(){
    var socketClass= Java.use("java.net.Socket");
    socketClass.$init.overload('java.net.InetAddress', 'int').implementation=function(ip,port){
        console.log("socket$init ",ip,":",port);
        return this.$init(ip,port);
    };
 
    var outputClass=Java.use("java.net.SocketOutputStream");
    outputClass.socketWrite0.implementation=function(fd,buff,off,len){
        console.log("tcp write fd:",fd);
        print_bytes(buff);
        return this.socketWrite0(fd,buff,off,len);
    };
 
    var inputClass=Java.use("java.net.SocketInputStream");
    inputClass.socketRead0.implementation=function(fd,buff,off,len,timeout){
        var res=this.socketRead0(fd,buff,off,len,timeout);
        console.log("tcp read fd:",fd)
        print_bytes(buff);
        return res;
    };
}
function hook_java(){
    Java.perform(function(){
        hook_tcp();
    })
}
function main(){
    hook_java();
}
setImmediate(main);

```

最后跑一下。将结果存储到文本中。搜索一下我们访问的两个地址`http://missking.cc/`以及`http://10.ip138.com`还有 tcp 服务器返回是否有收到

 

![](https://bbs.pediy.com/upload/attach/202106/659397_GRTSD5N7E7BG9FT.png)

 

![](https://bbs.pediy.com/upload/attach/202106/659397_5348N2AVAZWPT5W.png)

 

![](https://bbs.pediy.com/upload/attach/202106/659397_WT743FPHBK9BR62.png)

 

![](https://bbs.pediy.com/upload/attach/202106/659397_YKBKRRMJAAPCCGF.png)

 

上面的日志显示。成功抓到了两种 http 请求和 tcp 的包

### [](#2、jni层抓http包)2、jni 层抓 http 包

有的时候发包的步骤并不是在 java 层中。而是在 jni 层直接就发包了。所以我们即使在 java 层的最深处。也没法抓到。准备一个例子

 

所以我们继续追踪之前的 http 发包例子的后续。socketWrite0 和 socketRead0 的 c 层的调用链。通过编译配置，可以层层筛选找到代码在 libopenjdk.so 中。从当前测试手机中导出这个 so 文件。用 ida 打开后直接搜索就能找到 socketWrite0。

 

socketWrite0 解析的结果后发现调用的是 NET_Send。再往后调用 libc 的 sendto 来进行发送数据

 

![](https://bbs.pediy.com/upload/attach/202106/659397_Z27QFFHTEBA9X84.png)

 

socketRead0 解析的结果后发现调用的是 NET_Read。再往后调用 libc 的 recvfrom 来进行发送数据

 

![](https://bbs.pediy.com/upload/attach/202106/659397_FD6HQMWS2MKXT2Q.png)

 

最后开始写代码来处理。

```
function getSocketData(fd){
    console.log("fd:",fd);
    var socketType=Socket.type(fd)
    if(socketType!=null){
        var res="type:"+socketType+",loadAddress:"+JSON.stringify(Socket.localAddress(fd))+",peerAddress"+JSON.stringify(Socket.peerAddress(fd));
        return res;
    }else{
        return "type:"+socketType;
    }
}
 
function hook_jni_tcp(){
    var sendtoPtr=Module.getExportByName("libc.so","sendto");
    var recvfromPtr=Module.getExportByName("libc.so","recvfrom");
    console.log("sendto:",sendtoPtr,",recvfrom:",recvfromPtr);
    //sendto(int fd, const void *buf, size_t n, int flags, const struct sockaddr *addr, socklen_t addr_len)
    Interceptor.attach(sendtoPtr,{
        onEnter:function(args){
            var fd=args[0];
            var buff=args[1];
            var size=args[2];
            var sockdata=getSocketData(fd.toInt32());
            console.log(sockdata);
            console.log(hexdump(buff,{length:size.toInt32()}));
        },
        onLeave:function(retval){
 
        }
    });
    //recvfrom(int fd, void *buf, size_t n, int flags, struct sockaddr *addr, socklen_t *addr_len)
    Interceptor.attach(recvfromPtr,{
        onEnter:function(args){
            this.fd=args[0];
            this.buff=args[1];
            this.size=args[2];
        },
        onLeave:function(retval){
            var sockdata=getSocketData(this.fd.toInt32());
            console.log(sockdata);
            console.log(hexdump(this.buff,{length:this.size.toInt32()}));
        }
    });
}

```

最后的 hook 结果。这里注意。我们刚刚梳理的 jni 调用流程是 http 的。所以测试程序的请求连接要修改下。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_76TK8N3R42DT74X.png)

### [](#3、java层ssl抓包)3、java 层 ssl 抓包

https 实际上就是 http+ssl。由于 http 发送的数据直接就是明文。安全性非常差。https 会在数据发送前，先用 ssl 进行加密。如下图

 

![](https://bbs.pediy.com/upload/attach/202106/659397_ESP48FKA5234X36.png)

 

而加密则是使用对应的证书。接受到服务端数据后。再使用证书来进行解密

 

![](https://bbs.pediy.com/upload/attach/202106/659397_89PTBTS6F4VUKQC.png)

 

下面是 https 交互的流程。ssl 会先将证书中的公钥发送给服务器。然后服务器将自己的公钥返回给客户端。然后客户端拿到公钥后和证书中的私钥计算出共享密钥。然后就使用共享密钥来加解密服务端传递来的数据。服务端同样也是拿到客户端的公钥，就和自己的私钥计算出共享密钥。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_4WYUW9J3EHC7MRQ.png)

 

看完 https 的理论部分。我们就清楚如何达到自己的目的了。实际上只要在数据加密前的函数调用流程任意一个环节 hook 都能抓到明文。

 

下面直接修改访问地址成 https。由于有些证书是手机内置的无需我们自行添加证书。所以代码不需要修改。

```
GetByOkHttp("https://www.baidu.com/");
GetByHttpURL("https://www.jd.com/");

```

请求地址修改后。重新使用上面的脚本来 hook。发现无法再抓到 tcp 的请求数据了。说明调用链发生了变化。所以我们分析下访问 https 时的调用链

 

由于这个调用链比较长。我就只针对 OkHttp3 进行跟踪分析。这里我采用的是调试的方式来追踪这个函数。由于整个追踪跳转的较多。我就只针对重点部分进行记录了。在开始调试之前。我们先整理清楚自己的目的。

 

1、调试 GetByOkHttp 执行的流程。在执行过程中找到 ssl 相关的处理。

 

2、追踪 ssl 相关的处理。找到最后调用的 native 函数。

 

那么下面贴上调试时得到的信息

```
//最关键的是这个newCall。断点跟踪进去
public static void GetByOkHttp(String url) {
    ...
    Call call = okHttpClient.newCall(request);
    ...
}
//下面调用了一个静态的初始化RealCall
@Override public Call newCall(Request request) {
      return RealCall.newRealCall(this, request, false /* for web socket */);
}
//这里初始化RealCall
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
}
//到这里初始化好了。这里注意，retryAndFollowUpInterceptor这个实际上是一个拦截器，那么接下来我们应该把断点放在拦截器的intercept函数
private RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    this.client = client;
    this.originalRequest = originalRequest;
    this.forWebSocket = forWebSocket;
    this.retryAndFollowUpInterceptor = new RetryAndFollowUpInterceptor(client, forWebSocket);
}
 
//拦截器的触发函数。proceed里面的流程较长。里面经过了多次的跳转。
@Override public Response intercept(Chain chain) throws IOException {
        ...
      response = realChain.proceed(request, streamAllocation, null, null);
      ...
  }
//多次跳转后。最后走进了RealConnection.java的connect函数。
public void connect(int connectTimeout, int readTimeout, int writeTimeout,
      int pingIntervalMillis, boolean connectionRetryEnabled, Call call,
      EventListener eventListener) {
           ...
      //这里判断如果非ssl的处理
      if (route.address().sslSocketFactory() == null) {
        if (!connectionSpecs.contains(ConnectionSpec.CLEARTEXT)) {
          throw new RouteException(new UnknownServiceException(
              "CLEARTEXT communication not enabled for client"));
        }
        String host = route.address().url().host();
        if (!Platform.get().isCleartextTrafficPermitted(host)) {
          throw new RouteException(new UnknownServiceException(
              "CLEARTEXT communication to " + host + " not permitted by network security policy"));
        }
      }
          ...
      //这里面调用的socket.connect()
      connectSocket(connectTimeout, readTimeout, call, eventListener);
          ...
      //这里面发送的请求
      establishProtocol(connectionSpecSelector, pingIntervalMillis, call, eventListener);
          ...
  }
 
//先看看socket的连接部分
private void connectSocket(int connectTimeout, int readTimeout, Call call,
      EventListener eventListener) throws IOException {
      ...
    //根据不同的type创建不同的socket
    rawSocket = proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.HTTP
    ? address.socketFactory().createSocket()
    : new Socket(proxy);
      ...
    //socket的连接
    Platform.get().connectSocket(rawSocket, route.socketAddress(), connectTimeout);
      ...
}
//然后这里只是单纯的调用下连接。
public void connectSocket(Socket socket, InetSocketAddress address,
      int connectTimeout) throws IOException {
    socket.connect(address, connectTimeout);
  }
 
//下面继续看发送请求的函数establishProtocol
private void establishProtocol(ConnectionSpecSelector connectionSpecSelector,
      int pingIntervalMillis, Call call, EventListener eventListener) throws IOException {
      //非ssl的请求直接在这里就返回了
    if (route.address().sslSocketFactory() == null) {
      protocol = Protocol.HTTP_1_1;
      socket = rawSocket;
      return;
    }
      //后面是ssl相关的处理
    eventListener.secureConnectStart(call);
      //最关键的是这里会创建sslsocket。并且和服务端进行握手交互
    connectTls(connectionSpecSelector);
    eventListener.secureConnectEnd(call, handshake);
 
    if (protocol == Protocol.HTTP_2) {
      socket.setSoTimeout(0); // HTTP/2 connection timeouts are set per-stream.
      http2Connection = new Http2Connection.Builder(true)
          .socket(socket, route.address().url().host(), source, sink)
          .listener(this)
          .pingIntervalMillis(pingIntervalMillis)
          .build();
      http2Connection.start();
    }
  }
 
//最关键的部分。所以我就贴上完整的代码了。里面很多地方有英文注释。我就不自己画蛇添足了。
private void connectTls(ConnectionSpecSelector connectionSpecSelector) throws IOException {
    Address address = route.address();
    SSLSocketFactory sslSocketFactory = address.sslSocketFactory();
    boolean success = false;
    SSLSocket sslSocket = null;
    try {
      // Create the wrapper over the connected socket.
      sslSocket = (SSLSocket) sslSocketFactory.createSocket(
          rawSocket, address.url().host(), address.url().port(), true /* autoClose */);
 
      // Configure the socket's ciphers, TLS versions, and extensions.
      ConnectionSpec connectionSpec = connectionSpecSelector.configureSecureSocket(sslSocket);
      if (connectionSpec.supportsTlsExtensions()) {
        Platform.get().configureTlsExtensions(
            sslSocket, address.url().host(), address.protocols());
      }
 
      // Force handshake. This can throw!
      sslSocket.startHandshake();
      // block for session establishment
      SSLSession sslSocketSession = sslSocket.getSession();
      if (!isValid(sslSocketSession)) {
        throw new IOException("a valid ssl session was not established");
      }
      Handshake unverifiedHandshake = Handshake.get(sslSocketSession);
 
      // Verify that the socket's certificates are acceptable for the target host.
      if (!address.hostnameVerifier().verify(address.url().host(), sslSocketSession)) {
        X509Certificate cert = (X509Certificate) unverifiedHandshake.peerCertificates().get(0);
        throw new SSLPeerUnverifiedException("Hostname " + address.url().host() + " not verified:"
            + "\n    certificate: " + CertificatePinner.pin(cert)
            + "\n    DN: " + cert.getSubjectDN().getName()
            + "\n    subjectAltNames: " + OkHostnameVerifier.allSubjectAltNames(cert));
      }
            //这里是证书的校验
      // Check that the certificate pinner is satisfied by the certificates presented.
      address.certificatePinner().check(address.url().host(),
          unverifiedHandshake.peerCertificates());
 
      // Success! Save the handshake and the ALPN protocol.
      String maybeProtocol = connectionSpec.supportsTlsExtensions()
          ? Platform.get().getSelectedProtocol(sslSocket)
          : null;
      socket = sslSocket;
      source = Okio.buffer(Okio.source(socket));
      sink = Okio.buffer(Okio.sink(socket));
      handshake = unverifiedHandshake;
      protocol = maybeProtocol != null
          ? Protocol.get(maybeProtocol)
          : Protocol.HTTP_1_1;
      success = true;
    } catch (AssertionError e) {
      if (Util.isAndroidGetsocknameError(e)) throw new IOException(e);
      throw e;
    } finally {
      if (sslSocket != null) {
        Platform.get().afterHandshake(sslSocket);
      }
      if (!success) {
        closeQuietly(sslSocket);
      }
    }
  }
 
//上面流程走完。ssl的连接就建立起来了。继续跟踪后。走到下一个关键拦截器CallServerInterceptor.java。我们直接看拦截器函数实现部分
@Override public Response intercept(Chain chain) throws IOException {
    ...
        //前面的代码应该是准备header的数据。我就都省略了。下面这个函数是发送请求的关键函数
    httpCodec.finishRequest();
        ...
}
//继续往后跟踪flush
@Override public void finishRequest() throws IOException {
    sink.flush();
}
//继续跟踪sink.write
@Override public void flush() throws IOException {
    if (closed) throw new IllegalStateException("closed");
    if (buffer.size > 0) {
      sink.write(buffer, buffer.size);
    }
    sink.flush();
}
//继续往后看write
@Override public void write(Buffer source, long byteCount) throws IOException {
        checkOffsetAndCount(source.size, 0, byteCount);
      ...
    sink.write(source, toWrite);
    ...
}
//这个write就到一个关键的位置了。这个out对象就是我们关心的类了。我们查一下out的类型
@Override public void write(Buffer source, long byteCount) throws IOException {
    ...
    //out对象的类型是com.android.org.conscrypt.ConscryptFileDescriptorSocket$SSLOutputStream
    out.write(head.data, head.pos, toCopy);
        ...
}
//接下来我们就可以去找SSLOutputStream的write了。我们在调试中跟踪过去失败了。那么直接去翻源码把。下面是对应的write
public void write(byte[] buf, int offset, int byteCount) throws IOException {
        ...
    ssl.write(Platform.getFileDescriptor(socket), buf, offset, byteCount,writeTimeoutMilliseconds);
    ...
}
//继续看ssl.write。这个ssl的类型是SslWrapper。找到对应的write如下。
void write(FileDescriptor fd, byte[] buf, int offset, int len, int timeoutMillis)
            throws IOException {
        NativeCrypto.SSL_write(ssl, fd, handshakeCallbacks, buf, offset, len, timeoutMillis);
}
//继续找NativeCrypto类的SSL_write函数。到这里就结束了。终于到达java调用链的最后一层了。
static native void SSL_write(long sslNativePointer, FileDescriptor fd,
            SSLHandshakeCallbacks shc, byte[] b, int off, int len, int writeTimeoutMillis)
            throws IOException;

```

根据上面的一连串翻找。终于走完了 https 请求时的 java 部分的整个调用链。至于读取包的。可以直接猜测一下，看有没有对应的 SSL_read。搜了一下。果然是有的。很可能就是这个

```
static native int SSL_read(long sslNativePointer, FileDescriptor fd, SSLHandshakeCallbacks shc,
            byte[] b, int off, int len, int readTimeoutMillis) throws IOException;

```

接下来开始写 hook 脚本来处理 SSL_write 和 SSL_read。在这之前。我们要先找到 NativeCrypto 类的包名才行。用 frida 来搜索一下

```
function getFullName(name){
    Java.perform(function(){
        Java.enumerateLoadedClassesSync().forEach(function(className){
            if(className.indexOf(name)!=-1){
                console.log(className);
            }
        })
    });
}
getFullName("NativeCrypto")

```

最后得到结果`com.android.org.conscrypt.NativeCrypto`

 

然后来处理读取和写入

```
//NativeCrypto   SSL_write(long sslNativePointer, FileDescriptor fd,
//             SSLHandshakeCallbacks shc, byte[] b, int off, int len, int writeTimeoutMillis)
 
//NativeCrypto   SSL_read(long sslNativePointer, FileDescriptor fd, SSLHandshakeCallbacks shc,
//             byte[] b, int off, int len, int readTimeoutMillis)
function hook_ssl(){
    var NativeCryptoClass= Java.use("com.android.org.conscrypt.NativeCrypto");
    NativeCryptoClass.SSL_write.implementation=function(sslPtr,fd,shc,b,off,len,timeout){
        console.log("enter SSL_write");
        print_bytes(b);
        return this.SSL_write(sslPtr,fd,shc,b,off,len,timeout);
    };
    NativeCryptoClass.SSL_read.implementation=function(sslPtr,fd,shc,b,off,len,timeout){
        console.log("enter SSL_read");
        var res=this.SSL_read(sslPtr,fd,shc,b,off,len,timeout);
        print_bytes(b);
        return res;
    };
}

```

然后这里就能获取到 https 的包数据了。但是如果是 tcp 的情况会有个问题。就是没有连接的服务器 ip 地址和端口。在整个调用链环节中。我们可以找个尽量靠近 native 层调用，并且能取到服务器地址和端口的函数来 hook。比如直接 hook 了 SSLOutputStream 的 write 和 SSLInputStream 的 read。我们先看看断点调试走到最后的 SLLOutPutStream 的那里。然后展开这个 out 对象看看有些什么属性

 

![](https://bbs.pediy.com/upload/attach/202106/659397_UYVZ6EX2USPVS3S.png)

 

看到在 out 里面有个 this$0 里面的 socket 有我们想要的目标服务器地址和端口。这个 $0 实际上是指向当前这个类中类的父级对象。那么我们再 hook 一下这里

 

老样子先用之前的办法取到完整类名`com.android.org.conscrypt.ConscryptFileDescriptorSocket$SSLOutputStream`

```
function getSocketData2(stream){
    var data0=stream.this$0.value;
    var sockdata=data0.socket.value;
    return sockdata;
}
//SSLOutputStream  write(byte[] buf, int offset, int byteCount)
//SSLInputStream     read(byte[] buf, int offset, int byteCount)
function hook_ssl2(){
    var SSLOutputClass=Java.use("com.android.org.conscrypt.ConscryptFileDescriptorSocket$SSLOutputStream");
    SSLOutputClass.write.overload('[B', 'int', 'int').implementation=function(buf,off,cnt){
        console.log(getSocketData2(this));
        print_bytes(buf);
        return this.write(buf,off,cnt);
    };
    var SSLInputClass=Java.use("com.android.org.conscrypt.ConscryptFileDescriptorSocket$SSLInputStream");
    SSLInputClass.read.overload('[B', 'int', 'int').implementation=function(buf,off,cnt){
        var res=this.read(buf,off,cnt);
        console.log(getSocketData2(this));
        print_bytes(buf);
        return res;
    }
}

```

最后看看效果。能成功抓到请求了。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_TQ2SCQ9MG7P744D.png)

### [](#4、jni层抓https包)4、jni 层抓 https 包

继续跟着前面的线索往 jni 层跟踪。前面我们看到的最后一层是 SSL_write 和 SSL_read，所属类是 NativeCrypto。直接搜索函数 NativeCrypto_SSL_write。下面贴出关键代码

```
static void NativeCrypto_SSL_write(JNIEnv* env, jclass, jlong ssl_address, jobject fdObject,
                                   jobject shc, jbyteArray b, jint offset, jint len,
                                   jint write_timeout_millis) {
      ...
    ret = sslWrite(env, ssl, fdObject, shc, reinterpret_cast(&buf[0]), len,
                           sslError, write_timeout_millis);
      ...
 
} 
```

继续查看 sslWrite 的实现。发现里面又调用了一层 SSL_write。这个函数虽然和上面的函数同名。但是不是同一个了。该函数是 boringssl 中的了。这是谷歌创建的 openssl 的分支，内置在 android 中的。

```
static int sslWrite(JNIEnv* env, SSL* ssl, jobject fdObject, jobject shc, const char* buf, jint len,
                    OpenSslError& sslError, int write_timeout_millis) {
      ...
    int result = SSL_write(ssl, buf, len);
    ...
}

```

继续看 boringssl 中的 SSL_write。这里调用的函数是根据不同 ssl 版本调用的。所以找对应的函数应该带上对应版本比如 ssl3_write_app_data

```
int SSL_write(SSL *ssl, const void *buf, int num) {
      ...
    ret = ssl->method->write_app_data(ssl, &needs_handshake,
                                      (const uint8_t *)buf, num);
      ...
}

```

继续看 ssl3_write_app_data 的实现

```
int ssl3_write_app_data(SSL *ssl, int *out_needs_handshake, const uint8_t *buf,
                        int len) {
      ...
    int ret = do_ssl3_write(ssl, SSL3_RT_APPLICATION_DATA, &buf[tot], nw);
        ...
}

```

继续看 do_ssl3_write 的实现。这里可以看到这里就是明文的终点了。再往后面去的函数数据都是密文的了。从这里往前的流程都是明文。可以找任意一个觉得合适的地方来 hook。

```
static int do_ssl3_write(SSL *ssl, int type, const uint8_t *buf, unsigned len) {
    ...
/* Add any unflushed handshake data as a prefix. This may be a KeyUpdate
* acknowledgment or 0-RTT key change messages. |pending_flight| must be clear
* when data is added to |write_buffer| or it will be written in the wrong
* order. */
  if (ssl->s3->pending_flight != NULL) {
    OPENSSL_memcpy(
        out, ssl->s3->pending_flight->data + ssl->s3->pending_flight_offset,
        flight_len);
    BUF_MEM_free(ssl->s3->pending_flight);
    ssl->s3->pending_flight = NULL;
    ssl->s3->pending_flight_offset = 0;
  }
  if (!tls_seal_record(ssl, out + flight_len, &ciphertext_len,
                       max_out - flight_len, type, buf, len)) {
    return -1;
  }
  ssl_write_buffer_set_len(ssl, flight_len + ciphertext_len);
    ...
  /* we now just need to write the buffer */
  return ssl3_write_pending(ssl, type, buf, len);
}

```

在开始写 hook 处理前解决一个疑问。就是之前在 recvfrom 和 sendto 进行 hook。并没有取到 https 的密文。所以我们去看一下。boringssl 中是怎么处理最终发送数据的。在 boringssl 项目的文件`crypto/bio/socket.c`中

```
static int sock_read(BIO *b, char *out, int outl) {
  int ret = 0;
  if (out == NULL) {
    return 0;
  }
  bio_clear_socket_error();
#if defined(OPENSSL_WINDOWS)
  ret = recv(b->num, out, outl, 0);
#else
  ret = read(b->num, out, outl);
#endif
  BIO_clear_retry_flags(b);
  if (ret <= 0) {
    if (bio_fd_should_retry(ret)) {
      BIO_set_retry_read(b);
    }
  }
  return ret;
}
 
static int sock_write(BIO *b, const char *in, int inl) {
  int ret;
 
  bio_clear_socket_error();
#if defined(OPENSSL_WINDOWS)
  ret = send(b->num, in, inl, 0);
#else
  ret = write(b->num, in, inl);
#endif
  BIO_clear_retry_flags(b);
  if (ret <= 0) {
    if (bio_fd_should_retry(ret)) {
      BIO_set_retry_write(b);
    }
  }
  return ret;
}

```

这里可以看到如何是 jni 层的 ssl 里面最终发送数据的地方。判断了如果是 win 平台就用 send 和 recv。否则就使用 write 和 read。这样我们就知道 jni 层的 ssl 怎么抓明文和密文了。明文的 hook 点直接选择 openssl 调用的函数 SSL_write 和 SSL_read 接下来开始写代码。下面贴上完整的代码

```
//将数组转换成c++的byte[]。并且hexdump打印结果
function print_bytes(bytes) {
    var buf  = Memory.alloc(bytes.length);
    Memory.writeByteArray(buf, byte_to_ArrayBuffer(bytes));
    console.log(hexdump(buf, {offset: 0, length: bytes.length, header: false, ansi: true}));
}
//将java的数组转换成js的数组
function byte_to_ArrayBuffer(bytes) {
    var size=bytes.length;
    var tmparray = [];
    for (var i = 0; i < size; i++) {
        var val = bytes[i];
        if(val < 0){
            val += 256;
        }
        tmparray[i] = val
    }
    return tmparray;
}
// java.net.Socket$init(ip,port) 获取ip和端口
// socketWrite0(FileDescriptor fd, byte[] b, int off,int len) 获取发送的数据
// socketRead0(FileDescriptor fd,byte b[], int off, int len,int timeout) 获取接受的数据
function hook_tcp(){
    var socketClass= Java.use("java.net.Socket");
    socketClass.$init.overload('java.net.InetAddress', 'int').implementation=function(ip,port){
        console.log("socket$init ",ip,":",port);
        return this.$init(ip,port);
    };
    var outputClass=Java.use("java.net.SocketOutputStream");
    outputClass.socketWrite0.implementation=function(fd,buff,off,len){
        console.log("tcp write fd:",fd);
        print_bytes(buff);
        return this.socketWrite0(fd,buff,off,len);
    };
 
    var inputClass=Java.use("java.net.SocketInputStream");
    inputClass.socketRead0.implementation=function(fd,buff,off,len,timeout){
        var res=this.socketRead0(fd,buff,off,len,timeout);
        console.log("tcp read fd:",fd)
        print_bytes(buff);
        return res;
    };
}
//SSL_write(long sslNativePointer, FileDescriptor fd,
//             SSLHandshakeCallbacks shc, byte[] b, int off, int len, int writeTimeoutMillis)
//SSL_read(long sslNativePointer, FileDescriptor fd, SSLHandshakeCallbacks shc,
//             byte[] b, int off, int len, int readTimeoutMillis)
function getSocketData(fd){
    console.log("fd:",fd);
    var socketType=Socket.type(fd)
    if(socketType!=null){
        var res="type:"+socketType+",loadAddress:"+JSON.stringify(Socket.localAddress(fd))+",peerAddress"+JSON.stringify(Socket.peerAddress(fd));
        return res;
    }else{
        return "type:"+socketType;
    }
}
function getSocketData2(stream){
    var data0=stream.this$0.value;
    var sockdata=data0.socket.value;
    return sockdata;
}
function hook_ssl(){
    var NativeCryptoClass= Java.use("com.android.org.conscrypt.NativeCrypto");
    NativeCryptoClass.SSL_write.implementation=function(sslPtr,fd,shc,b,off,len,timeout){
        console.log("enter SSL_write");
        print_bytes(b);
        return this.SSL_write(sslPtr,fd,shc,b,off,len,timeout);
    };
    NativeCryptoClass.SSL_read.implementation=function(sslPtr,fd,shc,b,off,len,timeout){
        console.log("enter SSL_read");
        var res=this.SSL_read(sslPtr,fd,shc,b,off,len,timeout);
        print_bytes(b);
        return res;
    };
}
//SSLOutputStream  write(byte[] buf, int offset, int byteCount)
//SSLInputStream     read(byte[] buf, int offset, int byteCount)
function hook_ssl2(){
    var SSLOutputClass=Java.use("com.android.org.conscrypt.ConscryptFileDescriptorSocket$SSLOutputStream");
    SSLOutputClass.write.overload('[B', 'int', 'int').implementation=function(buf,off,cnt){
        console.log(getSocketData2(this));
        print_bytes(buf);
        return this.write(buf,off,cnt);
    };
    var SSLInputClass=Java.use("com.android.org.conscrypt.ConscryptFileDescriptorSocket$SSLInputStream");
    SSLInputClass.read.overload('[B', 'int', 'int').implementation=function(buf,off,cnt){
        var res=this.read(buf,off,cnt);
        console.log(getSocketData2(this));
        print_bytes(buf);
        return res;
    }
}
//jni的ssl的加密数据hook
function hook_jni_ssl_enc(){
    var writePtr=Module.getExportByName("libc.so","write");
    var readPtr=Module.getExportByName("libc.so","read");
    console.log("write:",writePtr,",read:",readPtr);
    Interceptor.attach(writePtr,{
        onEnter:function(args){
            var fd=args[0];
            var buff=args[1];
            var size=args[2];
            var sockdata=getSocketData(fd.toInt32());
            if(sockdata.indexOf("tcp")){
                console.log(sockdata);
                console.log(hexdump(buff,{length:size.toInt32()}));
            }
        },
        onLeave:function(retval){
 
        }
    });
    Interceptor.attach(readPtr,{
        onEnter:function(args){
            this.fd=args[0];
            this.buff=args[1];
            this.size=args[2];
        },
        onLeave:function(retval){
            var sockdata=getSocketData(this.fd.toInt32());
            if(sockdata.indexOf("tcp")){
                console.log(sockdata);
                console.log(hexdump(this.buff,{length:this.size.toInt32()}));
            }
        }
    });
}
//jni的ssl明文数据hook
function hook_jni_ssl(){
    var sslWritePtr=Module.getExportByName("libssl.so","SSL_write");
    var sslReadPtr=Module.getExportByName("libssl.so","SSL_read");
    console.log("sslWrite:",sslWritePtr,",sslRead:",sslReadPtr);
    var sslGetFdPtr=Module.getExportByName("libssl.so","SSL_get_rfd");
    var sslGetFdFunc=new NativeFunction(sslGetFdPtr,'int',['pointer']);
 
    //int SSL_write(SSL *ssl, const void *buf, int num)
    Interceptor.attach(sslWritePtr,{
        onEnter:function(args){
            var sslPtr=args[0];
            var buff=args[1];
            var size=args[2];
            var fd=sslGetFdFunc(sslPtr);
            var sockdata=getSocketData(fd);
            console.log(sockdata);
            console.log(hexdump(buff,{length:size.toInt32()}));
 
        },
        onLeave:function(retval){
 
        }
    });
    //int SSL_read(SSL *ssl, void *buf, int num)
    Interceptor.attach(sslReadPtr,{
        onEnter:function(args){
            this.sslPtr=args[0];
            this.buff=args[1];
            this.size=args[2];
        },
        onLeave:function(retval){
            var fd=sslGetFdFunc(this.sslPtr);
            var sockdata=getSocketData(fd);
            console.log(sockdata);
            console.log(hexdump(this.buff,{length:this.size.toInt32()}));
        }
    });
}
 
function hook_jni_tcp(){
    var sendtoPtr=Module.getExportByName("libc.so","sendto");
    var recvfromPtr=Module.getExportByName("libc.so","recvfrom");
    console.log("sendto:",sendtoPtr,",recvfrom:",recvfromPtr);
    //sendto(int fd, const void *buf, size_t n, int flags, const struct sockaddr *addr, socklen_t addr_len)
    Interceptor.attach(sendtoPtr,{
        onEnter:function(args){
            var fd=args[0];
            var buff=args[1];
            var size=args[2];
            var sockdata=getSocketData(fd.toInt32());
            console.log(sockdata);
            console.log(hexdump(buff,{length:size.toInt32()}));
        },
        onLeave:function(retval){
 
        }
    });
    //recvfrom(int fd, void *buf, size_t n, int flags, struct sockaddr *addr, socklen_t *addr_len)
    Interceptor.attach(recvfromPtr,{
        onEnter:function(args){
            this.fd=args[0];
            this.buff=args[1];
            this.size=args[2];
        },
        onLeave:function(retval){
            var sockdata=getSocketData(this.fd.toInt32());
            console.log(sockdata);
            console.log(hexdump(this.buff,{length:this.size.toInt32()}));
        }
    });
}
 
function hook_java(){
    Java.perform(function(){
        // hook_tcp();
        // hook_ssl();
        // hook_ssl2();
        // hook_jni_tcp();
        hook_jni_ssl_enc();
        hook_jni_ssl();
    })
}
function getFullName(name){
    Java.perform(function(){
        Java.enumerateLoadedClassesSync().forEach(function(className){
            if(className.indexOf(name)!=-1){
                console.log(className);
            }
        })
    });
}
 
function main(){
    hook_java();
    // getFullName("SSLOutputStream")
}
 
setImmediate(main);

```

跑起来后的结果如下。成功抓到 ssl 的密文和 ssl 的明文。想要回溯的话直接加上打印堆栈即可。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_23QR2YTFUTZAPNR.png)

 

![](https://bbs.pediy.com/upload/attach/202106/659397_9JTDDURCQC5N46X.png)

中间人抓包
-----

我们常常用的 burp 和 charles 就是属于中间人抓包。前面我们看过了 https 的加密方式，需要使用证书的公钥去交换加密再发送数据。那么这些工具是如何做到抓取 https 的数据包并解密的呢。方式就和他的名字一样。这个抓包的应用他也有一个证书。然后客户端向服务端发送数据时，中间人就假装自己是服务端。将自己证书的公钥发送给客户端，然后拿到客户发送的数据后，再假装自己是客户端，把自己的证书公钥发送给服务端。大概可以想象成一个双面间谍。客户端面前冒充服务端，服务端面前冒充客户端。下面贴上网上找的交互的示意图，包括握手流程都写的非常详细了。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_UQDVREJRH85C3CM.png)

 

而使用中间人抓包。常常会碰到防抓包手段。这些手段一般都是针对证书的检测。

### [](#1、服务端验证客户端证书)1、服务端验证客户端证书

服务端会检测客户端交互使用的证书是不是正确的。这种情况我们可以直接将客户端里面使用的证书导出来，然后设置让 charles 使用指定的证书来抓包。这里导出证书的办法有两种。

 

第一种是直接 hook 代码中设置证书的地方，直接重新设置一次空的证书。让其不要验证。

 

这里看一个网上找的 android 设置证书的例子

```
val resourceStream = resources.openRawResource(R.raw.infinisign_cert)
val keyStoreType = KeyStore.getDefaultType()
val keyStore = KeyStore.getInstance(keyStoreType)
keyStore.load(resourceStream, null)
 
val trustManagerAlgorithm = TrustManagerFactory.getDefaultAlgorithm()
val trustManagerFactory = TrustManagerFactory.getInstance(trustManagerAlgorithm)
trustManagerFactory.init(keyStore)
 
val sslContext = SSLContext.getInstance("TLS")
sslContext.init(null, trustManagerFactory.trustManagers, null)
val url = URL("http://infinisign.com/")
val urlConnection = url.openConnection() as HttpsURLConnection
urlConnection.sslSocketFactory = sslContext.socketFactory

```

大概意思是从资源文件加载到证书。然后用 keyStore 加载证书。后面再使用。所以我们直接看看 keyStore 的 load 函数

```
public final void load(InputStream stream, char[] password)
        throws IOException, NoSuchAlgorithmException, CertificateException
    {
        keyStoreSpi.engineLoad(stream, password);
        initialized = true;
    }

```

还有其他方式加载证书。看下面的我网上翻的另外一个例子

```
CertificateFactory factory = CertificateFactory.getInstance("X.509");//设置证书类型，X.509是一种格式标准
InputStream stream;
Certificate certificate;//Certificate是证书信息封装的一个bean类
 
KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
keyStore.load(null);//清除默认证书,使用我们自己制定的证书
 
CertificateFactory cf = CertificateFactory.getInstance("X.509");
InputStream caInput = getAssets().open("burp.der");
Certificate cert = cf.generateCertificate(caInput);
//设置自己的证书
keyStore.setCertificateEntry("misskings",cert);
 
TrustManagerFactory trustManagerFactory =       TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
trustManagerFactory.init(keyStore);//通过keyStore得到信任管理器
 
KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
keyManagerFactory.init(keyStore, "pwd".toCharArray());//通过keyStore得到密匙管理器
 
SSLContext sslContext = SSLContext.getInstance("TLS");
sslContext.init(keyManagerFactory.getKeyManagers(), trustManagerFactory.getTrustManagers(), new SecureRandom());
SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();//拿到SSLSocketFactory
 
TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
  return ;
}
X509TrustManager trustManager = (X509TrustManager) trustManagers[0];
okHttpClient.sslSocketFactory(sslSocketFactory, trustManager)//设置ssl证书
okHttpClient.build();

```

发现这个例子里面的 keyStore.load 是 null。是在后面进行再设置进去的。所以如果我们处理 load 函数。就会没有啥效果了。我们可以找找其他 hook 点。比如我直接在两个 init 的地方进行 hook

```
function hook_keystore(){
    var keyManager=Java.use("javax.net.ssl.KeyManagerFactory");
    var trustManager=Java.use("javax.net.ssl.TrustManagerFactory");
    keyManager.init.overload('java.security.KeyStore', '[C').implementation=function(ks,pwd){
        console.log("enter keyManager init");
        try{
            var savePath="/sdcard/keyManagerFactory_init.p12";
            var outStream=Java.use("java.io.FileOutputStream").$new(savePath);
            ks.store(outStream,Java.use("java.lang.String").$new(pwd).toCharArray());
            console.log("keyManager store success");
        }catch(ex){
            console.log(ex);
        }
        return this.init(ks,pwd);
    }
    trustManager.init.overload('java.security.KeyStore').implementation=function(ks){
        console.log("enter trustManager init");
        try{
            var savePath="/sdcard/TrustManagerFactory_init.p12";
            var outStream=Java.use("java.io.FileOutputStream").$new(savePath);
            ks.store(outStream);
            console.log("trustManager store success");
        }catch(ex){
            console.log(ex);
        }
        return this.init(ks);
    }
}

```

然后我们测试下自己的例子。`frida -U --no-pause -f com.example.zhuabao -l zhuabao.js`

```
trustManager store success
keyManager store success

```

最后导出来的文件。我对比了一下原文件。发现要去掉头部的 0x43 个字节（这里是因为我安卓代码里面设置了别名）和尾部的 0x15 个字节。就和原证书文件完全一致了。不过是否通用就不知道了。未测试多个样本。

 

第二种是直接解压 apk。然后在里面搜索证书特征的文件。例如下面。大概搜一下一些证书的后缀。一般也可能直接找到。

 

![](https://bbs.pediy.com/upload/attach/202106/659397_K44EVZZCV8VRW9J.png)

### [](#2、客户端验证服务端证书)2、客户端验证服务端证书

大概就是客户端向服务端请求。然后服务端把自己的证书返回了。客户端验证一下证书是否正确有效。没啥问题才正常通讯。而这种判断逻辑直接在客户端的。那就直接 hook 修改让他不要处理就行了。

 

解决这个的开源项目有很多。

 

xposed 解决方案:[JustTrustMe](https://github.com/Fuzion24/JustTrustMe)

 

frida 解决方案:[DroidSSLUnpinning](https://github.com/WooyunDota/DroidSSLUnpinning)

[【公告】看雪团队招聘安全工程师，将兴趣和工作融合在一起！看雪 20 年安全圈的口碑，助你快速成长！](https://job.kanxue.com/position-read-1104.htm)

[#基础理论](forum-161-1-117.htm) [#协议分析](forum-161-1-120.htm)