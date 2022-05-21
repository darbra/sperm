> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIxOTQ1OTA0Ng==&mid=2247484195&idx=1&sn=a41d54d3f3e85a414d293a9218a78385&chksm=97dbbfaaa0ac36bc9a5168b3107414fb1dc7de3cf7ac95d53bb1469d68824a3708dc9e3547e0&mpshare=1&scene=1&srcid=052124o36LBznRjI1gtBItF7&sharer_sharetime=1653142414873&sharer_shareid=9be5daf09995ef938577edacf59663a3&version=4.0.2.90460&platform=mac#rd)

这次的猿人学 2022 逆向比赛，和 darbra 老师组队拿到了第一名，在此先说一句，darbra 老师牛逼！

比赛链接：https://appmatch.yuanrenxue.com/

第七题是 quic，先百度一下 quic 是什么东西

QUIC （Quick UDP Internet Connections）是由 Google 从 2013 年开始研究的基于 UDP 的可靠传输协议，它最早的原型是 SPDY + QUIC-Crypto + Reliable UDP，后来经历了 SPDY 转型为 2015 年 5 月 IETF 正式发布的 HTTP/2.0，以及 2016 年 TLS/1.3 的正式发布。2016 年成立，IETF 的 QUIC 标准化工作组启动，考虑到 HTTP/2.0 和 TLS/1.3 的发布，它的核心协议族逐步进化为现在的 HTTP/3.0 + TLS/1.3 + QUIC-Transport 的组合。

然后尝试抓包，发现这题只有 28443 端口的一个 udp 数据包，而且内容是加密的

至此，便有一个问题，如何抓包他发送的明文包，百度了一下，因为 quic 是 tls 加密过的，所以需要抓到秘钥才可以解密明文数据，遂放弃了直接抓包的思路。

那么我们能不能另辟蹊径呢，bingo，如果我们模拟一个服务端，然后把 app 的目标服务器指向我们的服务端，是不是就可以得到明文数据了呢？既然有了思路那么就尝试实现一下。

首先是如何修改 app 指向的服务端，先尝试改 app 的文件，发现第七题的 so 中有 app 服务端的地址，但是并不知道怎么直接修改它，于是使用 frida hook，hook memcpy 这个函数，在它进行复制的时候，把服务端地址改掉，这里需要注意，服务端地址修改后的长度要和原来的长度一样，不然可能会有问题，然后我就改了一下 ip 地址，把服务端 hook 为 192.168.1.129，下面是 hook 的代码

```
import codecs
import frida
import sys
import threading
 
 
#device = frida.get_remote_device()
device = frida.get_usb_device()
print(device)
 
 
pending = []
sessions = []
scripts = []
event = threading.Event()
 
jscode = """ 
 function get_jbytes(sb) {
    var env = Java.vm.getEnv();
    return Memory.readCString(env.getByteArrayElements(sb));
}
 
function stringToUint8Array(str){
  var arr = [];
  for (var i = 0, j = str.length; i < j; ++i) {
    arr.push(str.charCodeAt(i));
  }
 
  var tmpUint8Array = new Uint8Array(arr);
  return tmpUint8Array
}
 
console.log('start')  
function inline_hook() {
    while(1)
    {
      var so_addr = Module.findBaseAddress("libmatch07.so");
      console.log("so_addr:", so_addr);
      
      if (so_addr) { 
     
          var addr_memcpy = Module.findExportByName("libmatch07.so", "memcpy")
           console.log("addr_memcpy:", addr_memcpy);
      
            Interceptor.attach(new NativePointer(addr_memcpy), { 
            onEnter: function (args)
            { 
               var nowdata=Memory.readCString(args[1]);
               if(nowdata && nowdata.indexOf('180.76.60.244')>-1)
               {
                console.log('args1', nowdata);  
                nowdata = nowdata.replace('180.76.60.244','192.168.1.129') 
                Memory.writeByteArray(args[1], stringToUint8Array(nowdata)); 
               }
            },
            onLeave: function (retval)
            {  
                 
            } 
        });  
 
        break;
    } 
    Thread.sleep(0);
    }
}
 
setImmediate(inline_hook) 
   
"""
 
pid = device.spawn(["com.yuanrenxue.match2022"])
  
session = device.attach(pid)
print(" Attach Application id:",pid)
device.resume(pid) 
script = session.create_script(jscode) 
print(' Running')
script.load()
sys.stdin.read()



```

然后再写找一个 quic 服务端进行测试，这里使用了 aioquic 这个库，代码如下 http3_server.py：

```
import argparse
import asyncio
import importlib
import logging
import time
from collections import deque
from email.utils import formatdate
from typing import Callable, Deque, Dict, List, Optional, Union, cast
 
import wsproto
import wsproto.events
 
import aioquic
from aioquic.asyncio import QuicConnectionProtocol, serve
from aioquic.h0.connection import H0_ALPN, H0Connection
from aioquic.h3.connection import H3_ALPN, H3Connection
from aioquic.h3.events import (
    DatagramReceived,
    DataReceived,
    H3Event,
    HeadersReceived,
    WebTransportStreamDataReceived,
)
from aioquic.h3.exceptions import NoAvailablePushIDError
from aioquic.quic.configuration import QuicConfiguration
from aioquic.quic.events import DatagramFrameReceived, ProtocolNegotiated, QuicEvent
from aioquic.quic.logger import QuicFileLogger
from aioquic.tls import SessionTicket
 
try:
    import uvloop
except ImportError:
    uvloop = None
 
AsgiApplication = Callable
HttpConnection = Union[H0Connection, H3Connection]
 
SERVER_NAME = "aioquic/" + aioquic.__version__
 
 
class HttpRequestHandler:
    def __init__(
        self,
        *,
        authority: bytes,
        connection: HttpConnection,
        protocol: QuicConnectionProtocol,
        scope: Dict,
        stream_ended: bool,
        stream_id: int,
        transmit: Callable[[], None],
    ) -> None:
        self.authority = authority
        self.connection = connection
        self.protocol = protocol
        self.queue: asyncio.Queue[Dict] = asyncio.Queue()
        self.scope = scope
        self.stream_id = stream_id
        self.transmit = transmit
 
        if stream_ended:
            self.queue.put_nowait({"type": "http.request"})
 
    def http_event_received(self, event: H3Event) -> None:
        if isinstance(event, DataReceived):
            self.queue.put_nowait(
                {
                    "type": "http.request",
                    "body": event.data,
                    "more_body": not event.stream_ended,
                }
            )
        elif isinstance(event, HeadersReceived) and event.stream_ended:
            self.queue.put_nowait(
                {"type": "http.request", "body": b"", "more_body": False}
            )
 
    async def run_asgi(self, app: AsgiApplication) -> None:
        await app(self.scope, self.receive, self.send)
 
    async def receive(self) -> Dict:
        return await self.queue.get()
 
    async def send(self, message: Dict) -> None:
        if message["type"] == "http.response.start":
            self.connection.send_headers(
                stream_id=self.stream_id,
                headers=[
                    (b":status", str(message["status"]).encode()),
                    (b"server", SERVER_NAME.encode()),
                    (b"date", formatdate(time.time(), usegmt=True).encode()),
                ]
                + [(k, v) for k, v in message["headers"]],
            )
        elif message["type"] == "http.response.body":
            self.connection.send_data(
                stream_id=self.stream_id,
                data=message.get("body", b""),
                end_stream=not message.get("more_body", False),
            )
        elif message["type"] == "http.response.push" and isinstance(
            self.connection, H3Connection
        ):
            request_headers = [
                (b":method", b"GET"),
                (b":scheme", b"https"),
                (b":authority", self.authority),
                (b":path", message["path"].encode()),
            ] + [(k, v) for k, v in message["headers"]]
 
            # send push promise
            try:
                push_stream_id = self.connection.send_push_promise(
                    stream_id=self.stream_id, headers=request_headers
                )
            except NoAvailablePushIDError:
                return
 
            # fake request
            cast(HttpServerProtocol, self.protocol).http_event_received(
                HeadersReceived(
                    headers=request_headers, stream_ended=True, stream_id=push_stream_id
                )
            )
        self.transmit()
 
 
class WebSocketHandler:
    def __init__(
        self,
        *,
        connection: HttpConnection,
        scope: Dict,
        stream_id: int,
        transmit: Callable[[], None],
    ) -> None:
        self.closed = False
        self.connection = connection
        self.http_event_queue: Deque[DataReceived] = deque()
        self.queue: asyncio.Queue[Dict] = asyncio.Queue()
        self.scope = scope
        self.stream_id = stream_id
        self.transmit = transmit
        self.websocket: Optional[wsproto.Connection] = None
 
    def http_event_received(self, event: H3Event) -> None:
        if isinstance(event, DataReceived) and not self.closed:
            if self.websocket is not None:
                self.websocket.receive_data(event.data)
 
                for ws_event in self.websocket.events():
                    self.websocket_event_received(ws_event)
            else:
                # delay event processing until we get `websocket.accept`
                # from the ASGI application
                self.http_event_queue.append(event)
 
    def websocket_event_received(self, event: wsproto.events.Event) -> None:
        if isinstance(event, wsproto.events.TextMessage):
            self.queue.put_nowait({"type": "websocket.receive", "text": event.data})
        elif isinstance(event, wsproto.events.Message):
            self.queue.put_nowait({"type": "websocket.receive", "bytes": event.data})
        elif isinstance(event, wsproto.events.CloseConnection):
            self.queue.put_nowait({"type": "websocket.disconnect", "code": event.code})
 
    async def run_asgi(self, app: AsgiApplication) -> None:
        self.queue.put_nowait({"type": "websocket.connect"})
 
        try:
            await app(self.scope, self.receive, self.send)
        finally:
            if not self.closed:
                await self.send({"type": "websocket.close", "code": 1000})
 
    async def receive(self) -> Dict:
        return await self.queue.get()
 
    async def send(self, message: Dict) -> None:
        data = b""
        end_stream = False
        if message["type"] == "websocket.accept":
            subprotocol = message.get("subprotocol")
 
            self.websocket = wsproto.Connection(wsproto.ConnectionType.SERVER)
 
            headers = [
                (b":status", b"200"),
                (b"server", SERVER_NAME.encode()),
                (b"date", formatdate(time.time(), usegmt=True).encode()),
            ]
            if subprotocol is not None:
                headers.append((b"sec-websocket-protocol", subprotocol.encode()))
            self.connection.send_headers(stream_id=self.stream_id, headers=headers)
 
            # consume backlog
            while self.http_event_queue:
                self.http_event_received(self.http_event_queue.popleft())
 
        elif message["type"] == "websocket.close":
            if self.websocket is not None:
                data = self.websocket.send(
                    wsproto.events.CloseConnection(code=message["code"])
                )
            else:
                self.connection.send_headers(
                    stream_id=self.stream_id, headers=[(b":status", b"403")]
                )
            end_stream = True
        elif message["type"] == "websocket.send":
            if message.get("text") is not None:
                data = self.websocket.send(
                    wsproto.events.TextMessage(data=message["text"])
                )
            elif message.get("bytes") is not None:
                data = self.websocket.send(
                    wsproto.events.Message(data=message["bytes"])
                )
 
        if data:
            self.connection.send_data(
                stream_id=self.stream_id, data=data, end_stream=end_stream
            )
        if end_stream:
            self.closed = True
        self.transmit()
 
 
class WebTransportHandler:
    def __init__(
        self,
        *,
        connection: HttpConnection,
        scope: Dict,
        stream_id: int,
        transmit: Callable[[], None],
    ) -> None:
        self.accepted = False
        self.closed = False
        self.connection = connection
        self.http_event_queue: Deque[DataReceived] = deque()
        self.queue: asyncio.Queue[Dict] = asyncio.Queue()
        self.scope = scope
        self.stream_id = stream_id
        self.transmit = transmit
 
    def http_event_received(self, event: H3Event) -> None:
        if not self.closed:
            if self.accepted:
                if isinstance(event, DatagramReceived):
                    self.queue.put_nowait(
                        {
                            "data": event.data,
                            "type": "webtransport.datagram.receive",
                        }
                    )
                elif isinstance(event, WebTransportStreamDataReceived):
                    self.queue.put_nowait(
                        {
                            "data": event.data,
                            "stream": event.stream_id,
                            "type": "webtransport.stream.receive",
                        }
                    )
            else:
                # delay event processing until we get `webtransport.accept`
                # from the ASGI application
                self.http_event_queue.append(event)
 
    async def run_asgi(self, app: AsgiApplication) -> None:
        self.queue.put_nowait({"type": "webtransport.connect"})
 
        try:
            await app(self.scope, self.receive, self.send)
        finally:
            if not self.closed:
                await self.send({"type": "webtransport.close"})
 
    async def receive(self) -> Dict:
        return await self.queue.get()
 
    async def send(self, message: Dict) -> None:
        data = b""
        end_stream = False
 
        if message["type"] == "webtransport.accept":
            self.accepted = True
 
            headers = [
                (b":status", b"200"),
                (b"server", SERVER_NAME.encode()),
                (b"date", formatdate(time.time(), usegmt=True).encode()),
                (b"sec-webtransport-http3-draft", b"draft02"),
            ]
            self.connection.send_headers(stream_id=self.stream_id, headers=headers)
 
            # consume backlog
            while self.http_event_queue:
                self.http_event_received(self.http_event_queue.popleft())
        elif message["type"] == "webtransport.close":
            if not self.accepted:
                self.connection.send_headers(
                    stream_id=self.stream_id, headers=[(b":status", b"403")]
                )
            end_stream = True
        elif message["type"] == "webtransport.datagram.send":
            self.connection.send_datagram(flow_id=self.stream_id, data=message["data"])
        elif message["type"] == "webtransport.stream.send":
            self.connection._quic.send_stream_data(
                stream_id=message["stream"], data=message["data"]
            )
 
        if data or end_stream:
            self.connection.send_data(
                stream_id=self.stream_id, data=data, end_stream=end_stream
            )
        if end_stream:
            self.closed = True
        self.transmit()
 
 
Handler = Union[HttpRequestHandler, WebSocketHandler, WebTransportHandler]
 
 
class HttpServerProtocol(QuicConnectionProtocol):
    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
        self._handlers: Dict[int, Handler] = {}
        self._http: Optional[HttpConnection] = None
 
    def http_event_received(self, event: H3Event) -> None:
        if isinstance(event, HeadersReceived) and event.stream_id not in self._handlers:
            authority = None
            headers = []
            http_version = "0.9" if isinstance(self._http, H0Connection) else "3"
            raw_path = b""
            method = ""
            protocol = None
            for header, value in event.headers:
                if header == b":authority":
                    authority = value
                    headers.append((b"host", value))
                elif header == b":method":
                    method = value.decode()
                elif header == b":path":
                    raw_path = value
                elif header == b":protocol":
                    protocol = value.decode()
                elif header and not header.startswith(b":"):
                    headers.append((header, value))
 
            if b"?" in raw_path:
                path_bytes, query_string = raw_path.split(b"?", maxsplit=1)
            else:
                path_bytes, query_string = raw_path, b""
            path = path_bytes.decode()
            self._quic._logger.info("HTTP request %s %s", method, path)
 
            # FIXME: add a public API to retrieve peer address
            client_addr = self._http._quic._network_paths[0].addr
            client = (client_addr[0], client_addr[1])
 
            handler: Handler
            scope: Dict
            if method == "CONNECT" and protocol == "websocket":
                subprotocols: List[str] = []
                for header, value in event.headers:
                    if header == b"sec-websocket-protocol":
                        subprotocols = [x.strip() for x in value.decode().split(",")]
                scope = {
                    "client": client,
                    "headers": headers,
                    "http_version": http_version,
                    "method": method,
                    "path": path,
                    "query_string": query_string,
                    "raw_path": raw_path,
                    "root_path": "",
                    "scheme": "wss",
                    "subprotocols": subprotocols,
                    "type": "websocket",
                }
                handler = WebSocketHandler(
                    connection=self._http,
                    scope=scope,
                    stream_id=event.stream_id,
                    transmit=self.transmit,
                )
            elif method == "CONNECT" and protocol == "webtransport":
                scope = {
                    "client": client,
                    "headers": headers,
                    "http_version": http_version,
                    "method": method,
                    "path": path,
                    "query_string": query_string,
                    "raw_path": raw_path,
                    "root_path": "",
                    "scheme": "https",
                    "type": "webtransport",
                }
                handler = WebTransportHandler(
                    connection=self._http,
                    scope=scope,
                    stream_id=event.stream_id,
                    transmit=self.transmit,
                )
            else:
                extensions: Dict[str, Dict] = {}
                if isinstance(self._http, H3Connection):
                    extensions["http.response.push"] = {}
                scope = {
                    "client": client,
                    "extensions": extensions,
                    "headers": headers,
                    "http_version": http_version,
                    "method": method,
                    "path": path,
                    "query_string": query_string,
                    "raw_path": raw_path,
                    "root_path": "",
                    "scheme": "https",
                    "type": "http",
                }
                handler = HttpRequestHandler(
                    authority=authority,
                    connection=self._http,
                    protocol=self,
                    scope=scope,
                    stream_ended=event.stream_ended,
                    stream_id=event.stream_id,
                    transmit=self.transmit,
                )
            self._handlers[event.stream_id] = handler
            asyncio.ensure_future(handler.run_asgi(application))
        elif (
            isinstance(event, (DataReceived, HeadersReceived))
            and event.stream_id in self._handlers
        ):
            handler = self._handlers[event.stream_id]
            handler.http_event_received(event)
        elif isinstance(event, DatagramReceived):
            handler = self._handlers[event.flow_id]
            handler.http_event_received(event)
        elif isinstance(event, WebTransportStreamDataReceived):
            handler = self._handlers[event.session_id]
            handler.http_event_received(event)
 
    def quic_event_received(self, event: QuicEvent) -> None:
        if isinstance(event, ProtocolNegotiated):
            if event.alpn_protocol in H3_ALPN:
                self._http = H3Connection(self._quic, enable_webtransport=True)
            elif event.alpn_protocol in H0_ALPN:
                self._http = H0Connection(self._quic)
        elif isinstance(event, DatagramFrameReceived):
            if event.data == b"quack":
                self._quic.send_datagram_frame(b"quack-ack")
 
        # ?pass event to the HTTP layer
        if self._http is not None:
            for http_event in self._http.handle_event(event):
                self.http_event_received(http_event)
 
 
class SessionTicketStore:
    """
    Simple in-memory store for session tickets.
    """
 
    def __init__(self) -> None:
        self.tickets: Dict[bytes, SessionTicket] = {}
 
    def add(self, ticket: SessionTicket) -> None:
        self.tickets[ticket.ticket] = ticket
 
    def pop(self, label: bytes) -> Optional[SessionTicket]:
        return self.tickets.pop(label, None)
 
 
if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="QUIC server")
    parser.add_argument(
        "app",
        type=str,
        nargs="?",
        default="api:app7",
        help="the ASGI application as <module>:<attribute>",
    )
    parser.add_argument(
        "-c",
        "--certificate",
        default="ssl_cert.pem",
        type=str, 
        help="load the TLS certificate from the specified file",
    )
    parser.add_argument(
        "--host",
        type=str,
        default="192.168.1.129",
        help="listen on the specified address (defaults to ::)",
    )
    parser.add_argument(
        "--port",
        type=int,
        default=28443,
        help="listen on the specified port (defaults to 4433)",
    )
    parser.add_argument(
        "-k",
        "--private-key",
        type=str,
        default="ssl_key.pem",
        help="load the TLS private key from the specified file",
    )
    parser.add_argument(
        "-l",
        "--secrets-log",
        type=str,
        help="log secrets to a file, for use with Wireshark",
    )
    parser.add_argument(
        "-q",
        "--quic-log",
        type=str,
        help="log QUIC events to QLOG files in the specified directory",
    )
    parser.add_argument(
        "--retry",
        action="store_true",
        help="send a retry for new connections",
    )
    parser.add_argument(
        "-v", "--verbose",default="True", action="store_true", help="increase logging verbosity"
    )
    args = parser.parse_args()
 
    logging.basicConfig(
        format="%(asctime)s %(levelname)s %(name)s %(message)s",
        level=logging.DEBUG if args.verbose else logging.INFO,
    )
 
    # import ASGI application
    module_str, attr_str = args.app.split(":", maxsplit=1)
    module = importlib.import_module(module_str)
    application = getattr(module, attr_str)
 
    # create QUIC logger
    if args.quic_log:
        quic_logger = QuicFileLogger(args.quic_log)
    else:
        quic_logger = None
 
    # open SSL log file
    if args.secrets_log:
        secrets_log_file = open(args.secrets_log, "a")
    else:
        secrets_log_file = None
 
    configuration = QuicConfiguration(
        alpn_protocols=H3_ALPN + H0_ALPN + ["siduck"],
        is_client=False,
        max_datagram_frame_size=65536,
        quic_logger=quic_logger,
        secrets_log_file=secrets_log_file,
    )
 
    # load SSL certificate and key
    configuration.load_cert_chain(args.certificate, args.private_key)
 
    ticket_store = SessionTicketStore()
 
    if uvloop is not None:
        uvloop.install()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(
        serve(
            args.host,
            args.port,
            configuration=configuration,
            create_protocol=HttpServerProtocol,
            session_ticket_fetcher=ticket_store.pop,
            session_ticket_handler=ticket_store.add,
            retry=args.retry,
        )
    )
    try:
        loop.run_forever()
    except KeyboardInterrupt:
        pass



```

api.py：

```
import datetime
import os
from urllib.parse import urlencode
 
import httpbin
from asgiref.wsgi import WsgiToAsgi
from starlette.applications import Starlette
from starlette.responses import PlainTextResponse, Response
from starlette.routing import Mount, Route, WebSocketRoute
from starlette.staticfiles import StaticFiles
from starlette.templating import Jinja2Templates
from starlette.types import Receive, Scope, Send
from starlette.websockets import WebSocketDisconnect
 
async def homepage(request):
    print('-----------------------')
    print(request.headers)
    return Response('123456', media_type='application/json')
    """
    Simple homepage.
    """
    print(request)
    print(request.headers)
    print(request.query_params)
    print(request.json)
    print(request.form)
    print(request.method)
    print(dir(request)) 
    content = request.body()
    media_type = request.headers.get("content-type")
    print('-----------------------',content,media_type)
    return Response('123456', media_type=media_type)
 
 
starlette = Starlette(
    routes=[
        Route("/api/app07", homepage), 
    ]
)
 
async def app7(scope: Scope, receive: Receive, send: Send) -> None: 
    await starlette(scope, receive, send)
    


```

这里的 / api/app07 一开始是不知道的，在模拟的时候会发现他提交的地址，然后改成这个的，然后就会发现，他请求的方式是 get，然后 headers 里面有个 x-data，这个应该就是请求的参数了。

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1ZzyMhQAVtG8n9tochzz6ajWpv54mm3UaRYqHHcCeQavdvIQruIsv1Usn6PSG7OgR1s1yskOicialQQ/640?wx_fmt=png)

然后尝试使用这个请求参数进行提交，发现服务器返回的数据如下：{"status":null,"state":null,"data":[{"value":"8124"},{"value":"7445"},{"value":"78"},{"value":"7530"},{"value":"8058"},{"value":"1177"},{"value":"6786"},{"value":"8510"},{"value":"2410"},{"value":"5905"},{"value":"8792"},{"value":"3595"},{"value":"4907"},{"value":"8898"},{"value":"4708"},{"value":"3810"},{"value":"8825"},{"value":"5726"},{"value":"2637"},{"value":"8576"}]} 这正是我们想要的数据，那么这个 x-data 是怎么来的呢？因为他是在 so 里面进行的 udp 发包，所以肯定会用到 sendto 函数，然后我们在 ida 搜索 sendto，再继续往上查找引用，最终定位到 sub_5CBA0 函数，这个函数应该就是进行请求的函数了，接下来我们分析这个函数通过刚才的 sendto 调用关系，我们知道 sub_638FC 是发包的地方，所以加密肯定是在 sub_638FC 之前进行的，而且他的第三个参数 v164 就是加密后的数据，所以我们跟着 v164 往上找，发现 v164 在 2 个地方进行过改动，改动都是类似下面这种的

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1ZzyMhQAVtG8n9tochzz6ajibRWSG2WEUbn1PDHibDD8dMSsT4gWzq1miafiaicsWHBCM6adVmwibN8rbYw/640?wx_fmt=png)

然后使用 fridahook 到这部分，发现 v43 和 v44 是固定值，v45 等于 v46 是时间戳，然后下面进行了一系列异或运算，最后赋值给 v164，然后通过 hook 参数得知这部分是 x-data 前半部分的加密，后半部分是差不多的，只不过算法改动了一下，后半部分代码如下

![](https://mmbiz.qpic.cn/mmbiz_png/8ib9picwJag1ZzyMhQAVtG8n9tochzz6ajiaiaLMbPmyjtpWynsjGjDangycU55wJCAza2BgicmkMQlTuG18HibAQx7g/640?wx_fmt=png)

然后我们可以通过 python 构建计算 x-data 的函数，代码如下

```
def rev(data):
    aacon=''
    for i in range(0,9,2):
        t=8-i
        aacon+=data[t:t+2]
    return aacon
     
def calcsign(page):
    a3=page
    v43=0x8a4162d9
    v44=0xf020eadf
    v42=int(time.time()) 
    v45=v42 
    v155 = v43 ^ a3 ^ v45 
    v47 = v44
    v1552 = v47 ^ v45 
    aa = hex(v155).replace('0x','').zfill(8)
    bb = hex(v1552).replace('0x','').zfill(8)
    aa=rev(aa)
    bb=rev(bb)
    return aa+bb 
print(calcsign(1))



```

试了一下，提交这个 x-data，返回了正确的数据然后我们就可以写出请求的代码：

```
import argparse
import asyncio
import logging
import os
import pickle
import ssl
import time
from collections import deque
from typing import BinaryIO, Callable, Deque, Dict, List, Optional, Union, cast
from urllib.parse import urlparse
 
import wsproto
import wsproto.events
 
import aioquic
from aioquic.asyncio.client import connect
from aioquic.asyncio.protocol import QuicConnectionProtocol
from aioquic.h0.connection import H0_ALPN, H0Connection
from aioquic.h3.connection import H3_ALPN, ErrorCode, H3Connection
from aioquic.h3.events import (
    DataReceived,
    H3Event,
    HeadersReceived,
    PushPromiseReceived,
)
from aioquic.quic.configuration import QuicConfiguration
from aioquic.quic.events import QuicEvent
from aioquic.quic.logger import QuicFileLogger
from aioquic.tls import CipherSuite, SessionTicket
 
 
def rev(data):
    aacon=''
    for i in range(0,9,2):
        t=8-i
        aacon+=data[t:t+2]
    return aacon
     
def calcsign(page):
    a3=page
    v43=0x8a4162d9
    v44=0xf020eadf
    v42=int(time.time()) 
    v45=v42 
    v155 = v43 ^ a3 ^ v45 
    v47 = v44
    v1552 = v47 ^ v45 
    aa = hex(v155).replace('0x','').zfill(8)
    bb = hex(v1552).replace('0x','').zfill(8)
    aa=rev(aa)
    bb=rev(bb)
    return aa+bb 
#print(calcsign(1))
 
 
try:
    import uvloop
except ImportError:
    uvloop = None
 
logger = logging.getLogger("client")
 
global sumdata
sumdata=0
HttpConnection = Union[H0Connection, H3Connection]
 
USER_AGENT = "quiche"
 
 
class URL:
    def __init__(self, url: str) -> None:
        parsed = urlparse(url)
 
        self.authority = parsed.netloc
        self.full_path = parsed.path or "/"
        if parsed.query:
            self.full_path += "?" + parsed.query
        self.scheme = parsed.scheme
 
 
class HttpRequest:
    def __init__(
        self,
        method: str,
        url: URL,
        content: bytes = b"",
        headers: Optional[Dict] = None,
    ) -> None:
        if headers is None:
            headers = {}
 
        self.content = content
        self.headers = headers
        self.method = method
        self.url = url
 
 
class WebSocket:
    def __init__(
        self, http: HttpConnection, stream_id: int, transmit: Callable[[], None]
    ) -> None:
        self.http = http
        self.queue: asyncio.Queue[str] = asyncio.Queue()
        self.stream_id = stream_id
        self.subprotocol: Optional[str] = None
        self.transmit = transmit
        self.websocket = wsproto.Connection(wsproto.ConnectionType.CLIENT)
 
    async def close(self, code: int = 1000, reason: str = "") -> None:
        """
        Perform the closing handshake.
        """
        data = self.websocket.send(
            wsproto.events.CloseConnection(code=code, reason=reason)
        )
        self.http.send_data(stream_id=self.stream_id, data=data, end_stream=True)
        self.transmit()
 
    async def recv(self) -> str:
        """
        Receive the next message.
        """
        return await self.queue.get()
 
    async def send(self, message: str) -> None:
        """
        Send a message.
        """
        assert isinstance(message, str)
 
        data = self.websocket.send(wsproto.events.TextMessage(data=message))
        self.http.send_data(stream_id=self.stream_id, data=data, end_stream=False)
        self.transmit()
 
    def http_event_received(self, event: H3Event) -> None:
        if isinstance(event, HeadersReceived):
            for header, value in event.headers:
                if header == b"sec-websocket-protocol":
                    self.subprotocol = value.decode()
        elif isinstance(event, DataReceived):
            self.websocket.receive_data(event.data)
 
        for ws_event in self.websocket.events():
            self.websocket_event_received(ws_event)
 
    def websocket_event_received(self, event: wsproto.events.Event) -> None:
        if isinstance(event, wsproto.events.TextMessage):
            self.queue.put_nowait(event.data)
 
 
class HttpClient(QuicConnectionProtocol):
    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, **kwargs)
 
        self.pushes: Dict[int, Deque[H3Event]] = {}
        self._http: Optional[HttpConnection] = None
        self._request_events: Dict[int, Deque[H3Event]] = {}
        self._request_waiter: Dict[int, asyncio.Future[Deque[H3Event]]] = {}
        self._websockets: Dict[int, WebSocket] = {}
 
        if self._quic.configuration.alpn_protocols[0].startswith("hq-"):
            self._http = H0Connection(self._quic)
        else:
            self._http = H3Connection(self._quic)
 
    async def get(self, url: str, headers: Optional[Dict] = None) -> Deque[H3Event]:
        """
        Perform a GET request.
        """
        return await self._request(
            HttpRequest(method="GET", url=URL(url), headers=headers)
        )
 
    async def post(
        self, url: str, data: bytes, headers: Optional[Dict] = None
    ) -> Deque[H3Event]:
        """
        Perform a POST request.
        """
        return await self._request(
            HttpRequest(method="POST", url=URL(url), content=data, headers=headers)
        )
 
    async def websocket(
        self, url: str, subprotocols: Optional[List[str]] = None
    ) -> WebSocket:
        """
        Open a WebSocket.
        """
        request = HttpRequest(method="CONNECT", url=URL(url))
        stream_id = self._quic.get_next_available_stream_id()
        websocket = WebSocket(
            http=self._http, stream_id=stream_id, transmit=self.transmit
        )
 
        self._websockets[stream_id] = websocket
 
        headers = [
            (b":method", b"CONNECT"),
            (b":scheme", b"https"),
            (b":authority", request.url.authority.encode()),
            (b":path", request.url.full_path.encode()),
            (b":protocol", b"websocket"),
            (b"user-agent", USER_AGENT.encode()),
            (b"sec-websocket-version", b"13"),
        ]
        if subprotocols:
            headers.append(
                (b"sec-websocket-protocol", ", ".join(subprotocols).encode())
            )
        self._http.send_headers(stream_id=stream_id, headers=headers)
 
        self.transmit()
 
        return websocket
 
    def http_event_received(self, event: H3Event) -> None:
        if isinstance(event, (HeadersReceived, DataReceived)):
            stream_id = event.stream_id
            if stream_id in self._request_events:
                # http
                self._request_events[event.stream_id].append(event)
                if event.stream_ended:
                    request_waiter = self._request_waiter.pop(stream_id)
                    request_waiter.set_result(self._request_events.pop(stream_id))
 
            elif stream_id in self._websockets:
                # websocket
                websocket = self._websockets[stream_id]
                websocket.http_event_received(event)
 
            elif event.push_id in self.pushes:
                # push
                self.pushes[event.push_id].append(event)
 
        elif isinstance(event, PushPromiseReceived):
            self.pushes[event.push_id] = deque()
            self.pushes[event.push_id].append(event)
 
    def quic_event_received(self, event: QuicEvent) -> None:
        # ?pass event to the HTTP layer
        if self._http is not None:
            for http_event in self._http.handle_event(event):
                self.http_event_received(http_event)
 
    async def _request(self, request: HttpRequest) -> Deque[H3Event]:
        stream_id = self._quic.get_next_available_stream_id()
        self._http.send_headers(
            stream_id=stream_id,
            #headers=[(k.encode(), v.encode()) for (k, v) in request.headers.items()],
            headers=[
                (b":method", request.method.encode()),
                (b":scheme", request.url.scheme.encode()),
                (b":authority", request.url.authority.encode()),
                (b":path", request.url.full_path.encode()),
                (b"user-agent", USER_AGENT.encode()),
            ]
            + [(k.encode(), v.encode()) for (k, v) in request.headers.items()],
            end_stream=not request.content,
        )
        if request.content:
            self._http.send_data(
                stream_id=stream_id, data=request.content, end_stream=True
            )
 
        waiter = self._loop.create_future()
        self._request_events[stream_id] = deque()
        self._request_waiter[stream_id] = waiter
        self.transmit()
 
        return await asyncio.shield(waiter)
 
 
async def perform_http_request(
    client: HttpClient,
    url: str, 
    include: bool,
    output_dir: Optional[str],
    headers: Optional[str]
) -> None:
    # perform request
    start = time.time()
    global sumdata
      
    http_events = await client.get(url,headers=headers)
    method = "GET"
    elapsed = time.time() - start
 
    # print speed
    octets = 0
    for http_event in http_events:
        if isinstance(http_event, DataReceived):
            octets += len(http_event.data)
    print(
        "Response received for %s %s : %d bytes in %.1f s (%.3f Mbps)"
        % (method, urlparse(url).path, octets, elapsed, octets * 8 / elapsed / 1000000)
    )
    sumdata=http_event.data
    # output response
    if output_dir is not None:
        output_path = os.path.join(
            output_dir, os.path.basename(urlparse(url).path) or "index.html"
        )
        with open(output_path, "wb") as output_file:
            write_response(
                http_events=http_events, include=include, output_file=output_file
            )
 
 
def process_http_pushes(
    client: HttpClient,
    include: bool,
    output_dir: Optional[str],
) -> None:
    for _, http_events in client.pushes.items():
        method = ""
        octets = 0
        path = ""
        for http_event in http_events:
            if isinstance(http_event, DataReceived):
                octets += len(http_event.data)
            elif isinstance(http_event, PushPromiseReceived):
                for header, value in http_event.headers:
                    if header == b":method":
                        method = value.decode()
                    elif header == b":path":
                        path = value.decode()
        logger.info("Push received for %s %s : %s bytes", method, path, octets)
 
        # output response
        if output_dir is not None:
            output_path = os.path.join(
                output_dir, os.path.basename(path) or "index.html"
            )
            with open(output_path, "wb") as output_file:
                write_response(
                    http_events=http_events, include=include, output_file=output_file
                )
 
 
def write_response(
    http_events: Deque[H3Event], output_file: BinaryIO, include: bool
) -> None:
    global sumdata
    for http_event in http_events:
        
        if isinstance(http_event, HeadersReceived) and include:
            headers = b""
            for k, v in http_event.headers:
                headers += k + b": " + v + b"\r\n"
            if headers: 
                output_file.write(headers + b"\r\n")
        elif isinstance(http_event, DataReceived):
            if(http_event.data):
                sumdata = http_event.data
            #print(http_event.data)
            output_file.write(http_event.data)
 
 
def save_session_ticket(ticket: SessionTicket) -> None:
    """
    Callback which is invoked by the TLS engine when a new session ticket
    is received.
    """
    logger.info("New session ticket received")
    #if args.session_ticket:
    #    with open(args.session_ticket, "wb") as fp:
    #        pickle.dump(ticket, fp)
 
 
 
async def run(page):
    configuration = QuicConfiguration(
        is_client=True, alpn_protocols=H3_ALPN
    )
    configuration.load_verify_locations('./ssl_cert_with_chain.pem')
    host='180.76.60.244'
    port=28443
    async with connect(
        host,
        port,
        configuration=configuration,
        create_protocol=HttpClient,
        session_ticket_handler=save_session_ticket,
        local_port=0,
        wait_connected=1,
    ) as client:
        client = cast(HttpClient, client)
        url='https://180.76.60.244:28443/api/app07' 
        include=''
        output_dir=''
        headers = { 
                'x-data': calcsign(page)
            } 
        await perform_http_request(
                client=client,
                url=url, 
                include=include,
                output_dir=output_dir,
                headers=headers
            ) 
            # process http pushes
        process_http_pushes(client=client, include=include, output_dir=output_dir)
        client._quic.close(error_code=ErrorCode.H3_NO_ERROR)
 
import json
async def main():
    global sumdata
    count2=0
    for i in range(1,101):
        now=-1
        while(1): 
            now=-1
            try:
                await run(i)
                print(sumdata)
                #sumdata=json.loads(sumdata)
                count = 0
                #print(sumdata)
                for j in sumdata['data']:
                    count+=int(j['value']) 
                now=count
            except Exception as ex:
                print(ex)
                now=-1
            if(now!=-1):
                break
            else:
                time.sleep(1)
        print(i,now)
        count2+=now 
    print('ok',count2)
      
loop = asyncio.get_event_loop()
loop.run_until_complete(
    main( )
)


```

PS: 这个代码是改的 aioquic 里面的 demo，由于它里面都是异步的而且我懒得改动，所以代码写的比较烂，但是是可以正常运行得到数据的。最终，我们得到了正确的 flag 然后提交，此题结束！

附上：

本人的 github：https://github.com/zixing131

本人的 52pojie：https://www.52pojie.cn/home.php?mod=space&uid=358970

Darbra 老师的 52pojie：https://www.52pojie.cn/home.php?mod=space&uid=1410198  

Darbra 老师的 github：https://github.com/darbra/sperm