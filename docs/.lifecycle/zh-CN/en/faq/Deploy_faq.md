---
displayed_sidebar: "Chinese"
---

# 部署

本主题提供对部署的一些常见问题的回答。

## 我如何在`fe.conf`文件的`priority_networks`参数中绑定固定IP地址？

### 问题描述

例如，如果您有两个IP地址：192.168.108.23和192.168.108.43。您可以按照以下方式提供IP地址：

- 如果您将地址指定为192.168.108.23/24，StarRocks将把它们识别为192.168.108.43。
- 如果您将地址指定为192.168.108.23/32，StarRocks将把它们识别为127.0.0.1。

### 解决方案

有以下两种方式来解决此问题：

- 不要在IP地址的末尾添加“32”，或者将“32”更改为“28”。
- 您也可以升级到 StarRocks 2.1 或更高版本。

## 在安装后启动后端（BE）时，为什么会出现错误“StarRocks BE http service did not start correctly, exiting”？

在安装BE时，系统报告了启动错误：StarRocks Be http service did not start correctly, exiting。

这个错误是由于BE的Web服务端口被占用。尝试修改`be.conf`文件中的端口，然后重新启动BE。

## 当运行Java Runtime Environment（JRE）中的程序时出现错误“ERROR 1064 (HY000): Could not initialize class com.starrocks.rpc.BackendServiceProxy”该怎么办？

当您运行Java Runtime Environment（JRE）中的程序时会出现此错误。要解决此问题，请用Java Development Kit（JDK）替换JRE。我们建议您使用Oracle的JDK 1.8或更高版本。

## 在部署企业版的StarRocks并配置节点时，为什么会出现错误“Failed to Distribute files to node”？

在安装了多个前端（FE）上不一致的Setuptools版本时会出现此错误。为了解决此问题，您可以以root用户身份执行以下命令。

```plaintext
yum remove python-setuptools

rm /usr/lib/python2.7/site-packages/setuptool* -rf

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocks的FE和BE配置是否可以修改后立即生效，而无需重新启动集群？

是的。执行以下步骤完成对FE和BE的修改：

- FE：您可以通过以下一种方式完成对FE的修改：
  - SQL

```plaintext
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

示例：

```plaintext
ADMIN SET FRONTEND CONFIG ("enable_statistic_collect" = "false");
```

- Shell

```plaintext
curl --location-trusted -u username:password \
http://<ip>:<fe_http_port/api/_set_config?key=value>
```

示例：

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE：您可以通过以下方式完成对BE的修改：

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意：确保用户具有远程登录权限。如果没有，您可以通过以下方式授予用户权限：

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## 在扩展BE磁盘空间后，如果出现错误“Failed to get scan range, no queryable replica found in tablet:xxxxx”该怎么办？

### 问题描述

这个错误可能在加载主键表数据时发生。在数据加载期间，目标BE没有足够的磁盘空间来存储加载的数据，因此BE会崩溃。然后会添加新磁盘以扩展磁盘空间。然而，主键表不支持磁盘空间再平衡，因此数据无法转移到其他磁盘。

### 解决方案

目前，针对此错误（主键表不支持BE磁盘空间再平衡）的修复程序仍在积极开发中。目前，您可以通过以下两种方式来解决：

- 在磁盘之间手动分发数据。例如，将占用空间较大的磁盘上的目录复制到空间较大的磁盘上。
- 如果这些磁盘上的数据不重要，我们建议您删除磁盘并修改磁盘路径。如果此错误仍然存在，请使用[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md)清除表中的数据，以释放一些空间。

## 在集群重新启动期间启动FE时为什么会出现错误“Fe type:unknown ,is ready :false.”？

检查主FE是否正在运行。如果没有运行，请逐个重新启动集群中的FE节点。

## 在部署集群时为什么会出现错误“failed to get service info err.”？

检查OpenSSH守护程序（sshd）是否已启用。如果没有启用，请运行`/etc/init.d/sshd`` status`命令以启用它。

## 在启动BE时为什么会出现错误“Fail to get master client from `cache. ``host= port=0 code=THRIFT_RPC_ERROR`”？

运行`netstat -anp |grep port`命令检查`be.conf`文件中的端口是否被占用。如果是，用空闲端口替换被占用的端口，然后重新启动BE。

## 在升级企业版集群时为什么会出现错误“Failed to transport upgrade files to agent host. src:…”？

在集群升级期间，StarRocks Manager将新版本的二进制文件分发到每个节点。如果部署目录中指定的磁盘空间不足，文件将无法分发到每个节点。为解决此问题，请添加数据磁盘。

## 为什么StarRocks Manager的诊断页面上显示新部署的FE节点正在正常运行，但日志显示“Search log failed.”？

默认情况下，StarRocks Manager在30秒内获取新部署的FE的路径配置。如果由于其他原因，FE启动速度较慢或在30秒内没有响应，就会出现此错误。通过以下路径检查Manager Web的日志：

`/starrocks-manager-xxx/center/log/webcenter/log/web/``drms.INFO`（您可以自定义路径）。然后查找日志中是否显示了消息“Failed to update FE configurations”。如果有，请重新启动相应的FE以获取新的路径配置。

## 在启动FE时为什么会出现错误“exceeds max permissable delta:5000ms.”？

当两台机器的时间差异大于5秒时就会出现此错误。要解决此问题，请使这两台机器的时间对齐。

## 如果BE有多个磁盘用于数据存储，我该如何设置`storage_root_path`参数？

在`be.conf`文件中配置`storage_root_path`参数，并用`;`分隔此参数的值。例如：`storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## 在添加FE到集群后为什么会出现错误“invalid cluster id: 209721925.”？

如果您在第一次启动集群时未为此FE添加`--helper`选项，则两台机器之间的元数据是不一致的，因此会出现这个错误。要解决此问题，您需要清除元数据目录下的所有元数据，然后使用`--helper`选项添加FE。

## 当FE正在运行并打印日志“transfer: follower”时，Alive是`false`是为什么？

当Java虚拟机（JVM）的内存使用超过一半且没有标记检查点时就会出现此问题。一般来说，在系统累积了50,000条日志后，会标记检查点。我们建议您在FE的负载不重时修改每个FE的JVM参数，并重启这些FE。

