---
displayed_sidebar: English
---

# 部署

本主题提供有关部署的一些常见问题的解答。

## 如何在 `fe.conf` 文件中使用 `priority_networks` 参数绑定固定IP地址？

### 问题描述

例如，您有两个 IP 地址：192.168.108.23 和 192.168.108.43。您可能会按如下方式提供 IP 地址：

- 如果您指定地址为 192.168.108.23/24，StarRocks 会将其识别为 192.168.108.43。
- 如果您指定地址为 192.168.108.23/32，StarRocks 会将其识别为 127.0.0.1。

### 解决方案

有以下两种方法可以解决这个问题：

- 不要在 IP 地址末尾添加“32”，或者将“32”改为“28”。
- 您也可以升级到 StarRocks 2.1 或更高版本。

## 安装后启动后端（BE）时，为什么会出现错误“StarRocks BE http 服务未正确启动，正在退出”？

在安装 BE 时，系统报告了一个启动错误：“StarRocks BE http 服务未正确启动，正在退出”。

这个错误是因为 BE 的 web 服务端口已被占用。尝试在 `be.conf` 文件中修改端口并重启 BE。

## 当出现错误“ERROR 1064 (HY000): Could not initialize class com.starrocks.rpc.BackendServiceProxy”时该怎么办？

当您在 Java 运行时环境（JRE）中运行程序时，可能会出现此错误。要解决这个问题，请用 Java Development Kit（JDK）替换 JRE。我们建议您使用 Oracle 的 JDK 1.8 或更高版本。

## 部署 StarRocks 企业版并配置节点时，为什么会出现“Failed to Distribute files to node”的错误？

当多个前端（FEs）上安装的 Setuptools 版本不一致时，会出现此错误。要解决这个问题，您可以以 root 用户身份执行以下命令：

```plaintext
yum remove python-setuptools

rm -rf /usr/lib/python2.7/site-packages/setuptool*

wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

## StarRocks 的 FE 和 BE 配置可以修改后立即生效，而不需要重启集群吗？

是的。按照以下步骤完成 FE 和 BE 的修改：

- FE：您可以通过以下任一方式完成 FE 的修改：
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
http://<ip>:<fe_http_port>/api/_set_config?key=value
```

示例：

```plaintext
curl --location-trusted -u <username>:<password> \
http://192.168.110.101:8030/api/_set_config?enable_statistic_collect=true
```

- BE：您可以通过以下方式完成 BE 的修改：

```plaintext
curl -XPOST -u username:password \
http://<ip>:<be_http_port>/api/update_config?key=value
```

> 注意：确保用户具有远程登录权限。如果没有，您可以通过以下方式授予用户权限：

```plaintext
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT SELECT_PRIV ON *.* TO 'test'@'%';
```

## 扩展 BE 磁盘空间后，出现“Failed to get scan range, no queryable replica found in tablet:xxxxx”的错误该怎么办？

### 问题描述

在向主键表加载数据时，可能会出现此错误。在数据加载过程中，目标 BE 的磁盘空间不足以存储加载的数据，导致 BE 崩溃。随后添加了新磁盘以扩展磁盘空间。但是，主键表不支持磁盘空间重新平衡，数据无法转移到其他磁盘。

### 解决方案

针对此问题（主键表不支持 BE 磁盘空间重新平衡）的补丁仍在开发中。目前，您可以通过以下两种方式解决：

- 手动在磁盘之间分配数据。例如，将数据从使用率较高的磁盘复制到空间较大的磁盘。
- 如果这些磁盘上的数据不重要，我们建议您删除这些磁盘并修改磁盘路径。如果问题仍然存在，请使用 [TRUNCATE TABLE](../sql-reference/sql-statements/data-definition/TRUNCATE_TABLE.md) 清空表中的数据以释放空间。

## 集群重启期间启动 FE 时，为什么会出现“Fe type:unknown, is ready:false.”的错误？

检查领导者 FE 是否在运行。如果不是，请依次重启集群中的 FE 节点。

## 部署集群时，为什么会出现“failed to get service info err.”的错误？

检查 OpenSSH 守护进程（sshd）是否已启用。如果没有，请运行 `/etc/init.d/sshd status` 命令来启用它。

## 启动 BE 时，为什么会出现“Fail to get master client from cache. host= port=0 code=THRIFT_RPC_ERROR”的错误？

运行 `netstat -anp | grep port` 命令检查 `be.conf` 文件中的端口是否被占用。如果是，请更换一个未被占用的端口，然后重启 BE。

## 升级企业版集群时，为什么会出现“Failed to transport upgrade files to agent host. src:…”的错误？

当部署目录中指定的磁盘空间不足时，会出现此错误。在集群升级过程中，StarRocks Manager 将新版本的二进制文件分发到每个节点。如果部署目录中的磁盘空间不足，文件无法分发到各个节点。解决此问题的方法是增加数据磁盘。

## StarRocks Manager 的诊断页面上，为什么新部署的正常运行的 FE 节点的日志显示“Search log failed.”？

默认情况下，StarRocks Manager 会在 30 秒内获取新部署的 FE 的路径配置。如果 FE 启动缓慢或由于其他原因在 30 秒内没有响应，就会出现这个错误。通过以下路径检查 Manager Web 的日志：

`/starrocks-manager-xxx/center/log/webcenter/log/web/drms.INFO`（路径可以自定义）。然后查看日志中是否显示“Failed to update FE configurations”。如果是，请重启对应的 FE 以获取新的路径配置。

## 启动 FE 时，为什么会出现“exceeds max permissible delta:5000ms.”的错误？

当两台机器之间的时间差超过 5 秒时，就会出现这个错误。要解决这个问题，需要同步这两台机器的时间。

## 如果 BE 有多个磁盘用于数据存储，应该如何设置 `storage_root_path` 参数？

在 `be.conf` 文件中配置 `storage_root_path` 参数，用分号 `;` 分隔各个值。例如：`storage_root_path=/the/path/to/storage1;/the/path/to/storage2;/the/path/to/storage3;`

## 在 FE 添加到我的集群后，为什么会出现“invalid cluster id: 209721925.”的错误？

如果在首次启动集群时没有为该 FE 添加 `--helper` 选项，两台机器之间的元数据将不一致，从而导致这个错误。要解决这个问题，您需要清除 meta 目录下的所有元数据，然后使用 `--helper` 选项再次添加 FE。

## 当 FE 正在运行并打印日志 `transfer: follower` 时，为什么 Alive 会是 `false`？

当 Java 虚拟机（JVM）的内存使用超过一半并且没有标记检查点时，就会出现这个问题。通常，系统累积 50,000 条日志后会标记一个检查点。我们建议您修改每个 FE 的 JVM 参数，并在负载较轻时重启这些 FE。