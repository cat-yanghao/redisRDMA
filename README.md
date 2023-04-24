# 1.环境配置

## 1.1虚拟机配置

### 1.1.1虚拟机版本

[Ubuntu 20.04.2.0 desktop amd64](https://link.zhihu.com/?target=https%3A//releases.ubuntu.com/20.04/)

### 1.1.2确认当前内核是否支持RXE

```
cat /boot/config-$(uname -r) | grep RXE
```

![](C:\Users\86139\Desktop\毕设\源码\picture\1.png)



结果为m或者y代表有。

（大部分linux系统都已经编译了rdma内核，如果没有，需要重新编译内核，或者选择更新的版本）

### 1.1.3 安装用户态动态链接库

| 软件包名          | 主要功能                           |
| ----------------- | ---------------------------------- |
| libibverbs1       | ibverbs动态链接库                  |
| ibverbs-utils     | ibverbs示例程序                    |
| librdmacm1        | rdmacm动态链接库                   |
| libibumad3        | ibumad动态链接库                   |
| ibverbs-providers | ibverbs各厂商用户态驱动（包括RXE） |
| rdma-core         | 文档及用户态配置文件               |

我使用的Ubuntu版本已经安装有这几个包，如果没有，执行上诉命令

```bash
sudo apt-get install libibverbs1 ibverbs-utils librdmacm1 libibumad3 ibverbs-providers rdma-core
```

可以使用如下命令，查看

```bash
dpkg -L libibverbs1
```

### 1.1.4 安装其他工具（可不安装）

iproute2是用来替代net-tools软件包的，是一组开源的网络工具集合，比如用更强大ip命令替换了以前常用的ifconfig。我们需要其中的rdma工具来对RXE进行配置。一般的操作系统都已经包含了，安装也很简单：

```bash
sudo apt-get install iproute2
```

perftest是一个基于Verbs接口开发的开源RDMA性能测试工具，可以对支持RDMA技术的节点进行带宽和时延测试。相比于rdma-core自带的示例程序 ，功能更加强大，当然也更复杂。使用如下命令安装：

```bash
sudo apt-get install perftest
```

### 1.1.5 rdma配置

1.网卡设置仅主机模式（vmnet1）

2.配置RXE网卡

```bash
modprobe rdma_rxe
```

3.用户态配置（需要配置，ifconfig先去查看网卡名称（ens33））

sudo rdma link add master type rxe netdev ens33
sudo rdma link add slave1 type rxe netdev ens33

4.用rdma工具查看是否添加成功

```text
rdma link
ibv_devicesrxe
ibv_devinfo
```

### 1.1.6 rdma通信测试

master端：

```
ib_send_bw -d master
```

![image-20230424171432416](/picture/1.png)

slave1端

```
ib_send_bw -d slave1 <master ip>
```

![image-20230424171533730](/picture/2.png)

# 2运行

## 2.1运行redis
cd redis-4.0.11
make
cd src
./redis-server ../redis.conf

安装redis-desktop

```
apt istall redis-tools
redis-cli -v
apt update
snap install redis-desktop-manager
```



## 2.2master节点初始化redis数据

cd redis-init
make
./redis-init

## 2.3 redis tcp/ip主从传输
cd redis-4.0.11/src
./redis-cli -h 192.168.238.128 -p 6379
slaveof 192.168.238.129 6379

## 2.4 redis rdma传输

### 2.4.1运行主节点RDMA服务

cd rdma/server
make
./rdma-server

### 2.4.2运行slave节点RDMA服务

cd rdma/client
make
./rdma-client 192.168.238.129  12345

# 3.性能测试步骤
