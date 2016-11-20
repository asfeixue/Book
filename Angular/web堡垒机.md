web堡垒机

#背景
堡垒机后端由jsch负责与具体的linux服务器交互，将输入输出流获取到之后，通过线程与前端的websocket建立对应关系即可建立数据交互通道。前端这里使用xterm.js负责控制台的处理。

#xterm介绍
源码参考：https://github.com/sourcelair/xterm.js/

这里主要记录如何使用此库。

没找到合适的办法进行懒加载处理，所以直接如下使用。
```
  <!-- jQuery -->
  <script src="vendor/jquery/jquery.min.js"></script>
  这里！！！！
  <script src="vendor/jquery/xterm/xterm.js"></script>

  <!-- Angular -->
  <script src="vendor/angular/angular.js"></script>
```

##html部分
```
<div ng-controller="keyboxCtrl">
    <div class="panel panel-default">
        <div class="panel-heading">
            <span>机器信息</span>
        </div>
        <div id="xterm-container"></div>
    </div>
</div>
```

##js部分
```
app.controller('keyboxCtrl', function($scope, $localStorage, toaster, httpService) {
    var server = httpService.wsInstance();

    var term = new Terminal({
        colors: Terminal.colors,
        theme: 'default',
        convertEol: false,
        termName: 'xterm',
        geometry: [140, 24],
        cursorBlink: true,
        visualBell: false,
        popOnBell: false,
        scrollback: 1000,
        screenKeys: false,
        debug: false,
        cancelEvents: false
    });

    term.open(document.getElementById('xterm-container'));

    term.on("data", function(data) {
        var requestBody = {
            token: $localStorage.token,
            requestType: "cmd",
            body: data
        }

        server.send(JSON.stringify(requestBody));
    });

    server.onMessage(function(data) {
        var msg = JSON.parse(data.data).output.toString();

        term.write(msg);
    });

    var initContext = {
        token : $localStorage.token,
        requestType : "init",
        body : {
            user : "xxx",
            host : "127.0.0.1",
            port : 22
        }
    };

    server.send(JSON.stringify(initContext));
```

在具体使用的时候，走了不少弯路。总想着前端要介入做些输入控制，响应渲染控制之类的。其实，shell已经很成熟了，直接简单的驱动使用起来就可以了。