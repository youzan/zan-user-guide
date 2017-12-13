swoole_redis 使用示例
=========================

1. swoole_redis 成员函数
    具体参考 API 文档：http://zandoc.zanphp.io/classes/swoole_redis.html


2. 简单代码示例如下：
::

    <?php

        $redis = new swoole_redis;
        $redis->on('Close', function (swoole_redis $redis) {
            echo "onClose\n";
        });

        $redis->on('Message', function (swoole_redis $redis, array $message) {
            echo "onMessage:" . json_encode($message) . "\n";
        });

        $redis->connect('127.0.0.1', 6379, function ($redis, $result) {
            $redis->set('test_key', 'value', function ($redis, $result) {
                $redis->get('test_key', function ($redis, $result) {
                    echo "value=$result". "\n";
                    $redis->close();
                    return;
                });
            });
        });
