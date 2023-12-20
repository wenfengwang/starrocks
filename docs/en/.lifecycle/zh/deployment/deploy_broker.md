---
unlisted: true
---

# 部署和管理 Broker 节点

本主题介绍如何部署 Broker 节点。通过 Broker 节点，StarRocks 能够从诸如 HDFS 和 S3 的数据源读取数据，并利用其自身的计算资源进行数据预处理、加载和备份。

我们建议您在每个托管 BE 节点的实例上部署一个 Broker 节点，并使用相同的 `broker_name` 添加所有 Broker 节点。Broker 节点在处理任务时会自动平衡数据传输负载。

Broker 节点通过网络连接将数据传输到 BE 节点。当 Broker 节点和 BE 节点部署在同一台机器上时，Broker 节点会将数据传输到本地 BE 节点。

## 在您开始之前

请确保您已按照[部署先决条件](../deployment/deployment_prerequisites.md)、[检查环境配置](../deployment/environment_configurations.md)和[准备部署文件](../deployment/prepare_deployment_files.md)中提供的说明完成所需的配置。

## 启动 Broker 服务

以下过程在 BE 实例上执行。

1. 导航至您之前准备的 [StarRocks Broker 部署文件](../deployment/prepare_deployment_files.md)所在目录，修改 Broker 配置文件 **apache_hdfs_broker/conf/apache_hdfs_broker.conf**。

   如果实例上的 HDFS Thrift RPC 端口（`broker_ipc_port`，默认值：`8000`）被占用，您必须在 Broker 配置文件中指定一个有效的备用端口。

   ```YAML
   broker_ipc_port = aaaa        # 默认值：8000
   ```

   下表列出了 Broker 支持的配置项。

   | 配置项 | 默认值 | 单位 | 说明 |
   |---|---|---|---|
   | hdfs_read_buffer_size_kb | 8192 | KB | 用于从 HDFS 读取数据的缓冲区大小。|
   | hdfs_write_buffer_size_kb | 1024 | KB | 用于将数据写入 HDFS 的缓冲区大小。|
   | client_expire_seconds | 300 | 秒 | 如果客户端会话在指定时间后未收到任何 ping，则会被删除。|
   | broker_ipc_port | 8000 | N/A | HDFS Thrift RPC 端口。|
   | disable_broker_client_expiration_checking | false | N/A | 是否禁用检查和清理过期的 OSS 文件描述符，这在某些情况下可能导致 broker 在 OSS 关闭时卡住。为了避免这种情况，您可以将此参数设置为 `true` 以禁用检查。|
   | sys_log_dir | `${BROKER_HOME}/log` | N/A | 存储系统日志（包括 INFO、WARNING、ERROR 和 FATAL）的目录。|
   | sys_log_level | INFO | N/A | 日志级别。有效值包括 INFO、WARNING、ERROR 和 FATAL。|
   | sys_log_roll_mode | SIZE-MB-1024 | N/A | 系统日志分段成日志卷的模式。有效值包括 TIME-DAY、TIME-HOUR 和 SIZE-MB-nnn。默认值表示日志被分段成每个 1 GB 的卷。|
   | sys_log_roll_num | 30 | N/A | 保留的日志卷数量。|
   | audit_log_dir | `${BROKER_HOME}/log` | N/A | 存储审计日志文件的目录。|
   | audit_log_modules | 空字符串 | N/A | StarRocks 生成审计日志条目的模块。默认情况下，StarRocks 为 slow_query 模块和 query 模块生成审计日志。您可以指定多个模块，模块名称之间必须用逗号（,）和空格分隔。|
   | audit_log_roll_mode | TIME-DAY | N/A | 有效值包括 `TIME-DAY`、`TIME-HOUR` 和 `SIZE-MB-<size>`。|
   | audit_log_roll_num | 10 | N/A | 如果 `audit_log_roll_mode` 设置为 `SIZE-MB-<size>`，则此配置项不生效。|
   | sys_log_verbose_modules | com.starrocks | N/A | StarRocks 生成系统日志的模块。有效值为 BE 中的命名空间，包括 `starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream` 和 `starrocks::workgroup`。|

2. 启动 Broker 节点。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. 检查 Broker 日志以验证 Broker 节点是否成功启动。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 您可以通过在其他实例上重复上述步骤来启动新的 Broker 节点。

## 将 Broker 节点添加到集群

以下过程在 MySQL 客户端上执行。您必须安装 MySQL 客户端 5.5.0 或更高版本。

1. 通过 MySQL 客户端连接到 StarRocks。您需要使用初始账户 `root` 登录，密码默认为空。

   ```Bash
   # 将 <fe_address> 替换为您连接的 FE 节点的 IP 地址（priority_networks）或 FQDN，
   # 将 <query_port>（默认值：9030）替换为您在 fe.conf 中指定的 query_port。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 运行以下命令将 Broker 节点添加到集群中。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_address>:<broker_ipc_port>";
   ```

      > **注意**
   - 您可以使用上述命令一次性添加多个 Broker 节点。每对 `<broker_address>:<broker_ipc_port>` 表示一个 Broker 节点。
   - 您可以添加多个具有相同 `broker_name` 的 Broker 节点。

3. 通过 MySQL 客户端验证 Broker 节点是否已正确添加到集群中。

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

当 `Alive` 字段为 `true` 时，表示该 Broker 已正确启动并添加到集群中。

## 停止 Broker 节点

运行以下命令以停止 Broker 节点。

```bash
./bin/stop_broker.sh --daemon
```

## 升级 Broker 节点

1. 导航至 Broker 节点的工作目录并停止该节点。

   ```Bash
   # 将 <broker_dir> 替换为 Broker 节点的部署目录。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 用新版本的部署文件替换 **bin** 和 **lib** 目录下的原始部署文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. 启动 Broker 节点。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. 检查 Broker 节点是否成功启动。

   ```Bash
   ps aux | grep broker
   ```

5. 重复上述步骤以升级其他 Broker 节点。

## 降级 Broker 节点

1. 导航至 Broker 节点的工作目录并停止该节点。

   ```Bash
   # 将 <broker_dir> 替换为 Broker 节点的部署目录。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 将 **bin** 和 **lib** 目录下的原始部署文件替换为早期版本的文件。

   ```Bash
   mv lib lib.bak 
   mv bin bin.bak
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/lib  .   
   cp -r /tmp/StarRocks-x.x.x/apache_hdfs_broker/bin  .
   ```

3. 启动 Broker 节点。

   ```Bash
   sh bin/start_broker.sh --daemon
   ```

4. 检查 Broker 节点是否成功启动。

   ```Bash
   ps aux | grep broker
   ```

5. 重复上述步骤以对其他 Broker 节点进行降级。