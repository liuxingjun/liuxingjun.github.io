---
title: swoole-example
date: 2018-04-17 14:16:01
tags:
  - swoole
  - websocket
categories: php
---


## 创建websocket服务对象

```php
$server = new swoole_websocket_server("0.0.0.0", 9501);
```

## 收到消息回调
```php
$server->on('message', function (swoole_websocket_server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    foreach ($server->connections as $fd) {
        $server->push($fd, json_encode([$frame->fd,$frame->data]));
    }
});
```
<!-- more -->
## 关闭连接回调
```php
$server->on('close', function ($ser, $fd) {
        echo "client {$fd} closed\n";
});
```
## 启动服务
```php
$server->start();
```

## 客户端cli
```php
$longopt = array(
    'msg:',
);
$param = getopt('', $longopt);
$msg = array_key_exists('msg',$param)?$param['msg']:null ;

$cli = new swoole_http_client('127.0.0.1', 9501);
$cli->on('message', function ($cli, $frame) {
   $cli->close();
});
$cli->upgrade('/', function ($cli) use ($msg) {
    $cli->push($msg);
});
```
在命令行执行
```shell
$ php ws_client.php --msg=大家好
```


## 客户端http
如果你的项目无法使用cli
你可以使用下面的方式：
1. 服务端增加 onRequest事件
```php
$server->on('request', function (swoole_http_request $request, swoole_http_response $response) {
    global $server;//调用外部的server
    // $server->connections 遍历所有websocket连接用户的fd，给所有用户推送
    foreach ($server->connections as $fd) {
        echo $request->fd."\n";
        //因为自己并不是websocket客户端，所以广播给除自己以外的客户端
        if($request->fd != $fd ){
            $server->push($fd, $request->get['message']);
        }
    }
    $response->end("ok");
    //关闭当前客户端请求，防止下次全员推送时包含非websocket客户端
    $server->close($request->fd);
});
```
  > 依赖 [pcre](http://www.pcre.org/ "pcre")库

2. 使用任何http请求 http://127.0.0.1:9501/ws_server.php?message=大家好 

