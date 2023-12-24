---
displayed_sidebar: English
---

# 部署

本主题提供了有关部署的一些常见问题的解答。

## 如何在`fe.conf`文件中使用`priority_networks`参数绑定固定IP地址？

### 问题描述

例如，如果您有两个IP地址：192.168.108.23和192.168.108.43。您可以按照以下方式提供IP地址：

- 如果您将地址指定为192.168.108.23/24，StarRocks将会将其识别为192.168.108.43。
- 如果您将地址指定为192.168.108.23/32，StarRocks将会将其识别为127.0.0.1。

### 解决方案

有以下两种方法可以解决这个问题：

- 请不要在IP地址末尾添加“32”，或者将“32”更改为“28”。
- 您也可以升级到StarRocks 2.1或更高版本。

## 安装后端（BE）后为什么启动时会报错“StarRocks BE http service did not start correctly, exiting”？

安装BE时，系统会报告启动错误：StarRocks BE http service did not start correctly, exiting。

这是因为BE的Web服务端口被占用。尝试修改`be.conf`文件中的端口，然后重新启动BE。

## 在运行Java Runtime Environment (JRE)程序时，出现错误“ERROR 1064 (HY000): Could not initialize class com.starrocks.rpc.BackendServiceProxy”该怎么办？

当您在Java Runtime Environment (JRE)中运行程序时会出现此错误。要解决此问题，请将JRE替换为Java Development Kit (JDK)。我们建议您使用Oracle的JDK 1.8或更新版本。

## 部署企业版StarRocks并配置节点时，为什么会出现“Failed to Distribute files to node”错误？

在多个前端（FE）上安装的Setuptools版本不一致时，会出现此错误。要解决此问题，您可以以root用户身份执行以下命令。

```plaintext
yum remove python-setuptools

rm /usr/lib/python2.7/site-packages/setuptool* -rf

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocks的FE和BE配置是否可以在不重启集群的情况下修改生效？

是的。完成对FE和BE的修改，步骤如下：

- FE：您可以通过以下两种方式完成对FE的修改：
  - SQL

```plaintext
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

例：

```plaintext
ADMIN SET FRONTEND CONFIG ("enable_statistic_collect" = "false");
```

- Shell

```plaintext
curl --location-trusted -u username:password \
http://<ip>:<fe_http_port/api/_set_config?key=value>
```

例：

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE：您可以通过以下方式完成对BE的修改：

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意：请确保用户具有远程登录的权限。如果没有，您可以通过以下方式向用户授予权限：

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## 扩展BE磁盘空间后出现“无法获取扫描范围，在tablet:xxxxx中找不到可查询副本”的错误该怎么办？

### 问题描述

在将数据加载到主键表中时，可能会发生此错误。在数据加载过程中，目标BE没有足够的磁盘空间用于加载数据，导致BE崩溃。然后添加新磁盘以扩展磁盘空间。但是，主键表不支持磁盘空间重新平衡，并且无法将数据卸载到其他磁盘。

### 解决方案

此bug（主键表不支持BE磁盘空间重新平衡）的补丁仍在积极开发中。目前，您可以通过以下两种方式之一解决：

- 在磁盘之间手动分发数据。例如，将空间使用率高的磁盘上的目录复制到空间较大的磁盘上。
- 如果这些磁盘上的数据不重要，建议您删除磁盘并修改磁盘路径。如果此错误仍然存在，请使用[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md)清除表中的数据以释放一些空间。

## 在集群重启时启动FE时会出现错误“Fe type:unknown，is ready:false.”该怎么办？

检查leader FE是否正在运行。如果没有，请逐个重启集群中的FE节点。

## 部署集群时为什么会出现“无法获取服务信息错误”？

检查是否启用了OpenSSH守护程序（sshd）。如果没有，请运行命令`/etc/init.d/sshd`` status`以启用它。

## 启动BE时为什么会出现“cache.``host= port=0 code=THRIFT_RPC_ERROR”无法从中获取主客户端错误？

运行`netstat -anp |grep port`命令，检查`be.conf`文件中的端口是否被占用。如果是，请将占用的端口替换为空闲端口，然后重新启动BE。

## 升级企业版集群时为什么会出现“无法将升级文件传输到代理主机。来源：…”错误？

在集群升级期间，StarRocks Manager会将新版本的二进制文件分发到每个节点。如果部署目录下指定的磁盘空间不足，文件无法分发到每个节点。要解决此问题，请添加数据磁盘。

## 新部署的正常运行的FE节点在StarRocks Manager诊断页面的FE节点日志中显示“Search log failed.”该怎么办？

默认情况下，StarRocks Manager会在30秒内获取新部署的FE的路径配置。当FE启动缓慢或由于其他原因在30秒内没有响应时，会出现此错误。通过以下路径检查Manager Web的日志：

`/starrocks-manager-xxx/center/log/webcenter/log/web/``drms.INFO`（您可以自定义路径）。然后查看日志中是否显示“更新FE配置失败”的消息。如果是这样，请重启对应的FE以获取新的路径配置。

## 启动FE时为什么会出现“exceeds max permissable delta:5000ms.”错误？

当两台计算机之间的时间差超过5秒时，会出现此错误。要解决此问题，请对齐这两台机器的时间。

## 如果一个BE中有多个磁盘用于数据存储`storage_root_path`，如何设置参数？

在`be.conf`文件中配置参数`storage_root_path`，并使用`；`分隔此参数的值。例如：`storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## 将FE添加到集群后为什么会出现“invalid cluster id: 209721925.”错误？

如果在首次启动集群时未为此FE添加`--helper`选项，则两台机器之间的元数据不一致，因此会出现此错误。要解决这个问题，需要清除meta目录下的所有元数据，然后添加一个带有选项的FE`--helper`。

## FE运行时打印日志“transfer: follower”时Alive为`false`是为什么？

当使用了Java虚拟机（JVM）的一半以上的内存，并且没有标记检查点时，会出现此问题。一般情况下，系统累积50,000条日志后会标记一个检查点。建议您在这些FE没有负载较重时，修改每个FE的JVM参数并重启这些FE。
