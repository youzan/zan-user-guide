Timer 定时器使用示例
=========================


1. Timer 
    具体参考 API 文档：http://zandoc.zanphp.io/namespaces/default.html
    
    swoole_timer_tick、swoole_timer_after、swoole_timer_clear、swoole_timer_exists、swoole_timer_set


2. 使用示例:
::

    <?php
    #如下示例演示了 swoole_timer_after、swoole_timer_clear、swoole_timer_exists 的用法
    class TimerTest {
        public static $count = 0;
        private $timer_id = null;

        protected function resetTimer($ms) {
            if ($this->timer_id && swoole_timer_exists($this->timer_id)) {
                swoole_timer_clear($this->timer_id);
                $this->timer_id = null;
            }
            if (self::$count == 10) {
                return;
            }
            $this->timer_id = swoole_timer_after($ms, array($this, 'onTimerTick'));
            assert($this->timer_id > 0);
        }

        public function onTimerTick() {
            self::$count++;
            echo "onTimerTick:" . self::$count . "\n";
            $this->resetTimer(10);
        }
    }

    $timer_test = new TimerTest();
    $timer_test->onTimerTick();


::

    <?php
    #如下示例是我们在使用 swoole 写服务端代码时经常用到的
    #我们在第 1 个 worker 进程是启动一个定时器，定时查询 www.baidu.com 的 ip 地址；

    $serv = new swoole_server("127.0.0.1", 9501, SWOOLE_PROCESS);
    $serv->set(array(
        'net_worker_num' => 2,
        'worker_num' => 2,
        'daemonize' => false,
        'log_file' => '/tmp/swoole_log.log',
    ));

    $serv->on('start', function ($serv){
        echo "Server started, master_pid:" . $serv->master_pid .  "\n";
    });

    $serv->on('WorkerStart', function($serv, $worker_id){
        echo "WorkerStart worker_id: " . $worker_id . "\n";

        if ($worker_id == 0) {
            $timerId = swoole_timer_tick(5000, function() use ($serv) {
                echo "Timer tick....\n";
                swoole_async_dns_lookup("www.baidu.com", function($host, $ip){
                    echo "baidu: {$host} : {$ip}\n";
                });
            });
        }
    });

    $serv->on('WorkerStop', function($serv, $worker_id){
        echo "WorkerStop worker_id:" . $worker_id . "\n";
    });

    $serv->on('connect', function ($serv, $fd){
        echo "Client Connect, fd: " . $fd . "\n";
        $serv->send($fd, 'Swoole: Hello Connect!');
    });

    $serv->on('receive', function ($serv, $fd, $from_id, $data) {
        echo "Receive data:" . $data . "\n";
        $serv->send($fd, 'Swoole: Hello Receive!');
    });

    $serv->on('close', function ($serv, $fd) {
        echo "Client Close fd: " . $fd . "\n";
    });

    $serv->start();


