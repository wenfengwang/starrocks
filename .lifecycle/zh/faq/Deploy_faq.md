---
displayed_sidebar: English
---

# 部署

本主题提供了一些关于部署的常见问题解答。

## 如何在fe.conf文件中使用priority_networks参数绑定固定IP地址？

### 问题描述

例如，您有两个IP地址：192.168.108.23和192.168.108.43。您可能会这样提供IP地址：

- 如果您指定地址为192.168.108.23/24，StarRocks会识别为192.168.108.43。
- 如果您指定地址为192.168.108.23/32，StarRocks会识别为127.0.0.1。

### 解决方案

有以下两种方法可以解决这个问题：

- 不要在IP地址后面添加“32”，或者将“32”改为“28”。
- 您也可以升级到StarRocks 2.1或更高版本。

## 为什么在安装后启动后端（BE）时会出现“StarRocks BE http服务未正确启动，正在退出”的错误？

安装BE时，系统报告了一个启动错误：“StarRocks BE http服务未正确启动，正在退出”。

这个错误是因为BE的Web服务端口已被占用。尝试修改be.conf文件中的端口并重启BE。

## 当出现错误“ERROR 1064 (HY000): 无法初始化类com.starrocks.rpc.BackendServiceProxy”时该怎么办？

在Java运行时环境（JRE）中运行程序时，可能会出现这个错误。要解决这个问题，可以用Java Development Kit（JDK）替换JRE。我们建议您使用Oracle的JDK 1.8或更新版本。

## 为什么在部署StarRocks企业版并配置节点时会出现“无法将文件分发到节点”的错误？

当多个前端（FEs）上安装的Setuptools版本不一致时，可能会出现这个错误。要解决这个问题，您可以以root用户身份执行以下命令。

```plaintext
yum remove python-setuptools

rm /usr/lib/python2.7/site-packages/setuptool* -rf

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocks的FE和BE配置可以修改后不重启集群即可生效吗？

可以。按照以下步骤完成FE和BE的修改：

- FE：您可以通过以下任一方式完成FE的修改：
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

- BE：您可以通过以下方式完成BE的修改：

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意：确保用户具有远程登录权限。如果没有，您可以通过以下方式授予用户权限：

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON . TO 'test'@'%';
```

## 如果在BE扩容后出现“获取扫描范围失败，在tablet:xxxxx中未找到可查询副本”的错误，该怎么办？

### 问题描述

在将数据加载到主键表时，可能会出现这个错误。在数据加载过程中，目标BE的磁盘空间不足以存放加载的数据，导致BE崩溃。之后添加了新磁盘来扩展磁盘空间。然而，主键表不支持磁盘空间重新平衡，数据无法转移到其他磁盘。

### 解决方案

针对这个问题（主键表不支持BE磁盘空间重新平衡）的修补程序仍在开发中。目前，您可以通过以下两种方式解决：

- 手动在磁盘之间分配数据。例如，将数据从高使用率的磁盘复制到空间较大的磁盘。
- 如果这些磁盘上的数据不重要，我们建议您删除磁盘并修改磁盘路径。如果问题仍然存在，请使用[TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md)命令清空表中的数据，以释放空间。

## 当我在集群重启期间启动FE时，为什么会出现“Fe类型：未知，已就绪：false”的错误？

检查领导者FE是否在运行。如果不是，请依次重启集群中的FE节点。

## 当我部署集群时，为什么会出现“failed to get service info err”错误？

检查OpenSSH守护进程（sshd）是否已启用。如果没有，请运行/etc/init.d/sshd status命令来启用它。

## 当我启动BE时，为什么会出现“无法从缓存中获取主客户端。host= port=0 code=THRIFT_RPC_ERROR”错误？

运行netstat -anp | grep port命令检查be.conf文件中的端口是否被占用。如果是，请更换为未被占用的端口，然后重启BE。

## 为什么在升级企业版集群时会出现“无法将升级文件传输到代理主机。源：…”的错误？

当部署目录中指定的磁盘空间不足时，会出现此错误。在集群升级期间，StarRocks Manager会将新版本的二进制文件分发给每个节点。如果部署目录中指定的磁盘空间不足，文件将无法分发给每个节点。要解决此问题，请添加数据磁盘。

## 为什么StarRocks Manager的诊断页面上，新部署的正常运行的FE节点的日志会显示“搜索日志失败”？

默认情况下，StarRocks Manager会在30秒内获取新部署的FE的路径配置。如果FE启动缓慢或由于其他原因在30秒内无响应，将出现此错误。通过以下路径检查Manager Web的日志：

/starrocks-manager-xxx/center/log/webcenter/log/web/drms.INFO（您可以自定义路径）。然后查看日志中是否显示“更新FE配置失败”的消息。如果是，请重启相应的FE以获取新的路径配置。

## 当我启动FE时，为什么会出现“超出最大允许的偏差：5000ms”的错误？

当两台机器之间的时间差超过5秒时，就会出现此错误。要解决此问题，请同步这两台机器的时间。

## 如果BE有多个用于数据存储的磁盘，我应该如何设置storage_root_path参数？

在be.conf文件中设置storage_root_path参数，并用分号分隔各个值。例如：storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;

## 在FE添加到我的集群后，为什么会出现“无效的集群ID：209721925”错误？

如果您在首次启动集群时没有为该FE添加--helper选项，两台机器之间的元数据可能不一致，导致此错误。要解决此问题，您需要清除meta目录下的所有元数据，然后使用--helper选项再次添加FE。

## 为什么FE运行时Alive为false，并且日志显示“transfer: follower”？

当Java虚拟机（JVM）的内存使用超过一半并且没有标记检查点时，就会发生这种情况。通常，系统积累了5万条日志后会标记一个检查点。我们建议您修改每个FE的JVM参数，并在负载较轻时重启这些FE。
