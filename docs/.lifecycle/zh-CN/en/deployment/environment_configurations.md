---
displayed_sidebar: "Chinese"
---

# 检查环境配置

本主题列出了在部署StarRocks之前必须检查和设置的所有环境和系统配置项。正确设置这些配置项可以让您的StarRocks集群具有高可用性和性能。

## 端口

StarRocks为不同的服务使用特定的端口。如果您在实例上部署了其他服务，需要检查每个实例上这些端口是否被占用。

### FE端口

在用于FE部署的实例上，您需要检查以下端口：

- `8030`：FE HTTP服务器端口（`http_port`）
- `9020`：FE Thrift服务器端口（`rpc_port`）
- `9030`：FE MySQL服务器端口（`query_port`）
- `9010`：FE内部通信端口（`edit_log_port`）

在FE实例上运行以下命令以检查这些端口是否被占用：

```Bash
netstat -tunlp | grep 8030
netstat -tunlp | grep 9020
netstat -tunlp | grep 9030
netstat -tunlp | grep 9010
```

如果以上任何端口被占用，您必须寻找替代方案，并在部署FE节点时稍后指定它们。有关详细说明，请参见[部署StarRocks - 启动Leader FE节点](../deployment/deploy_manually.md#step-1-start-the-leader-fe-node)。

### BE端口

在用于BE部署的实例上，您需要检查以下端口：

- `9060`：BE Thrift服务器端口（`be_port`）
- `8040`：BE HTTP服务器端口（`be_http_port`）
- `9050`：BE心跳服务端口（`heartbeat_service_port`）
- `8060`：BE bRPC端口（`brpc_port`）

在BE实例上运行以下命令以检查这些端口是否被占用：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
```

如果以上任何端口被占用，您必须寻找替代方案，并在部署BE节点时稍后指定它们。有关详细说明，请参见[部署StarRocks - 启动BE服务](../deployment/deploy_manually.md#step-2-start-the-be-service)。

### CN端口

在用于CN部署的实例上，您需要检查以下端口：

- `9060`：CN Thrift服务器端口（`be_port`）
- `8040`：CN HTTP服务器端口（`be_http_port`）
- `9050`：CN心跳服务端口（`heartbeat_service_port`）
- `8060`：CN bRPC端口（`brpc_port`）
- `9070`：CN的额外代理服务端口（在共享数据集群中是BE v3.0）（`starlet_port`）

在CN实例上运行以下命令以检查这些端口是否被占用：

```Bash
netstat -tunlp | grep 9060
netstat -tunlp | grep 8040
netstat -tunlp | grep 9050
netstat -tunlp | grep 8060
netstat -tunlp | grep 9070
```

如果以上任何端口被占用，您必须寻找替代方案，并在部署CN节点时稍后指定它们。有关详细说明，请参见[部署StarRocks - 启动CN服务](../deployment/deploy_manually.md#step-3-optional-start-the-cn-service)。

## 主机名

如果您希望为StarRocks集群[启用FQDN访问](../administration/enable_fqdn.md)，您必须为每个实例分配主机名。

在每个实例的文件**/etc/hosts**中，您必须指定集群中所有其他实例的IP地址和相应的主机名。

> **注意**
>
> 文件**/etc/hosts**中的所有IP地址必须是唯一的。

## JDK配置

StarRocks依赖于环境变量`JAVA_HOME`来定位实例上的Java依赖项。

运行以下命令以检查环境变量`JAVA_HOME`：

```Bash
echo $JAVA_HOME
```

按照以下步骤设置`JAVA_HOME`：

1. 在文件**/etc/profile**中设置`JAVA_HOME`：

   ```Bash
   sudo vi /etc/profile
   # 用JDK安装路径替换<path_to_JDK>。
   export JAVA_HOME=<path_to_JDK>
   export PATH=$PATH:$JAVA_HOME/bin
   ```

2. 生效更改：

   ```Bash
   source /etc/profile
   ```

运行以下命令以验证更改：

```Bash
java -version
```

## CPU性能调节器

此配置项是**可选**的。如果CPU不支持性能调节器，则可以跳过此项。

CPU性能调节器控制CPU的电源模式。如果您的CPU支持，我们建议您将其设置为`performance`以获得更好的CPU性能：

```Bash
echo 'performance' | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

## 内存配置

### 内存超额分配

内存超额分配允许操作系统向进程超额分配内存资源。我们建议您启用内存超额分配。

```Bash
echo 1 | sudo tee /proc/sys/vm/overcommit_memory
```

### 透明大页

透明大页默认启用。我们建议您禁用此功能，因为它可能会干扰内存分配器，从而导致性能下降。

```Bash
echo 'madvise' | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
```

### 交换空间

我们建议您禁用交换空间。

按照以下步骤检查并禁用交换空间：

1. 禁用交换空间。

   ```SQL
   swapoff /<path_to_swap_space>
   ```

2. 从配置文件**/etc/fstab**中删除交换空间信息。

   ```Bash
   /<path_to_swap_space> swap swap defaults 0 0
   ```

3. 验证交换空间是否已禁用。

   ```Bash
   free -m
   ```

### Swappiness

我们建议您禁用swappiness以消除其对性能的影响。

```Bash
echo 0 | sudo tee /proc/sys/vm/swappiness
```

## 存储配置

我们建议您根据所使用的存储介质选择适合的调度程序算法。

您可以运行以下命令检查自己正在使用的调度程序算法：

```Bash
cat /sys/block/${disk}/queue/scheduler
# 例如，运行 cat /sys/block/vdb/queue/scheduler
```

我们建议您对SATA磁盘使用mq-deadline调度程序算法，并对SSD和NVMe磁盘使用kyber调度程序算法。

### SATA

mq-deadline调度程序算法适用于SATA磁盘。

暂时修改此项：

```Bash
echo mq-deadline | sudo tee /sys/block/${disk}/queue/scheduler
```

在修改此项后，运行以下命令使更改永久生效：

```Bash
chmod +x /etc/rc.d/rc.local
```

### SSD和NVMe

kyber调度程序算法适用于NVMe或SSD磁盘。

暂时修改此项：

```Bash
echo kyber | sudo tee /sys/block/${disk}/queue/scheduler
```

如果您的系统不支持SSD和NVMe的kyber调度程序，我们建议您使用none（或noop）调度程序。

```Bash
echo none | sudo tee /sys/block/${disk}/queue/scheduler
```

在修改此项后，运行以下命令使更改永久生效：

```Bash
chmod +x /etc/rc.d/rc.local
```

## SELinux

我们建议您禁用SELinux。

```Bash
sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sed -i 's/SELINUXTYPE/#SELINUXTYPE/' /etc/selinux/config
setenforce 0 
```

## 防火墙

如果您的防火墙已启用，请为FE节点、BE节点和Broker打开内部端口。

```Bash
systemctl stop firewalld.service
systemctl disable firewalld.service
```

## LANG变量

运行以下命令手动检查和配置LANG变量：

```Bash
echo "export LANG=en_US.UTF8" >> /etc/profile
source /etc/profile
```

## 时区

根据实际时区设置此项。

以下示例将时区设置为`/Asia/Shanghai`。

```Bash
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock
```

## ulimit配置

如果**最大文件描述符**和**最大用户进程**的值异常小，可能会导致StarRocks出现问题。

### 最大文件描述符

您可以通过运行以下命令设置最大文件描述符数：

```Bash
ulimit -n 655350
```

### 最大用户进程

您可以通过运行以下命令设置最大用户进程数：

```Bash
```Bash
ulimit -u 40960
```

## 文件系统配置

我们建议您使用 ext4 或 xfs 日志文件系统。您可以运行以下命令检查挂载类型：

```Bash
df -Th
```

## 网络配置

### tcp_abort_on_overflow

如果系统当前受到新连接尝试的过载而守护程序无法处理新连接尝试，允许系统重置新连接：

```Bash
echo 1 | sudo tee /proc/sys/net/ipv4/tcp_abort_on_overflow
```

### somaxconn

指定任何侦听套接字排队的最大连接请求数为 `1024`：

```Bash
echo 1024 | sudo tee /proc/sys/net/core/somaxconn
```

## NTP 配置

您必须在 StarRocks 集群内的节点之间配置时间同步，以确保事务的线性一致性。您可以使用 pool.ntp.org 提供的互联网时间服务，也可以在离线环境中使用内置的 NTP 服务。例如，您可以使用云服务提供商提供的 NTP 服务。

1. 检查 NTP 时间服务器是否存在。

   ```Bash
   rpm -qa | grep ntp
   ```

2. 如果不存在，则安装 NTP 服务。

   ```Bash
   sudo yum install ntp ntpdate && \
   sudo systemctl start ntpd.service && \
   sudo systemctl enable ntpd.service
   ```

3. 检查 NTP 服务。

   ```Bash
   systemctl list-unit-files | grep ntp
   ```

4. 检查 NTP 服务的连接和监控状态。

   ```Bash
   netstat -tlunp | grep ntp
   ```

5. 检查应用程序是否与 NTP 服务器同步。

   ```Bash
   ntpstat
   ```

6. 检查网络中所有配置的 NTP 服务器的状态。

   ```Bash
   ntpq -p
   ```

## 高并发配置

如果您的 StarRocks 集群具有高负载并发性，我们建议您设置以下配置：

```Bash
echo 120000 > /proc/sys/kernel/threads-max
echo 262144 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
```