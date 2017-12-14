一个简单的 TCP Server 和 Client 示例
======================================

1. TCPServer 示例
::

    <?php
    $serv = new swoole_server("127.0.0.1", 9501, SWOOLE_PROCESS);
    $serv->set(array(
        'net_worker_num' => swoole_cpu_num(),
        'worker_num' => swoole_cpu_num(),
        'task_worker_num' => 1, 
        'daemonize' => false,
        'log_file' => '/tmp/swoole_log.log',
    ));

    $serv->on('start', function ($serv){
        echo "Server started, master_pid=" . $serv->master_pid .  "\n";
        echo "zan version: " . swoole_version() . "\n";
    });

    $serv->on('WorkerStart', function($serv, $worker_id){
        echo "onWorkerStart worker_id=$worker_id\n";
    });

    $serv->on('WorkerStop', function($serv, $worker_id){
        echo "WorkerStop worker_id=" . $worker_id . "\n";
    });

    $serv->on('connect', function ($serv, $fd){
        echo "Client Connect, fd=" . $fd . "\n";
    });

    $serv->on('receive', function ($serv, $fd, $from_id, $data) {
        echo "Receive data:" . $data . "\n";
        $serv->send($fd, 'Swoole: Hello Client!');

        $serv->task($data);
    });

    $serv->on('close', function ($serv, $fd) {
        echo "Client Close fd:$fd\n";
    });


    $serv->on('Task', function ($serv, $task_id, $src_worker_id, $data) {
        echo "onTask in: task_id:" . $task_id . ", src_worker_id:" . $src_worker_id . ",data:" . $data . "\n";
        
        $serv->finish(100);
    });

    $serv->on('Finish', function ($serv, $task_id, $data) {
        echo "onFinish in: task_id:" . $task_id . ",data:" . $data . "\n";
    });

    $serv->start();



2. TCPClient 示例
::
    <?php
    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
    //设置事件回调函数
    $client->on("connect", function($cli) {
        $serv = $cli->getpeername();
        var_dump($serv);

        $cli->send("Hello Server!\n");
    });
    $client->on("receive", function($cli, $data){
        echo "Received: ".$data."\n";
    });
    $client->on("error", function($cli){
        echo "Connect failed\n";
    });
    $client->on("close", function($cli){
        echo "Connection close\n";
    });
    //发起网络连接
    $client->connect('127.0.0.1', 9501, 0.5);


