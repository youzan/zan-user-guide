swoole_http_server 简单用法示例
=====================================

1. swoole_http_server 成员函数
    具体参考 API 文档：http://zandoc.zanphp.io/classes/swoole_http_server.html

    swoole_http_server 继承自 swoole_server，新增了事件回调 onRequest;


2. 使用示例
::

    <?php
    ##########先看一个最简单的 http_server 示例
    $serv = new swoole_http_server("127.0.0.1", 9502);

    $serv->set(array(
        'net_worker_num' => 1,
        'worker_num' => 1,
        'daemonize' => false,
        'log_file' => '/tmp/swoole_log.log',
    ));


    $serv->on('Request', function($request, $response) {
        var_dump($request->server);
        
        $response->cookie("User", "Swoole");
        $response->header("X-Server", "Swoole");
        $response->end("<h1>Hello Swoole!</h1>");
    });

    $serv->start();

::

    使用 curl 测试上面的 http_server: curl http://127.0.0.1:9502
    输出：<h1>Hello Swoole!</h1>

3. 我们可以在 onRequest 里干点别的事
::
    
    <?php
    #############仅用来测试，例子本身并没有什么用
    #############我们在 onRequest 事件中，新建一个 http_client，向另一个 http_server 发送 get 请求
    $serv = new swoole_http_server("0.0.0.0", 9502);

    $serv->set([
        'net_worker_num' => 1,
        'worker_num' => 1,
        'daemonize' => false,
        'log_file' => '/tmp/swoole_log.log',
    ]);

    $serv->on('Start', function() use($serv) {
        echo "Http server start!\n";
    });

    $serv->on('WorkerStart', function($serv, $worker_id){
        echo "Http server worker start, worker_id:" . $worker_id . "\n";
    });

    $serv->on('Request', function($req, $resp) {
        $cli = new swoole_http_client("127.0.0.1",  9503);

        $timerId = swoole_timer_after(5000, static function() use($cli) {
            echo "client timeout\n";
            $cli->close();
        });

        $cli->on("close", static function() use($timerId) {
            if (swoole_timer_exists($timerId)) {
                swoole_timer_clear($timerId);
            }
            echo "client onClosed\n";
        });

        $cli->get("/", function($res) use($timerId, $resp, $cli) {
            if (swoole_timer_exists($timerId)) {
                swoole_timer_clear($timerId);
            }
            
            $resp->status(200);
            $resp->end($res->body);
            $cli->close();
        });
    });

    $serv->start();

4. 根据 API 处理 Request
    如下示例中，我们处理了 http://127.0.0.1:9502/getbaiduip 这个 API 的请求

::

    <?php

    $serv = new swoole_http_server("0.0.0.0", 9502);
    $serv->set([
        'net_worker_num' => 1,
        'worker_num' => 1,
        'daemonize' => false,
        'log_file' => '/tmp/swoole_log.log',
    ]);

    $serv->on('Start', function() use($serv) {
        echo "Http server start!\n";
    });

    $serv->on('WorkerStart', function($serv, $worker_id){
        echo "Http server worker start, worker_id:" . $worker_id . "\n";
    });

    ############ 取 uri 进行不同分支处理
    $serv->on('Request', function($req, $resp) {
        $uri = $req->server['request_uri'];
        if (0 == strcmp($uri, "/getbaiduip")) {
            swoole_async_dns_lookup("www.baidu.com", function($host, $ip) use($resp) {
                echo "{$host} : {$ip}\n";
                $resp->status(200);
                $resp->end("{$host} : {$ip}");
            });
        } else {
            $resp->status(400);
            $resp->end("Error uri:$uri");
        }
    });

    $serv->start();

::

    在浏览器中输入 http://127.0.0.1:9502/getbaiduip 进行测试
    或在命令行输入 curl http://127.0.0.1:9502/getbaiduip 进行测试


