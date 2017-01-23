+++
date = "2017-01-22T18:11:48+08:00"
title = "前端使用websocket的问题"
draft = false
+++

　　

> websocket相关概念

* [websocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

> 随着互联网的发展，传统的HTTP协议已经很难满足Web应用日益复杂的需求了。近年来，随着HTML5的诞生，WebSocket协议被提出，它实现了浏览器与服务器的全双工通信，扩展了浏览器与服务端的通信功能，使服务端也能主动向客户端发送数据。
IE10以上才支持HTML5，才支持websocket，chrome，火狐，opera等浏览器支持较好。

* [Socket.IO](http://socket.io/)

> WebSocket是HTML5的一种新通信协议，它实现了浏览器与服务器之间的双向通讯。而Socket.IO是一个完全由JavaScript实现、基于Node.js、支持WebSocket的协议用于实时通信、跨平台的开源框架，它包括了**客户端的JavaScript**和**服务器端的Node.js**。
Socket.IO是一个跨平台，支持多种连接方式，如websocket，flashsocket,ajax等，如果客户端不支持websocket时，可以切换使用其他链接方式，比如ajax轮询等。

```
<script src="/lib/socket.io/socket.io.js"></script>
<script>
   var socket = io();
   socket.on('connect', function() {
           /* 具体操作 */
   });
</script>
```

最近笔者新参与的一个web项目，拟定采用vue2.0来编写，期间遇到有关使用websocket的问题，记录一下，个中遇到的一些问题和解决方法，分享给有需要的人。

当前服务端采用了websocket的方式，这需要客户端也相应的采用websocket来与客户端建立ws连接，这样客户端和服务端就可以双向的发送数据。笔者当时一拿到这个就会觉得是不是又需要npm安装什么包似的，因为这个功能是从另一个项目（暂且称其为项目B）的功能迁移过来的，而项目B前端是用的angularjs开发的，前端直接用的angularjs的一个websocket的包（angular-websocke），然后几行代码就敲定了，创建连接，发送数据，接收数据，很是方便。当前项目采用的是vue框架，很自然的就去搜有木有vue的websocket包，很快找到了vue-websocket，安装好后，调试，直接报错：
　　
```
XMLHttpRequest cannot load http://192.168.0.239:9000/socket.io/?EIO=3&transport=polling&t=LbjddEK. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. The response had HTTP status code 404.
```

笔者对比了项目B，同样在客户端启动本地server，能正常创建到服务端的ws连接，不会像http请求一样有跨域的问题，同时另一问题就是，明明是要创建ws连接，为何浏览器的控制台里会提示是http的连接？vue-websocket没有调试成功，马上去找找有没有通用的包，于是在github找了js相关的websocket的相关包，比如ws，笔者对照了angularjs-websocket包，它也是依赖的ws。于是按照ws的说明文档，安装，然后webpack直接报错了：

```
ERROR in ./~/ws/lib/WebSocketServer.js
Module not found: Error: Cannot resolve module 'tls' in xx\node_modules\ws\lib
@ ./~/ws/lib/WebSocketServer.js 15:10-24
ERROR in ./~/options/lib/options.js
Module not found: Error: Cannot resolve module 'fs' in xx\node_modules\options\lib
@ ./~/options/lib/options.js 6:9-22
```

查找资料，说是这两个包是服务端node用的，客户端没有所以导致报错，需要修改webpack的配置文件，使其不加载客户端不需要的包
[https://github.com/webpack/react-starter/issues/3#issuecomment-53395089](https://github.com/webpack/react-starter/issues/3#issuecomment-53395089)
然后，webpack不报错了，但是前端调用ws生成实例的时候，依然会报错，因为ws链接创建的时候，先发起了http的链接请求，按照ws文档：
```
var WebSocket = require('ws')
var ws = new WebSocket('ws://xxx/ws', 'ws')
```
在代码里，生成ws实例的时候，还是会产生http的链接，跟前面用vue-websocket的情况类似，因为是在本地启动的前端服务，所以这样发起到服务器的请求，会导致跨越报错。奇怪的是项目B里的angular使用的angular-websocket包，同样是基于ws包做的，在本地启动，是直接创建的ws链接，没有跨越的问题。然后在vue论坛上查找相关资料，类似的问题比如：[https://forum-archive.vuejs.org/topic/1293/webpack-bundling-issue-with-websockets/3](https://forum-archive.vuejs.org/topic/1293/webpack-bundling-issue-with-websockets/3)
有人提到：Can't you just use window.WebSocket instead of ws?
提醒到了我，我摒弃了ws，直接用window.WebSocket，来创建ws连接，果然成功了。那为啥angular-websocket基于ws，能正常在浏览器执行，vue这边直接用ws不行，于是去查看angular-websocket的源码，才发现angular-websocket基于ws做了封装，它提供的$websocket对象，在当前浏览器环境下，本质上也是window.WebSocket的对象。
```
import angular from 'angular';
var Socket;
if (typeof window === 'undefined') {
try {
var ws = require('ws');
Socket = (ws.Client||ws.client||ws);
} catch(e) {}
}
// Browser
Socket = (Socket || window.WebSocket || window.MozWebSocket);
////////////////////////////////////////////////////////////
function $WebSocketBackendProvider($log) {
this.create = function create(url, protocols) {
var match = /wss?:\/\//.exec(url);
if (!match) {
throw new Error('Invalid url provided');
}
if (protocols) {
return new Socket(url, protocols);
}
return new Socket(url);
};
```
看源码里面，angular-websocket也是用window.WebSocket创建的，只有在没有window对象的时候，才会基于ws包来创建websocket对象。然后瞬间感觉找准了方向，目前我们的场景主要是PC端的浏览器使用，可以直接用window.WebSocket对象来，于是查阅了[window.WebSocket的文档](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)，很快问题就解决了，原来自带的window对象就已经可以解决问题了，自己绕了一大圈，还是回到原点，细看了它文档里写的相关方法，确实和前面讲到的angular-websocket很像，确实angular-websocket是对这些方法，比如onopen，send，onmessange等方法进行了封装，让开发可以更方便的调用。

另外，文章最前面提到的[Socket.IO](http://socket.io/)，笔者在客户端安装了Socket.IO，并且按照文档说明，去建立ws链接，发现以下问题：

#### 1、安装socket.io-client，安装完之后，webpack报错
```
ERROR in ./~/socket.io-client/~/component-emitter/index.js
Module build failed: Error: ENOENT: no such file or directory, open 'xx\node_modules\socket.io-client\node_modules\component-emitter\index.js'
    at Error (native)
 @ ./~/socket.io-client/lib/manager.js 8:14-42
ERROR in ./~/engine.io-client/~/component-emitter/index.js
Module build failed: Error: ENOENT: no such file or directory, open 'xx\node_modules\engine.io-client\node_modules\component-emitter\index.js'
    at Error (native)
 @ ./~/engine.io-client/lib/socket.js 6:14-42
 ```

参考[https://github.com/socketio/socket.io-client/issues/933](https://github.com/socketio/socket.io-client/issues/933)这个需要修改一下webpack的配置文件，添加以下两行后，webpack可以打包通过。
```
module: {
    noParse: ['ws']
  },
  externals: ['ws']
```

#### 2、创建连接报错
创建连接`var socket = io(url);`，这样创建的链接，Socket.IO默认是按轮询方式发起的http请求（很奇怪，当前浏览器明明是支持websocket的），这样首先就出现了前面的http跨域请求报错：
```
XMLHttpRequest cannot load http://192.168.0.239:9000/socket.io/?EIO=3&transport=polling&t=LbjddEK. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8080' is therefore not allowed access. The response had HTTP status code 404.
```

查阅资料，创建连接的时候，可以指定参数`io(WS_URL, {transports: ['websocket', 'polling', 'flashsocket']})`，设置其发起websocket链接，这样在console里看到的确实是发起的ws请求，然而依然报404错：
```
WebSocket connection to 'ws://192.168.0.239:9000/socket.io/?EIO=3&transport=websocket' failed: Error during WebSocket handshake: Unexpected response code: 404
```

提示是404找不到，确实是服务端没有`'ws://192.168.0.239:9000/socket.io/?EIO=3&transport=websocket'`这样的url，这种url是默认的ws服务端的url，可以设置 path，path默认是`/socket.io`，设置path后，`io(WS_URL, {path: '/ws', transports: ['websocket', 'polling', 'flashsocket']})`，依然报301错：
```
WebSocket connection to 'ws://192.168.0.239:9000/ws/?EIO=3&transport=websocket' failed: Error during WebSocket handshake: Unexpected response code: 301
```

笔者对比了成功建立ws链接时的url，`ws://192.168.0.239:9000/ws`，然后用工具测试了下，只有path是`/ws`，即ws://192.168.0.239:9000/ws?EIO=3&transport=websocket`这样的url才能成功建立ws链接，这里报301的错误，是因为socketio会在path后面添加`/?EIO=3&transport=websocket`这样一串参数，这样导致了服务端路由匹配不上，导致出现了404或301的情况，所以这里需要服务端配合调整一下路由规则，即可。

最终，结合当前的实际情况还是采用了原生的websocket，Socket.IO对url的格式有一定的规范要求，目前查阅了socketio的相关api文档，尚未找到可以自定义url格式的方法，比较适合客户端和服务端都同时可控的情况。


