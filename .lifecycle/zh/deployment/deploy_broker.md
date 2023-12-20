---
unlisted: true
---

# 部署和管理Broker节点

本主题介绍如何部署Broker节点。通过Broker节点，StarRocks能够从诸如HDFS和S3这样的数据源读取数据，并使用其自身的计算资源进行数据的预处理、加载和备份。

我们建议您在每个托管BE节点的实例上部署一个Broker节点，并且使用相同的broker_name来添加所有Broker节点。Broker节点在处理任务时会自动平衡数据传输负载。

Broker节点通过网络连接向BE节点传输数据。当Broker节点和BE节点部署在同一台机器上时，Broker节点会将数据传输给本地的BE节点。

## 开始之前

请确保您已经按照[部署先决条件](../deployment/deployment_prerequisites.md)、[检查环境配置](../deployment/environment_configurations.md)和[准备部署文件](../deployment/prepare_deployment_files.md)中提供的指南完成了必要的配置。

## 启动Broker服务

以下步骤在BE实例上执行。

1. 导航至您之前准备的[StarRocks Broker部署文件](../deployment/prepare_deployment_files.md)存放的目录，并修改Broker配置文件**apache_hdfs_broker/conf/apache_hdfs_broker.conf**。

   如果实例上的HDFS Thrift RPC端口（broker_ipc_port，默认值：8000）已被占用，您必须在Broker配置文件中指定一个有效的备用端口。

   ```YAML
   broker_ipc_port = aaaa        # Default: 8000
   ```

   下表列出了Broker支持的配置项。

   |配置项|默认|单位|说明|
|---|---|---|---|
   |hdfs_read_buffer_size_kb|8192|KB|用于从 HDFS 读取数据的缓冲区的大小。|
   |hdfs_write_buffer_size_kb|1024|KB|用于将数据写入 HDFS 的缓冲区的大小。|
   |client_expire_seconds|300|第二个|如果客户端会话在指定时间后未收到任何 ping，则将被删除。|
   |broker_ipc_port|8000|N/A|HDFS thrift RPC 端口。|
   |disable_broker_client_expiration_checking|false|N/A|是否禁止检查和清除过期的OSS文件描述符，这在某些情况下会导致在OSS关闭时broker卡住。为了避免这种情况，您可以将此参数设置为 true 以禁用检查。|
   |sys_log_dir|${BROKER_HOME}/log|N/A|用于存储系统日志（包括INFO、WARNING、ERROR和FATAL）的目录。|
   |sys_log_level|INFO|N/A|日志级别。有效值包括 INFO、WARNING、ERROR 和 FATAL。|
   |sys_log_roll_mode|SIZE-MB-1024|N/A|系统日志分段为日志卷的模式。有效值包括 TIME-DAY、TIME-HOUR 和 SIZE-MB-nnn。默认值表示日志被分段为每个卷 1 GB。|
   |sys_log_roll_num|30|N/A|要保留的日志卷数。|
   |audit_log_dir|${BROKER_HOME}/log|N/A|存储审核日志文件的目录。|
   |audit_log_modules|空字符串|N/A|StarRocks 为其生成审核日志条目的模块。默认情况下，StarRocks 会生成 Slow_query 模块和 query 模块的审核日志。您可以指定多个模块，模块名称必须用逗号 (,) 和空格分隔。|
   |audit_log_roll_mode|TIME-DAY|N/A|有效值包括 TIME-DAY、TIME-HOUR 和 SIZE-MB-<大小>。|
   |audit_log_roll_num|10|N/A|如果audit_log_roll_mode 设置为SIZE-MB-<size>，则此配置不起作用。|
   |sys_log_verbose_modules|com.starrocks|N/A|StarRocks 为其生成系统日志的模块。有效值为 BE 中的命名空间，包括 starrocks、starrocks::debug、starrocks::fs、starrocks::io、starrocks::lake、starrocks::pipeline、starrocks::query_cache、starrocks::stream 和 starrocks::workgroup .|

2. 启动Broker节点。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. 检查Broker日志，以确认Broker节点是否成功启动。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 您可以通过在其他实例上重复上述步骤来启动新的Broker节点。

## 将Broker节点添加到集群

以下步骤在MySQL客户端上执行。您必须安装5.5.0版本或更高版本的MySQL客户端。

1. 通过MySQL客户端连接到StarRocks。您需要使用初始账户root登录，密码默认为空。

   ```Bash
   # Replace <fe_address> with the IP address (priority_networks) or FQDN 
   # of the FE node you connect to, and replace <query_port> (Default: 9030) 
   # with the query_port you specified in fe.conf.
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 运行以下命令将Broker节点添加到集群。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_address>:<broker_ipc_port>";
   ```

      > **注意**
   - 您可以使用上述命令一次添加多个Broker节点。每对<broker_address>:<broker_ipc_port>表示一个Broker节点。
   - 您可以添加多个具有相同broker_name的Broker节点。

3. 通过MySQL客户端验证Broker节点是否已正确添加至集群。

```sql
SHOW PROC "/brokers"\G
```

示例：

```plain
MySQL [(none)]> SHOW PROC "/brokers"\G
*************************** 1. row ***************************
          Name: broker1
            IP: x.x.x.x
          Port: 8000
         Alive: true
 LastStartTime: 2022-05-19 11:21:36
LastUpdateTime: 2022-05-19 11:28:31
        ErrMsg:
1 row in set (0.00 sec)
```

当字段Alive为true时，这表示Broker已正确启动并添加到了集群中。

## 停止Broker节点

运行以下命令以停止Broker节点。

```bash
./bin/stop_broker.sh --daemon
```

## 升级Broker节点

1. 导航至Broker节点的工作目录并停止节点。

   ```Bash
   # Replace <broker_dir> with the deployment directory of the Broker node.
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 用新版本的部署文件替换**bin**和**lib**目录下的原始文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. 启动Broker节点。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. 检查Broker节点是否成功启动。

   ```Bash
   ps aux | grep broker
   ```

5. 重复上述步骤以升级其他Broker节点。

## 降级Broker节点

1. 导航至Broker节点的工作目录并停止节点。

   ```Bash
   # Replace <broker_dir> with the deployment directory of the Broker node.
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 用旧版本的部署文件替换**bin**和**lib**目录下的原始文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. 启动Broker节点。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. 检查Broker节点是否成功启动。

   ```Bash
   ps aux | grep broker
   ```

5. 重复上述步骤以降级其他Broker节点。
