# Websocket

## 基本原理

laravel使用redis来实现broadcast的基本原理是：

laravel事件触发，该事件由于实现了 ShouldBroadcastNow接口，会通过Redis的Publish命令发送信息。

我们建立的websocket server会监听Redis Publish的信息。然后把该信息通过websocket协议传送给前端。  


## Build Websocket Server

服务器端需要Build Websocket Server. 在这里使用laravel echo server\(一个在socket.io基础上建立的第三方开源nodejs websocket server\)

`<any dir>$ npm install -g laravel-echo-server`

init:

`<project root>$ laravel-echo-server init`

该命令会在当前目录生成laravel-echo-server.json文件。可以进一步配置该文件。见后面补充说明。

start:

`<project root>$ laravel-echo-server start`

接下来需要写event并implement ShouldBroadcastNow接口，以下是一个典型的简单broadcast event类

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class LotteryBroadcast extends Event implements ShouldBroadcastNow
{
    public $user_id;
    public $name;

    /**  * Create a new event instance.  *  * @param $user_id */
    public function __construct($user_id)
    {
        $this->user_id = $user_id;
        $this->name = 'xu';
    }

    public function broadcastOn()
    {
        return ['test-channel-' . $this->user_id];
    }

    public function broadcastAs()
    {
        return 'BroadcastEvent';
    }
}
```

> public成员变量默认都会作为信息发送。
>
> broadcastOn返回的是channel名，数组是laravel5.1的格式。laravel 5.4以上返回的是Channel或者PrivateChannel对象。
>
> broadcastAs可以定义对于前端的事件名，如果没有该方法，会使用类名。
>
> 这两个命名都需要前端监听时用到！！

event里如果返回PrivateChannel like this:

```php
/**
 * Get the channels the event should broadcast on.
 *
 * @return array
 */
public function broadcastOn()
{
    return new PrivateChannel('order.'.$this->update->order_id);
}
```

对应的真实channel名会加前缀'private-'

## Frontend Connection

前端需要安装laravel-echo npm package并import, require, compile

```javascript
window.echo = require('laravel-echo');
// import Echo from 'laravel-echo';
// window.Pusher = require('pusher-js');
// window.Echo = new Echo({// broadcaster: 'socket.io',// host: '127.0.0.1:6001'// });
```

在需要建立websocket connection的page，需要引用socket.io.js和前面编译好的js

```markup
<script type="text/javascript" src="http://localhost:6001/socket.io/socket.io.js"></script>
<script type="text/javascript" src="app.js"></script>
```

> http://localhost:6001/需要换成相应的server host

然后建立websocket连接并监听处理事件：

```markup
<script type="text/javascript">
    echo1 = new echo({ broadcaster: 'socket.io', host: 'http://localhost:6001', });

    var id = 1;
    echo1.channel('test-channel-' + id).listen('.BroadcastEvent', (e) => { console.log(e); });
</script>
```

以上是对于**公有频道 \( public channel \)**。

{% hint style="info" %}
p.s. 关于listen的event名字（注意前面的dot！！）

If you customize the broadcast name using the broadcastAs method, you should make sure to register your listener with a leading dot \(`.`\) character. This will instruct Echo to not prepend the application's namespace to the event
{% endhint %}

  
对于**私有频道 \( private channel \)**：

```markup
<script type="text/javascript">
    var token = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...';
    echo1 = new echo({ 
        broadcaster: 'socket.io', 
        host: 'http://localhost:6001', 
        auth: { headers: { 'Authorization': 'Bearer ' + token } } 
    });

    var id = 1;
    echo1.private('test-channel-' + id).listen('.BroadcastEvent', (e) => { console.log(e); });
</script>
```

  
私有频道需要验证（上面使用的jwt token验证）。为了实现验证，需要确认两件事：

First, 在`Providers/BroadcastServiceProvider.php`里的`boot()`里确认或修改route的属性，比如配置中间件。

`Broadcast::routes([ 'middleware' => [ 'auth:api' ] ]);`

Second, 在`routes/channels.php`里设置频道验证：

`Broadcast::channel('test-channel', function ($user){  return true;});`

这里要跟channel name匹配（没有private前缀）

## Testing/Debug

由于Laravel Broadcast使用的本质上的Redis的Publish命令作为给websocket server发送信息的跳板，我们可以直接使用Redis的命令来实现数据发送。

Laravel Broadcast 对应的Redis Publish命令:

```php
PUBLISH test-channel '{"event":"BroadcastEvent","data":{"user_id":111,"name":"xu","socket":null},"socket":null}'
```

  
对应的Laravel Redis Facade命令:

```php
Redis::publish('test-channel', json_encode(["event"=>"BroadcastEvent","data"=>["user_id"=>111,"name"=>"xu","socket"=>null],"socket"=>null]));
```

Then we can check if laravel echo has message output on console. \(make sure`laravel-echo-server.json` -&gt; `"devMode": true`\)

## Miscellaneous

* 如果Redis有密码需要在 laravel-echo-server.json 设置： "databaseConfig": {"redis": { "password": "123456"},"sqlite": { "databasePath": "/database/laravel-echo-server.sqlite"}}



* SSL方式（https）需要在 laravel-echo-server.json 设置：protocol需要配置成https必须配置sslCertPath和sslKeyPathhost可以配置成null  &lt;?&gt;



* 在生产系统部署的一个Problem

在一个CentOS系统启动laravel-echo-server。只要客户端一连接server，马上crash，报segmentation fault。查到一个post：

[https://github.com/tlaverdure/laravel-echo-server/issues/191](https://github.com/tlaverdure/laravel-echo-server/issues/191)

{% tabs %}
{% tab title="Post" %}
Sorry for the late,Well, the problem was solved by downloading the NodeJS from [https://nodejs.org](https://nodejs.org/)

> DO NOT INSTALL nodejs from EPEL Repository or other repo, it'll crash \(Segmentation Fault\).

Solution on my environment:

* OS: Linux CentOS 7
* Kernel: Linux 3.10.0-514.26.2.el7.x86\_64

`$ sudo yum groupinstall 'Development Tools'`

`$ cd /opt`

`$ curl --silent --location` [`https://rpm.nodesource.com/setup_6.x`](https://rpm.nodesource.com/setup_6.x) `| sudo bash -`

`$ yum -y install nodejs`

Hope can help...
{% endtab %}
{% endtabs %}

重新按该post安装nodejs，再启动laravel-echo-server，连接正常，成功解决。

