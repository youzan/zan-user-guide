zan 框架简介及编译安装
==========================

1. 编译安装
::
    项目托管在 github 上： git@github.com:youzan/zan.git
    目前，最新的版本是 feature/refactor 分支，我们很快会将该分支合并到 master。

    编译：
        在我的 mac 笔记本上，编译安装命令为: $sudo sh build.sh "PHP的bin路径"
        $sudo sh build.sh /usr/local/Cellar/php71/7.1.5_17/bin

    验证:
        $php -m|grep zan
         zan

        还可以通过如下命令查看 zan 扩展默认配置
        $php -ini|grep zan

    注意：
        zan 扩展和 swoole 不兼容，如果你之前安装了 swoole，那么需要卸载 swoole 扩展；
        PHP 版本：推荐 7.1 以上版本

    配置php.ini:
        修改 php.ini
        extension=zan.so

2. 日志级别
::

    zan 框架默认日志级别为 WARNING
    我们可以通过修改环境变量的方式修改日志级别: export ZANEXT_DEBUG_LOG_LEVEL=1
    DEBUG   = 1
    TRUACE  = 2
    INFO    = 3
    NOTICE  = 4
    WARNING = 5
    ERROR   = 6

