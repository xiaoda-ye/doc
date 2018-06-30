# Shadowsocks服务端搭建

####在 CentOS 系统中，`yum upgrade` 与 `yum update` 都会将系统包更新到最新版本。

不同的是，`upgrade` 会删除过时的包，而 `update` 则不会。

```shell
yum update
yum upgrade
```

#### 安装运行环境

```she
yum install epel-release
yum update
yum install git python-setuptools libsodium supervisor 
easy_install pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```



#### 编辑配置文件

```shell
vim /etc/shadowsocks.json
```

```json
{
    "server":"0.0.0.0",
    "server_port":8338,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb"
    "fast_open": false
}
```

#### 赋予shadowsocks配置文件权限

```shell
chmod 755 /etc/shadowsocks.json
```

#### 后台运行shadowsocks

```shell
ssserver -c /etc/shadowsocks.json -d start
```

#### 停止命令

```shell
ssserver -c /etc/shadowsocks.json -d stop
```

#### 设置shadowsocks开机自启动

```shell
vimv  /etc/rc.local
```

```shell
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
ssserver -c /etc/shadowsocks.json -d start
exit 0
```

#### 备注

最后还是连不上，应该是防火墙的问题，懂linux的人可以在配置里开放shadowsocks.json里配置的端口，不懂可以执行以下命令

```shell
service iptables stop
```

