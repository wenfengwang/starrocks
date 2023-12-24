---
unlisted: true
---

# 部署和管理 Broker 节点

本主题描述如何部署 Broker 节点。使用 Broker 节点，StarRocks 可以从 HDFS 和 S3 等来源读取数据，并利用其自身的计算资源对数据进行预处理、加载和备份。

我们建议您在托管 BE 节点的每个实例上部署一个 Broker 节点，并使用相同的 `broker_name`。在处理任务时，Broker 节点会自动平衡数据传输负载。

Broker 节点使用网络连接将数据传输到 BE 节点。当 Broker 节点和 BE 节点部署在同一台机器上时，Broker 节点会将数据传输到本地 BE 节点。

## 准备工作

请确保您已按照[部署先决条件](../deployment/deployment_prerequisites.md)中提供的说明完成所需的配置，以及[检查环境配置](../deployment/environment_configurations.md)和[准备部署文件](../deployment/prepare_deployment_files.md)。

## 启动 Broker 服务

以下过程在 BE 实例上执行。

1. 进入存储[StarRocks Broker 部署文件](../deployment/prepare_deployment_files.md)的目录，并修改 Broker 配置文件 **apache_hdfs_broker/conf/apache_hdfs_broker.conf**。

   如果实例上的 HDFS Thrift RPC 端口（`broker_ipc_port` 默认：`8000`）被占用，则必须在 Broker 配置文件中指定有效的替代端口。

   ```YAML
   broker_ipc_port = aaaa        # 默认值：8000
   ```

   下表列出了 Broker 支持的配置项。

   | 配置项 | 默认值 | 单位 | 描述 |
   | ------------------------- | ------------------ | ------ | ------------------------------------------------------------ |
   | hdfs_read_buffer_size_kb | 8192 | KB | 用于从 HDFS 读取数据的缓冲区大小。 |
   | hdfs_write_buffer_size_kb | 1024 | KB | 用于将数据写入 HDFS 的缓冲区大小。 |
   | client_expire_seconds | 300 | 秒 | 如果客户端会话在指定时间内未收到任何 ping，则将删除这些会话。 |
   | broker_ipc_port | 8000 | N/A | HDFS Thrift RPC 端口。 |
   | disable_broker_client_expiration_checking | false | N/A | 是否禁用对过期 OSS 文件描述符的检查和清除，某些情况下会导致 Broker 在关闭 OSS 时出现卡顿。为避免这种情况，您可以将此参数设置为 `true` 以禁用检查。 |
   | sys_log_dir | `${BROKER_HOME}/log` | N/A | 用于存储系统日志（包括 INFO、WARNING、ERROR 和 FATAL）的目录。 |
   | sys_log_level | INFO | N/A | 日志级别。有效值包括 INFO、WARNING、ERROR 和 FATAL。 |
   | sys_log_roll_mode | SIZE-MB-1024 | N/A | 系统日志切分为日志卷的模式。有效值包括 TIME-DAY、TIME-HOUR 和 SIZE-MB-nnn。默认值表示日志切分为每个 1GB 的卷。 |
   | sys_log_roll_num | 30 | N/A | 要保留的日志卷数。 |
   | audit_log_dir | `${BROKER_HOME}/log` | N/A | 存储审计日志文件的目录。 |
   | audit_log_modules | 空字符串 | N/A | StarRocks 生成审计日志条目的模块。默认情况下，StarRocks 为 slow_query 模块和 query 模块生成审计日志。您可以指定多个模块，模块名称必须用逗号（,）和空格分隔。 |
   | audit_log_roll_mode | TIME-DAY | N/A | 有效值包括 `TIME-DAY`、`TIME-HOUR` 和 `SIZE-MB-<size>`。 |
   | audit_log_roll_num | 10 | N/A | 如果 audit_log_roll_mode 设置为 `SIZE-MB-<size>`，则此配置无效。 |
   | sys_log_verbose_modules | com.starrocks | N/A | StarRocks 生成系统日志的模块。有效值为 BE 中的命名空间，包括 `starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream` 和 `starrocks::workgroup`。 |

2. 启动 Broker 节点。

   ```bash
   ./apache_hdfs_broker/bin/start_broker.sh --daemon
   ```

3. 检查 Broker 日志，验证 Broker 节点是否成功启动。

   ```Bash
   cat apache_hdfs_broker/log/apache_hdfs_broker.log | grep thrift
   ```

4. 您可以通过在其他实例上重复上述过程来启动新的 Broker 节点。

## 将 Broker 节点添加到集群

以下过程在 MySQL 客户端上执行。您必须安装 MySQL 客户端 5.5.0 或更高版本。

1. 通过 MySQL 客户端连接 StarRocks。您需要使用初始帐户 `root` 登录，默认密码为空。

   ```Bash
   # 用 FE 节点的 IP 地址（priority_networks）或 FQDN 替换 <fe_address>，并用 fe.conf 中指定的 query_port（默认值：9030）替换 <query_port>。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 执行以下命令，将 Broker 节点添加到集群中。

   ```sql
   ALTER SYSTEM ADD BROKER <broker_name> "<broker_address>:<broker_ipc_port>";
   ```

   > **注意**
   >
   > - 您可以使用上述命令一次添加多个 Broker 节点。每个 `<broker_address>:<broker_ipc_port>` 对代表一个 Broker 节点。
   > - 您可以使用相同的 `broker_name` 添加多个 Broker 节点。

3. 通过 MySQL 客户端验证 Broker 节点是否已正确添加。

```sql
SHOW PROC "/brokers"\G
```

示例：

```plain text
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

1. 进入 Broker 节点的工作目录并停止节点。

   ```Bash
   # 用 Broker 节点的部署目录替换 <broker_dir>。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 用新版本的**部署文件**替换**bin**和**lib**目录下的原始部署文件。

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

5. 重复上述过程以升级其他 Broker 节点。

## 降级 Broker 节点

1. 进入 Broker 节点的工作目录并停止节点。

   ```Bash
   # 用 Broker 节点的部署目录替换 <broker_dir>。
   cd <broker_dir>/apache_hdfs_broker
   sh bin/stop_broker.sh
   ```

2. 用早期版本的**部署文件**替换**bin**和**lib**目录下的原始部署文件。

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

5. 重复上述过程以降级其他 Broker 节点。
