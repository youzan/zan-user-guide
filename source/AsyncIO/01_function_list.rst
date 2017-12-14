AsyncIO 异步 IO 示例
=========================

1. AsyncIO 
    具体参考 API 文档：http://zandoc.zanphp.io/namespaces/default.html
    
    swoole_async_dns_lookup、swoole_async_read、swoole_async_write、swoole_async_set


2. 使用示例:
::

    <?php
    function recursiveWrite($dep = 0, $size = 1024 * 1024)
    {
        static $data;
        if ($data === null) {
            $data = file_get_contents("/dev/urandom", null, null, null, $size);
        }

        $file = "tmp.file";

        swoole_async_write($file, $data, -1, function ($file, $len) use(&$recursiveWrite, $dep, $size) {
            if ($dep > 100) {
                echo "SUCCESS";
                unlink($file);
                return false;
            }

            assert($len === $size);
            recursiveWrite(++$dep);
            return true;
        });
    }

    recursiveWrite();


::

    <?php
        swoole_async_read(__FILE__, function ($filename, $content) {
            $size = strlen($content);
            if ($size === 0) {
                echo "SUCCESS";
                return false;
            } else {
                assert(filesize(__FILE__) === $size);
                return true;
            }
        }, 8192);

::

    <?php
        swoole_async_dns_lookup("www.baidu.com", function($host, $ip) {
            echo "host:" . $host . " ip:" . $ip . "\n";
            echo "SUCCESS";
        });
