# phalcon_swoole_crontab
在phalcon框架下用swoole编写的crontab

在 https://github.com/osgochina/swoole-crontab 基础上移植到phalcon框架中

###1.概述
<pre>
基于swoole的定时器程序，支持秒级处理.
异步多进程处理。
完全兼容crontab语法，且支持秒的配置,可使用数组规定好精确操作时间
请使用swoole扩展1.7.9-stable及以上版本.Swoole
支持worker处理redis队列任务
</pre>

###2.配置的支持

具体配置文件请看 config/crontab.php 介绍一下时间配置
<pre>
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59)[可省略，如果没有0位,则最小时间粒度是分钟]
</pre>

###3.帮助信息

<pre>
* Usage: /path/to/php main.php [options] -- [args...]

* -h [--help]        显示帮助信息
* -s start           启动进程
* -s stop            停止进程
* -s restart         重启进程
* -d [--daemon]      是否后台运行
* -r [--reload]      重新载入配置文件
* -m [--monitor]     监控进程是否在运行,如果在运行则不管,未运行则启动进程
* --worker           开启worker 可以针对redis队列读取并编写处理逻辑
* --checktime        默认精确对时(如果精确对时,程序则会延时到分钟开始0秒启动) 值为false则不精确对时
</pre>

###4.worker进程配置

在config/worker.php 中写入配置，并且启动的时候加上 --worker选项就能启动worker工作进程 配置如下:

<pre>
return array(
    //key是要加载的worker类名
    "ReadBook"=>array(
        "name"=>"队列1",            //备注名
        "processNum"=>1,           //启动的进程数量
        "redis"=>array(
            "queue"=>"abc"          // redis队列名
        )
    )
);

</pre>

具体的业务逻辑在/tasks/ 文件夹下。可以自己定义业务逻辑类，只需要继承AbstractWorker.php中的AbstractWorker类就可以

###5.例子

你可以在配置文件中加上以下配置:
<pre>
return array(
    'taskid1' =>
        array(
            'taskname' => 'php -i',  //任务名称
            'rule' => '* * * * * *',//定时规则,可以使用数组精确设置时间 如：array("22:18","2015-11-11 00:00:00 ","10:20:39")
            "unique" => 2, //排他数量，如果已经有这么多任务在执行，即使到了下一次执行时间，也不执行
            'execute'  => 'Cmd',//命令处理类
            'args' =>
                array(
                    'cmd'    => 'php -i',//命令
                    "ext": ""
                ),
        ),
);
</pre>

然后,执行

/path/to/php main.php -s start
执行完成以后你就可以在/tmp/test.log看到输出了，每秒输出一次

如果你需要写自己的代码逻辑，你也可以到tasks目录下，实现一个AbstractCrontab.php接口的类. 在其中写自己的逻辑代码。


