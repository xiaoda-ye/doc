# Nginx配置详解

## 1、Nginx配置语法

####块配置项

快配置项均由一个块配置项名和一对大括号组成，具体示例如下

```c++
events {
	…
}
http {
    upstream backend {
    	server 127.0.0.1:8080;
	}
    gzip on;
	server {
        …
        location /webstatic {
        	gzip off;
        }
    }
}
```

####配置项注释

```
#pid logs/nginx.pid;
```

####在配置中使用变量

```
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
'$status $bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"';
```

其中，remote_addr 是一个变量，使用它的时候前面要加上 $ 符号。需要注意的是，这
种变量只有少数模块支持，并不是通用的。

##2、用于调试进程和定位问题的配置项

#### （1）是否以守护进程方式运行 Nginx

语法：daemon on | off;
默认：daemon on;
守护进程（daemon）是脱离终端并且在后台运行的进程。它脱离终端是为了避免进程
执行过程中的信息在任何终端上显示，这样一来，进程也不会被任何终端所产生的信息所打断。Nginx 毫无疑问是一个需要以守护进程方式运行的服务，因此，默认都是以这种方式运行的。
不过 Nginx 还是提供了关闭守护进程的模式，之所以提供这种模式，是为了方便跟踪调
试 Nginx，毕竟用 gdb 调试进程时最烦琐的就是如何继续跟进 fork 出的子进程了。这在第三部分研究 Nginx 架构时很有用。

####（2）是否以 master/worker 方式工作

语法：master_process on | off;
默认：master_process on;
可以看到，在如图 2-1 所示的产品环境中，是以一个 master 进程管理多个 worker 进程
的方式运行的，几乎所有的产品环境下，Nginx 都以这种方式工作。
与 daemon 配置相同，提供 master_process 配置也是为了方便跟踪调试 Nginx。如果用
off 关闭了 master_process 方式，就不会 fork 出 worker 子进程来处理请求，而是用 master 进程自身来处理请求。

####（3）error 日志的设置

语法：error_log /path/file level;
默认：error_log logs/error.log error;
error 日志是定位 Nginx 问题的最佳工具，我们可以根据自己的需求妥善设置 error 日志
的路径和级别。
/path/file 参数可以是一个具体的文件，例如，默认情况下是 logs/error.log 文件，最好将
它放到一个磁盘空间足够大的位置 ；/path/file 也可以是 /dev/null，这样就不会输出任何日志了，这也是关闭 error 日志的唯一手段 ；/path/file 也可以是 stderr，这样日志会输出到标准错误文件中。
level 是日志的输出级别，取值范围是 debug、info、notice、warn、error、crit、alert、
emerg，从左至右级别依次增大。当设定为一个级别时，大于或等于该级别的日志都会被输出到 /path/file 文件中，小于该级别的日志则不会输出。例如，当设定为 error 级别时，error、crit、alert、emerg 级别的日志都会输出。
如果设定的日志级别是 debug，则会输出所有的日志，这样数据量会很大，需要预先确
保 /path/file 所在磁盘有足够的磁盘空间。

```
注意 如果日志级别设定到 debug，必须在 configure 时加入 --with-debug 配置项。
```

##3、正常运行的配置项

下面是正常运行的配置项的相关介绍。

####（1）定义环境变量

语法：env VAR|VAR=VALUE
这个配置项可以让用户直接设置操作系统上的环境变量。例如：
env TESTPATH=/tmp/;

####（2）嵌入其他配置文件

语法：include /path/file;
include 配置项可以将其他配置文件嵌入到当前的 nginx.conf 文件中，它的参数既可以是绝
对路径，也可以是相对路径（相对于 Nginx 的配置目录，即 nginx.conf 所在的目录），例如：
include mime.types;
include vhost/*.conf;
可以看到，参数的值可以是一个明确的文件名，也可以是含有通配符 * 的文件名，同时
可以一次嵌入多个配置文件。

####（3）pid 文件的路径

语法：pid path/file;
默认：pid logs/nginx.pid;
保存 master 进程 ID 的 pid 文件存放路径。默认与 configure 执行时的参数“--pid-path”
所指定的路径是相同的，也可以随时修改，但应确保 Nginx 有权在相应的目标中创建 pid 文件，该文件直接影响 Nginx 是否可以运行。

####（4）Nginx worker 进程运行的用户及用户组

语法：user username [groupname];
默认：user nobody nobody;
user 用于设置 master 进程启动后，fork 出的 worker 进程运行在哪个用户和用户组下。
当按照“user username;”设置时，用户组名与用户名相同。
若用户在 configure 命令执行时使用了参数 --user=username 和 --group=groupname，此时 nginx.conf 将使用参数中指定的用户和用户组。

####（5）指定 Nginx worker 进程可以打开的最大句柄描述符个数

语法：worker_rlimit_nofile limit;
设置一个 worker 进程可以打开的最大文件句柄数。

####（6）限制信号队列

语法：worker_rlimit_sigpending limit;
设置每个用户发往 Nginx 的信号队列的大小。也就是说，当某个用户的信号队列满了，
这个用户再发送的信号量会被丢掉。

##4、优化性能的配置项

下面是优化性能的配置项的相关介绍。

####（1）Nginx worker 进程个数

语法：worker_processes number;
默认：worker_processes 1;
在 master/worker 运行方式下，定义 worker 进程的个数。
worker 进程的数量会直接影响性能。那么，用户配置多少个 worker 进程才好呢？这实
际上与业务需求有关。
每个 worker 进程都是单线程的进程，它们会调用各个模块以实现多种多样的功能。如
果这些模块确认不会出现阻塞式的调用，那么，有多少 CPU 内核就应该配置多少个进程；反之，如果有可能出现阻塞式调用，那么需要配置稍多一些的 worker 进程。
例如，如果业务方面会致使用户请求大量读取本地磁盘上的静态资源文件，而且服务器
上的内存较小，以至于大部分的请求访问静态资源文件时都必须读取磁盘（磁头的寻址是缓慢的），而不是内存中的磁盘缓存，那么磁盘 I/O 调用可能会阻塞住 worker 进程少量时间，进而导致服务整体性能下降。
多 worker 进程可以充分利用多核系统架构，但若 worker 进程的数量多于 CPU 内核数，
那么会增大进程间切换带来的消耗（Linux 是抢占式内核）。一般情况下，用户要配置与 CPU内核数相等的 worker 进程，并且使用下面的 worker_cpu_affinity 配置来绑定 CPU 内核。

####（2）绑定 Nginx worker 进程到指定的 CPU 内核

语法：worker_cpu_affinity cpumask [cpumask...]
为什么要绑定 worker 进程到指定的 CPU 内核呢？假定每一个 worker 进程都是非常繁忙
的，如果多个 worker 进程都在抢同一个 CPU，那么这就会出现同步问题。反之，如果每一个 worker 进程都独享一个 CPU，就在内核的调度策略上实现了完全的并发。
例如，如果有 4 颗 CPU 内核，就可以进行如下配置：
worker_processes 4;
worker_cpu_affinity 1000 0100 0010 0001;
注意 worker_cpu_affinity 配置仅对 Linux 操作系统有效。Linux 操作系统使用 sched_
setaffinity() 系统调用实现这个功能。

####（3）SSL 硬件加速

语法：ssl_engine device；
如果服务器上有 SSL 硬件加速设备，那么就可以进行配置以加快 SSL 协议的处理速度。
用户可以使用 OpenSSL 提供的命令来查看是否有 SSL 硬件加速设备：

```
openssl engine -t
```



####（4）系统调用 gettimeofday 的执行频率

语法：timer_resolution t;
默认情况下，每次内核的事件调用（如 epoll、select、poll、kqueue 等）返回时，都会
执行一次 gettimeofday，实现用内核的时钟来更新 Nginx 中的缓存时钟。在早期的 Linux 内核中，gettimeofday 的执行代价不小，因为中间有一次内核态到用户态的内存复制。当需要降低 gettimeofday 的调用频率时，可以使用 timer_resolution 配置。例如，“timer_resolution100ms；”表示至少每 100ms 才调用一次 gettimeofday。
但在目前的大多数内核中，如 x86-64 体系架构，gettimeofday 只是一次 vsyscall，仅仅
对共享内存页中的数据做访问，并不是通常的系统调用，代价并不大，一般不必使用这个配置。而且，如果希望日志文件中每行打印的时间更准确，也可以使用它。

####（5）Nginx worker 进程优先级设置

语法：worker_priority nice;
默认：worker_priority 0;
该配置项用于设置 Nginx worker 进程的 nice 优先级。
在 Linux 或其他类 UNIX 操作系统中，当许多进程都处于可执行状态时，将按照所有进
程的优先级来决定本次内核选择哪一个进程执行。进程所分配的 CPU 时间片大小也与进程
优先级相关，优先级越高，进程分配到的时间片也就越大（例如，在默认配置下，最小的时间片只有 5ms，最大的时间片则有 800ms）。这样，优先级高的进程会占有更多的系统资源。
优先级由静态优先级和内核根据进程执行情况所做的动态调整（目前只有 ±5 的调整）
共同决定。nice 值是进程的静态优先级，它的取值范围是 –20 ～ +19，–20 是最高优先级，+19 是最低优先级。因此，如果用户希望 Nginx 占有更多的系统资源，那么可以把 nice 值配置得更小一些，但不建议比内核进程的 nice 值（通常为 –5）还要小。

## 5、事件类配置项

下面是事件类配置项的相关介绍。

####（1）是否打开 accept 锁

语法：accept_mutex [on | off]
默认：accept_mutext on;
accept_mutex 是 Nginx 的负载均衡锁，本书会在第 9 章事件处理框架中详述 Nginx 是
如何实现负载均衡的。这里，读者仅需要知道 accept_mutex 这把锁可以让多个 worker 进程
轮流地、序列化地与新的客户端建立 TCP 连接。当某一个 worker 进程建立的连接数量达
到 worker_connections 配置的最大连接数的 7/8 时，会大大地减小该 worker 进程试图建立新
TCP 连接的机会，以此实现所有 worker 进程之上处理的客户端请求数尽量接近。

accept 锁默认是打开的，如果关闭它，那么建立 TCP 连接的耗时会更短，但 worker 进
程之间的负载会非常不均衡，因此不建议关闭它。

####（2）lock 文件的路径

语法：lock_file path/file;
默认：lock_file logs/nginx.lock;
accept 锁可能需要这个 lock 文件，如果 accept 锁关闭，lock_file 配置完全不生效。如果
打开了 accept 锁，并且由于编译程序、操作系统架构等因素导致 Nginx 不支持原子锁，这时才会用文件锁实现 accept 锁（14.8.1 节将会介绍文件锁的用法），这样 lock_file 指定的 lock文件才会生效。
注意 在基于 i386、AMD64、Sparc64、PPC64 体系架构的操作系统上，若使用 GCC、
Intel C++ 、SunPro C++ 编译器来编译 Nginx，则可以肯定这时的 Nginx 是支持原子锁的，因为 Nginx 会利用 CPU 的特性并用汇编语言来实现它（可以参考 14.3 节 x86 架构下原子操作的实现）。这时的 lock_file 配置是没有意义的。

####（3）使用 accept 锁后到真正建立连接之间的延迟时间

语法：accept_mutex_delay Nms;
默认：accept_mutex_delay 500ms;
在使用 accept 锁后，同一时间只有一个 worker 进程能够取到 accept 锁。这个 accept 锁
不是阻塞锁，如果取不到会立刻返回。如果有一个 worker 进程试图取 accept 锁而没有取到，它至少要等 accept_mutex_delay 定义的时间间隔后才能再次试图取锁。

####（4）批量建立新连接

语法：multi_accept [ on | off ];
默认：multi_accept off;
当事件模型通知有新连接时，尽可能地对本次调度中客户端发起的所有 TCP 请求都建
立连接。

####（5）选择事件模型

语法：use [ kqueue | rtsig | epoll | /dev/poll | select | poll | eventport ];
默认：Nginx 会自动使用最适合的事件模型。
对于 Linux 操作系统来说，可供选择的事件驱动模型有 poll、select、epoll 三种。epoll
当然是性能最高的一种，在 9.6 节会解释 epoll 为什么可以处理大并发连接。

####（6）每个 worker 的最大连接数

语法：worker_connections number;
定义每个 worker 进程可以同时处理的最大连接数。

## 用 HTTP 核心模块配置一个静态 Web 服务器

静态 Web 服务器的主要功能由 ngx_http_core_module 模块（HTTP 框架的主要成员）实
现，当然，一个完整的静态 Web 服务器还有许多功能是由其他的 HTTP 模块实现的。本节
主要讨论如何配置一个包含基本功能的静态 Web 服务器，文中会完整地说明 ngx_http_core_module 模块提供的配置项及变量的用法，但不会过多说明其他 HTTP 模块的配置项。在阅读完本节内容后，读者应当可以通过简单的查询相关模块（如 ngx_http_gzip_filter_module、ngx_http_image_filter_module 等）的配置项说明，方便地在 nginx.conf 配置文件中加入新的配置项，从而实现更多的 Web 服务器功能。
除了 2.3 节提到的基本配置项外，一个典型的静态 Web 服务器还会包含多个 server 块和
location 块，例如：

```
http {
    gzip on;
    upstream {
    	…
    }
    …
    server {
    	listen localhost:80;
    	…
    	location /webstatic {
            if … {
                …
            }
            root /opt/webresource;
            …
    	}
        location ~* .(jpg|jpeg|png|jpe|gif)$ {
        	…
        }
    }
    server {
    	…
    }
}

```

所有的 HTTP 配置项都必须直属于 http 块、server 块、location 块、upstream 块或 if 块
等（HTTP 配置项自然必须全部在 http{} 块之内，这里的“直属于”是指配置项直接所属的大括号对应的配置块），同时，在描述每个配置项的功能时，会说明它可以在上述的哪个块中存在，因为有些配置项可以任意地出现在某一个块中，而有些配置项只能出现在特定的块中，在第 4 章介绍自定义配置项的读取时，相信读者就会体会到这种设计思路。
Nginx 为配置一个完整的静态 Web 服务器提供了非常多的功能，下面会把这些配置项分为以下 8 类进行详述 ：虚拟主机与请求的分发、文件路径的定义、内存及磁盘资源的分配、网络连接的设置、MIME 类型的设置、对客户端请求的限制、文件操作的优化、对客户端请求的特殊处理。这种划分只是为了帮助大家从功能上理解这些配置项。
在这之后会列出 ngx_http_core_module 模块提供的变量，以及简单说明它们的意义。

## 虚拟主机与请求的分发

