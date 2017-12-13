Master 进程对子进程的监控
=======================================

简介:
    master 进程是如何对子进程进行监控的

1. Master 进程中，函数 zan_master_process_loop 完成对所有子进程的监控；
2. 首先设置各个信号的处理函数；
3. 当全局变量 ServerG.running 大于 0 时，Master 阻塞于 wait 调用；
4. 当全局亦是 ServerG.running 不大于 0 时，Master 进程退出 while 循环，kill 各个进程并退出；
5. 当某一个子进程，worker 进程，Networker 进程，Taskworker 进程，Userworker 进程中的任意一个子进程意外退出、或被 kill
   wait 函数返回退出进程的 pid，Master 进程会重新 fork 该进程，并设置相关参数，再次阻塞于 wait 调用；


