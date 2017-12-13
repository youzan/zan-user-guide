swoole_http_client 异步客户端
===================================

1. swoole_http_server 成员函数
    具体参考 API 文档：http://zandoc.zanphp.io/classes/swoole_http_client.html


2. 一个简单示例
::

    <?php

    $cli = new swoole_http_client('127.0.0.1', 9502);
    $cli->setHeaders([
        'Host' => 'localdomain',
        "User-Agent" => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $cli->setCookies(array('test' => 'value'));


    $cli->on("connect", function(\swoole_http_client $cli) {
        echo "http client: connect.\n";
    });

    $cli->on("error", function(\swoole_http_client $cli) {
        echo "http client: error.\n";
    });

    $cli->on("close", function(\swoole_http_client $cli) {
         echo "http client: close.\n";
    });

    $cli->get('/', function (swoole_http_client $cli) {
        echo $cli->body;
    });


3. 可以先通过 swoole_async_dns_lookup 查询域名得到 ip 地址，再构造 client
::

    <?php

    swoole_async_dns_lookup("www.baidu.com", function ($domainName, $ip) {
        $cli = new swoole_http_client($ip, 80);
        $cli->setHeaders([
            'Host' => $domainName,
            "User-Agent" => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->get('/index.html', function ($cli) {
            echo "Length: " . strlen($cli->body) . "\n";
            echo $cli->body;
        });
    });

4. 其它接口，如 setMethod、setCookies 可以参考 HttpServer 中的例子


