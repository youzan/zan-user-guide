swoole_mysql 客户端使用示例
======================================

1. API 接口，详见：
    http://zandoc.zanphp.io/classes/swoole_mysql.html


2. 下例演示 mysql client 的简单用法
::

    <?php

    function swoole_mysql_query($sql, callable $onQuery)
    {
        $swoole_mysql = new \swoole_mysql();

        $swoole_mysql->on("close", function() {
            // echo "closed\n";
        });

        $swoole_mysql->conn_timeout = swoole_timer_after(1000, function() use($onQuery, $swoole_mysql) {
            $onQuery($swoole_mysql, "connecte timeout");
        });

        $swoole_mysql->connect([
            "host" => "127.0.0.1",
            "port" => 3306,
            "user" => "root",
            "password" => "123456",
            "database" => "test",      ############################数据库名
            "charset" => "utf8mb4",    
        ], function(\swoole_mysql $swoole_mysql, $result) use($sql, $onQuery) {
            swoole_timer_clear($swoole_mysql->conn_timeout);

            if ($result) {
                $swoole_mysql->query_timeout = swoole_timer_after(1000, function() use($onQuery, $swoole_mysql) {
                    $onQuery($swoole_mysql, "query timeout");
                });

                $swoole_mysql->query($sql, function(\swoole_mysql $swoole_mysql, $result) use($onQuery) {
                    swoole_timer_clear($swoole_mysql->query_timeout);
                    $onQuery($swoole_mysql, $result);
                });
            } else {
                echo "connect error [errno=$swoole_mysql->connect_errno, error=$swoole_mysql->connect_error]";
            }
        });
    }

    ######################往数据库 test 的表 mytable 中插入一条数据；
    $sql = "insert into mytable1 (`name`, `sex`, `birth`, `birthaddr`) VALUES ('zhang', 'f', '1991-09-04', 'china')";

    #insert
    swoole_mysql_query($sql, function($swoole_mysql, $result) {
        ob_start();
        echo "result: " . $result . "\n";
        
        assert($result === true);
        assert($swoole_mysql->errno === 0);
        if ($buf = ob_get_clean()) {
            fprintf(STDERR, $buf);
        }
        swoole_event_exit();
        fprintf(STDERR, "Suucess!");
    });


::

    <?php
    #############################演示基本的 prepare 用法
    $swoole_mysql = new \swoole_mysql();

    $swoole_mysql->on("close", function() {
        echo "closed\n";
    });

    $swoole_mysql->conn_timeout = swoole_timer_after(3000, function() {
        assert(false, "connect timeout");
    });

    $swoole_mysql->connect([
        "host" => "127.0.0.1",
        "port" => 3306,
        "user" => "root",
        "password" => "123456",
        "database" => "test",         ############数据库名
        "charset" => "utf8mb4",
    ], function(\swoole_mysql $swoole_mysql, $result) {
        swoole_timer_clear($swoole_mysql->conn_timeout);

        if ($result) {
            $swoole_mysql->query_timeout = swoole_timer_after(2000, function() {
                echo "Error: query timeout, exit.\n";
                exit(1);
            });

            ##############表 mytable1
            ##############在数据库 test 的 mytable1 表中，查询 name="zhang" AND sex="f" 的条数
            $sql = "SELECT COUNT(1) AS cnt FROM mytable1 WHERE name = :name AND sex = :sex";
            $arr = ["name" => "zhang", "sex" => 'f'];
            $swoole_mysql->safe_query($sql, $arr, function(\swoole_mysql $swoole_mysql, $result) {
                echo "result:" . json_encode($result) . "\n";
                echo "errno: " . $swoole_mysql->errno . "\n";

                swoole_timer_clear($swoole_mysql->query_timeout);
                fprintf(STDERR, "SUCCESS");
                swoole_event_exit();
            });
        } else {
            echo "connect error [errno=$swoole_mysql->connect_errno, error=$swoole_mysql->connect_error]";
        }
    });