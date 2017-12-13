swoole_http_server 底层 cookie 等属性操作
============================================

1. 下面的例子演示了 http_server 底层 cookie 等操作
::

    <?php
    $host = "127.0.0.1";
    $port = 9502;

    $httpServ = new \swoole_http_server($host, $port, SWOOLE_PROCESS, SWOOLE_SOCK_TCP);
        
    $httpServ->set([
        'user' => 'www-data',
        'group' => 'www-data',
        'log_file' => '/tmp/swoole.log',
        'worker_num' => 1,
        'net_worker_num' => 1,
        ]);

    $httpServ->on('WorkerStart', function (\swoole_http_server $httpServ, $workerId) {
        echo "onWorkerStart! worker_id=$workerId\n";
    });

    $httpServ->on('WorkerStop', function (\swoole_http_server $httpServ, $workerId) {
        echo "onWorkerStop! worker_id=$workerId\n";
    });

    $httpServ->on('connect', function ($httpServ, $fd) {
        echo "HttpServer: onConnected, client_fd=$fd\n";
    });

    $httpServ->on('receive', function (\swoole_http_server $swooleServer, $fd, $fromId, $data) {
        echo "HttpServer: onReceive: $data\n";
    });


    $httpServ->on('request', function (\swoole_http_request $request, \swoole_http_response $response) use($httpServ) {
        $uri = $request->server["request_uri"];
        if ($uri === "/favicon.ico")  {
            $response->status(404);
            $response->end();
            $httpServ->shutdown();
            return;
        }

        testSetCookie:
        {
            $name = "name";
            $value = "value";
            $expire = 0;
            $path = "/";
            $domain = "";
            $secure = false;
            $httpOnly = true;
            $response->cookie($name, $value, $expire, $path, $domain, $secure, $httpOnly);
            $expect = "name=value; path=/; httponly";
            assert(in_array($expect, $response->cookie, true));
        }

        if ($uri === "/ping")  {
            echo "onRequest/ping\n";
            $this->httpServ->send($request->fd, "HTTP/1.1 200 OK\r\nContent-Length: 4\r\n\r\npong\r\n");
        }

        if ($uri === "/gzip")  {
            echo "onRequest/gzip\n";
            $level = 9;
            $response->gzip($level);
            $response->end(RandStr::gen(1024 * 1024 * 2, RandStr::ALL));
        }

        if ($uri === "/info") {
            echo "onRequest/info\n";
            ob_start();
            print("request_uri: {$uri}\n");
            print("request_method: {$request->server['request_method']}\n");

            if (property_exists($request, "get")) {
                print("get:" . var_export($request->get, true) . "\n");
            }
            if (property_exists($request, "post")) {
                print("post:" . var_export($request->post, true) . "\n");
            }
            if (property_exists($request, "cookie")) {
                print("cookie:" . var_export($request->cookie, true) . "\n");
            }
            if (property_exists($request, "header")) {
                print("header:" . var_export($request->header, true) . "\n");
            }

            $response->end(nl2br(ob_get_clean()));
        }

        if ($uri === "/uri") {
           echo "onRequest/uri\n";
            $response->end($request->server['request_uri']);
        }

        if ($uri === "/method") {
           echo "onRequest/method\n";
            $response->end($request->server['request_method']);
        }

        if ($uri === "/get") {
           echo "onRequest/get\n";
            if (property_exists($request, "get")) {
                $response->end(json_encode($request->get));
            } else {
                $response->end("{}");
            }
        }

        if ($uri === "/post") {
           echo "onRequest/post\n";
            if (property_exists($request, "post")) {
                $response->end(json_encode($request->post));
            } else {
                $response->end("{}");
            }
        }

        if ($uri === "/cookie") {
           echo "onRequest/cookie\n";
            if (property_exists($request, "cookie")) {
                $response->end(json_encode($request->cookie));
            } else {
                $response->end("{}");
            }
        }

        if ($uri === "/header") {
           echo "onRequest/header\n";
            if (property_exists($request, "header")) {
                $response->end(json_encode($request->header));
            } else {
                $response->end("{}");
            }
        }

        if ($uri === "/sleep") {
           echo "onRequest/404\n";
            swoole_timer_after(1000, function() use($response) {
                $response->end();
            });
        }

        if ($uri === "/404") {
            echo "onRequest/404\n";
            $response->status(404);
            $response->end();
        }

        if ($uri === "/302") {
            echo "onRequest/302\n";
            $response->header("Location", "http://www.youzan.com/");
            $response->status(302);
            $response->end();
        }

        if ($uri === "/code") {
            echo "onRequest/code\n";
            swoole_async_readfile(__FILE__, function($filename, $contents) use($response) {
                $response->end(highlight_string($contents, true));
            });
        }

        if ($uri === "/json") {
            echo "onRequest/json\n";
            $response->header("Content-Type", "application/json");
            $response->end(json_encode($request->server, JSON_PRETTY_PRINT));
        }

        if ($uri === "/chunked") {
            echo "onRequest/chunked\n";
            $write = function($str) use($request) { return $this->httpServ->send($request->fd, $str); };

            $write("HTTP/1.1 200 OK\r\n");
            $write("Content-Encoding: chunked\r\n");
            $write("Transfer-Encoding: chunked\r\n");
            $write("Content-Type: text/html\r\n");
            $write("Connection: keep-alive\r\n");
            $write("\r\n");

            // "0\r\n\r\n" finish
            $writeChunk = function($str = "") use($write) {
                $hexLen = dechex(strlen($str));
                return $write("$hexLen\r\n$str\r\n");
            };
            $timer = swoole_timer_tick(200, function() use(&$timer, $writeChunk) {
                static $i = 0;
                $str = RandStr::gen($i++ % 40 + 1, RandStr::CHINESE) . "<br>";
                if ($writeChunk($str) === false) {
                    swoole_timer_clear($timer);
                }
            });
        }

        if ($uri === "/content_length") {
            echo "onRequest/content_length\n";
            if (property_exists($request, "header")) {
                if (isset($request->header['content-length'])) {
                    $response->end($request->header['content-length']);
                } else {
                    $response->end(0);
                }
            }
        }

        if ($uri === "/file") {
            echo "onRequest/file\n";
            $response->header("Content-Type", "text");
            $response->header("Content-Disposition", "attachment; filename=\"test.php\"");
            $response->sendfile(__FILE__);
        }

        if ($uri === "/rawcontent") {
            echo "onRequest/rawcontent\n";
            $response->rawcontent($request->rawcontent());
            $response->end("Hello World!");
        }

        if ($uri === "/rawcookie") {
            echo "onRequest/rawcookie\n";
            $response->cookie($name, $value, $expire, $path, $domain, $secure, $httpOnly);
            $response->rawcookie("rawcookie", $request->rawcontent());
            $response->end("Hello World!");
        }

        echo "onRequest end!\n";
    });

    $httpServ->start();

::

    <?php
    ########################测试代码示例
    $httpClient = new swoole_http_client('127.0.0.1', 9502);
    $httpClient->setHeaders(array('User-Agent' => 'swoole-http-client'));

    $httpClient->setMethod("POST");
    #########################test /rawcokkie
    $httpClient->setData("Hello Rawcookie!");
    $ok = $httpClient->execute("/rawcookie", function(\swoole_http_client $httpClient) {
        assert($httpClient->statusCode === 200);
        assert($httpClient->errCode === 0);
        echo "rawcookie: " . $httpClient->headers["set-cookie"]. "\n";
        echo "SUCCESS";
        return;
    });
    assert($ok == true);


::

    <?php
    ########################测试代码示例
    $httpClient = new swoole_http_client('127.0.0.1', 9502);
    $httpClient->setHeaders(array('User-Agent' => 'swoole-http-client'));

    $httpClient->setMethod("POST");
    #########################test /rawcontent
    $httpClient->setData("Hello Rawcontent!");
    $ok = $httpClient->execute("/rawcontent", function(\swoole_http_client $httpClient) {
        assert($httpClient->statusCode === 200);
        assert($httpClient->errCode === 0);

        echo "rawContend: " . $httpClient->body ."\n";
        echo "SUCCESS\n";
    });
    assert($ok == true);


::

    <?php
    ####################测试代码示例，sendfile
    $httpClient = new swoole_http_client('127.0.0.1', 9502);
    $httpClient->setHeaders(array('User-Agent' => 'swoole-http-client'));

    $httpClient->setMethod("GET");
    $ok = $httpClient->execute("/file", function(\swoole_http_client $httpClient) {
        assert($httpClient->statusCode === 200);
        assert($httpClient->errCode === 0);

        echo "fileContend: " . $httpClient->body . "\n";
        echo "SUCCESS\n";
    });
    assert($ok);
