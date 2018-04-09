# 源码安装

源码安装Nginx的过程也很简单，下面以Debian 9为例讲述如何源码安装Nginx。

## 安装依赖

Nginx提供了丰富的module，这包括既有的一些官方核心模块，也有第三方提供的种类繁多的功能模块，而模块自身的依赖可能需要单独安装。
最常见的莫过于http gzip所需的zlib、rewrite指令需要的pcre支持以及ssl指令所需的openssl支持，我们可以通过如下命令来安装这些依赖：

```
apt-get install -y gcc make libpcre3-dev libpcre3 openssl libssl1.0-dev libssl1.0.2 perl libperl-dev zlib1g-dev zlib1g
```

## 编译参数

和其他C/C++开发的软件类似，Nginx同样是通过configure脚本来完成一些预编译参数的生成，它在幕后做了大量的工作，包括检测操作系统平台版本，确认已安装的模块依赖，以及一些参数和中间目录还有C源码（主要是宏）、Makefile的生成，这里我们只需简单执行configure即可：

```
./configure
```

## make

在成功执行configure脚本后，我们可以发现目录下多出一个Makefile，接下来便是使用该Makefile来做build或install了！我们不妨看一下生成出来的Makefile里面的内容：

```
# cat Makefile

default:    build

clean:
    rm -rf Makefile objs

build:
    $(MAKE) -f objs/Makefile

install:
    $(MAKE) -f objs/Makefile install

modules:
    $(MAKE) -f objs/Makefile modules

upgrade:
    /usr/local/nginx/sbin/nginx -t

    kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
    sleep 1
    test -f /usr/local/nginx/logs/nginx.pid.oldbin

    kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`
```

这里有几个值得关注的点：

1. 该Makefile只是一个全局性质的入口文件，build和install背后还是通过`objs/Makefile`这个文件来实现真正的构建和安装逻辑；

2. `modules`选项提供的是针对模块本身的编译，这可以用来做dynamic module或模块本身的调试，详见[官方文档](https://www.nginx.com/resources/wiki/extending/converting/#compiling-dynamic)；

3. `upgrade`选项提供的是nginx自身版本的升级，这也适用于用deb安装的情况，不难发现，升级时可以通过发送`-USR2`信号给正在运行的nginx进程，nginx的master进程将会把现有的PID文件重命名为`nginx.pid.oldbin`，然后尝试新启动一个nginx服务（这时候假设已经更新了nginx的可执行文件且启动路径未变，因此启动的便是新版本的服务），并且可以通过`kill -WINCH $(cat /usr/local/nginx/logs/nginx.pid.oldbin)`、`kill -QUIT $(cat /usr/local/nginx/logs/nginx.pid.oldbin)`的方式平滑停止老版本nginx服务，这即是nginx官方提供的[平滑升级方案](http://nginx.org/en/docs/control.html#upgrade)。

回过头来，上文也提到build/install背后是`objs/Makefile`，简单列出`objs`下的目录结构：

```
# tree ./objs
.
├── autoconf.err
├── Makefile
├── ngx_auto_config.h
├── ngx_auto_headers.h
├── ngx_modules.c
└── src
    ├── core
    ├── event
    │   └── modules
    ├── http
    │   ├── modules
    │   │   └── perl
    │   └── v2
    ├── mail
    ├── misc
    ├── os
    │   ├── unix
    │   └── win32
    └── stream
```

之前configure脚本配置生成的模块列表的定义放在`ngx_modules.c`文件里，所需的依赖也放到了`src`目录下，关于Makefile具体详细的内容，我们后面在自定义模块的部分再来详细讨论，现在我们只需要执行`make install`即可。

```
# make install
# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.13.8
built by gcc 6.3.0 20170516 (Debian 6.3.0-18+deb9u1)
configure arguments:
```

Enjoy!
