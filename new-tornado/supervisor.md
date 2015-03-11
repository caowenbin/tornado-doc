# 使用supervisor管理web程序

Supervisor是一个客户/服务器系统，它可以在类Unix系统中管理控制大量进程。Supervisor使用python开发，有多年历史，目前很多生产环境下的服务器都在使用Supervisor。

Supervisor的服务器端称为supervisord，主要负责在启动自身时启动管理的子进程，响应客户端的命令，重启崩溃或退出的子进程，记录子进程stdout和stderr输出，生成和处理子进程生命周期中的事件。可以在一个配置文件中配置相关参数，包括Supervisord自身的状态，其管理的各个子进程的相关属性。配置文件一般位于/etc/supervisord.conf。

Supervisor的客户端称为supervisorctl，它提供了一个类shell的接口（即命令行）来使用supervisord服务端提供的功能。通过supervisorctl，用户可以连接到supervisord服务器进程，获得服务器进程控制的子进程的状态，启动和停止子进程，获得正在运行的进程列表。客户端通过Unix域套接字或者TCP套接字与服务端进行通信，服务器端具有身份凭证认证机制，可以有效提升安全性。当客户端和服务器位于同一台机器上时，客户端与服务器共用同一个配置文件/etc/supervisord.conf，通过不同标签来区分两者的配置。

**注意:** 平台要求

    * Supervisor可以运行在大多数Unix系统上，但不支持在Windows系统上运行。
    * Supervisor需要Python2.4及以上版本，但任何Python 3版本都不支持。

可以通过PyPI或者系统安装软件的方法来安装

```
sudo pip install supervisor
```

生成默认配置文件

```
echo_supervisord_conf > /etc/supervisord.conf
```

通常来说Tornado进程都会跑好几个，几个依赖于系统负载，最理想的负载是60%（CPU和内存使用）。运行多个实例是为了充分使用现代计算机的多处理器和多核能力。

这里给出我们在线环境当中使用的supervisord配置文件（仅列出Tornado相关部分）。

```
[program:tornadowebs]
command=python /opt/tornadoweb/manage.py --port=60%(process_num)02d
process_name=%(program_name)s_%(process_num)02d
umask=022
startsecs=0
stopwaitsecs=0
redirect_stderr=true
stdout_logfile=/var/log/is_web.log
numprocs=4
numprocs_start=1
```