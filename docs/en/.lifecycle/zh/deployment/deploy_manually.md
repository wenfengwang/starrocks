---
displayed_sidebar: English
---

# 手动部署 StarRocks

本主题描述如何手动部署无共享 StarRocks（其中 BE 负责存储和计算）。有关其他安装模式，请参阅[部署概述](../deployment/deployment_overview.md)。

要部署共享数据 StarRocks 集群（存储和计算解耦），请参阅[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)

## 步骤 1：启动 Leader FE 节点

以下操作在 FE 实例上执行。

1. 创建专用目录用于存储元数据。我们建议将元数据存储在与 FE 部署文件不同的目录中。确保该目录存在，并且您具有对其的写入权限。

   ```YAML
   # 用要创建的元数据目录替换 <meta_dir>。
   mkdir -p <meta_dir>
   ```

2. 进入之前准备的[StarRocks FE 部署文件](../deployment/prepare_deployment_files.md)目录，并修改 FE 配置文件 **fe/conf/fe.conf**。

   a. 在配置项 `meta_dir` 中指定元数据目录。

      ```YAML
      # 用您创建的元数据目录替换 <meta_dir>。
      meta_dir = <meta_dir>
      ```

   b. 如果[环境配置清单](../deployment/environment_configurations.md#fe-ports)中提到的任何 FE 端口已被占用，必须在 FE 配置文件中分配有效的替代项。

      ```YAML
      http_port = aaaa        # 默认：8030
      rpc_port = bbbb         # 默认：9020
      query_port = cccc       # 默认：9030
      edit_log_port = dddd    # 默认：9010
      ```

      > **注意**
      >
      > 如果要在集群中部署多个 FE 节点，必须为每个 FE 节点分配相同的 `http_port`。

   c. 如果要为集群启用 IP 地址访问，必须在配置文件中添加配置项 `priority_networks`，并为 FE 节点分配一个专用的 IP 地址（CIDR 格式）。如果要为集群启用[FQDN 访问](../administration/enable_fqdn.md)，可以忽略此配置项。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > 您可以在终端中运行 `ifconfig` 查看实例拥有的 IP 地址。

   d. 如果在实例上安装了多个 JDK，并且要使用与环境变量 `JAVA_HOME` 中指定的 JDK 不同的特定 JDK，必须在配置文件中添加配置项 `JAVA_HOME`，指定所选 JDK 安装的路径。

      ```YAML
      # 用所选 JDK 安装的路径替换 <path_to_JDK>。
      JAVA_HOME = <path_to_JDK>
      ```

   f. 有关高级配置项的信息，请参阅[参数配置 - FE 配置项](../administration/FE_configuration.md#fe-configuration-items)。

3. 启动 FE 节点。

   - 要为集群启用 IP 地址访问，请运行以下命令启动 FE 节点：

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - 要为集群启用 FQDN 访问，请运行以下命令启动 FE 节点：

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     请注意，首次启动节点时只需指定一次 `--host_type` 参数。

     > **注意**
     >
     > 在启动启用了 FQDN 访问的 FE 节点之前，请确保您已为所有实例在 **/etc/hosts** 中分配了主机名。有关详细信息，请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。

4. 检查 FE 日志，验证 FE 节点是否成功启动。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   如果日志中出现类似 "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020." 的记录，则表示 FE 节点已成功启动。

## 步骤 2：启动 BE 服务

以下操作在 BE 实例上执行。

1. 创建专用目录用于数据存储。我们建议将数据存储在与 BE 部署目录不同的目录中。确保该目录存在，并且您具有对其的写入权限。

   ```YAML
   # 用要创建的数据存储目录替换 <storage_root_path>。
   mkdir -p <storage_root_path>
   ```

2. 进入之前准备的[StarRocks BE 部署文件](../deployment/prepare_deployment_files.md)目录，并修改 BE 配置文件 **be/conf/be.conf**。

   a. 在配置项 `storage_root_path` 中指定数据目录。

      ```YAML
      # 用您创建的数据目录替换 <storage_root_path>。
      storage_root_path = <storage_root_path>
      ```

   b. 如果[环境配置清单](../deployment/environment_configurations.md#be-ports)中提到的任何 BE 端口已被占用，必须在 BE 配置文件中分配有效的替代项。

      ```YAML
      be_port = vvvv                   # 默认：9060
      be_http_port = xxxx              # 默认：8040
      heartbeat_service_port = yyyy    # 默认：9050
      brpc_port = zzzz                 # 默认：8060
      ```

   c. 如果要为集群启用 IP 地址访问，必须在配置文件中添加配置项 `priority_networks`，并为 BE 节点分配一个专用的 IP 地址（CIDR 格式）。如果要为集群启用 FQDN 访问，可以忽略此配置项。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > 您可以在终端中运行 `ifconfig` 查看实例拥有的 IP 地址。

   d. 如果在实例上安装了多个 JDK，并且要使用与环境变量 `JAVA_HOME` 中指定的 JDK 不同的特定 JDK，必须在配置文件中添加配置项 `JAVA_HOME`，指定所选 JDK 安装的路径。

      ```YAML
      # 用所选 JDK 安装的路径替换 <path_to_JDK>。
      JAVA_HOME = <path_to_JDK>
      ```

   有关高级配置项的信息，请参阅[参数配置 - BE 配置项](../administration/BE_configuration.md#be-configuration-items)。

3. 启动 BE 节点。

      ```Bash
      ./be/bin/start_be.sh --daemon
      ```

      > **注意**
      >
      > - 在启用 FQDN 访问的情况下启动 BE 节点之前，请确保已为所有实例在 **/etc/hosts** 中分配了主机名。有关详细信息，请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。
      > - 启动 BE 节点时无需指定 `--host_type` 参数。

4. 检查 BE 日志，验证 BE 节点是否成功启动。

      ```Bash
      cat be/log/be.INFO | grep heartbeat
      ```

      如果日志中出现类似 "I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend" 的记录，则表示 BE 节点已成功启动。

5. 您可以在其他 BE 实例上重复上述操作，启动新的 BE 节点。

> **注意**
>
> 当至少部署并添加 3 个 BE 节点到 StarRocks 集群时，将自动形成一个 BE 高可用集群。

## 步骤 3：（可选）启动 CN 服务

计算节点（CN）是一种无状态计算服务，不维护数据。您可以选择将 CN 节点添加到集群中，为查询提供额外的计算资源。您可以使用 BE 部署文件部署 CN 节点。从 v2.4 开始支持计算节点。


1. 导航到之前准备的[StarRocks BE部署文件](../deployment/prepare_deployment_files.md)目录，并修改CN配置文件**be/conf/cn.conf**。

   a. 如果[环境配置清单](../deployment/environment_configurations.md)中提到的任何CN端口已被占用，则必须在CN配置文件中分配有效的备用端口。

      ```YAML
      be_port = vvvv                   # 默认：9060
      be_http_port = xxxx              # 默认：8040
      heartbeat_service_port = yyyy    # 默认：9050
      brpc_port = zzzz                 # 默认：8060
      ```

   b. 如果要为集群启用IP地址访问，必须在配置文件中添加配置项`priority_networks`，并为CN节点分配一个专用的IP地址（CIDR格式）。如果要为集群启用FQDN访问，则可以忽略此配置项。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > 您可以在终端中运行`ifconfig`来查看实例拥有的IP地址。

   c. 如果实例上安装了多个JDK，并且要使用与环境变量`JAVA_HOME`中指定的JDK不同的特定JDK，则必须在配置文件中添加配置项`JAVA_HOME`，并指定所选JDK的安装路径。

      ```YAML
      # 用所选JDK安装的路径替换<path_to_JDK>。
      JAVA_HOME = <path_to_JDK>
      ```

   有关高级配置项的信息，请参阅[参数配置 - BE配置项](../administration/BE_configuration.md#be-configuration-items)，因为大多数CN参数都是从BE继承而来。

2. 启动CN节点。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

   > **注意**
   >
   > - 在启动启用FQDN访问的CN节点之前，请确保已为**/etc/hosts**中的所有实例分配了主机名。有关详细信息，请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。
   > - 启动CN节点时无需指定参数`--host_type`。

3. 检查CN日志，以验证CN节点是否已成功启动。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   类似“I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050”的日志记录表明CN节点已正确启动。

4. 您可以通过在其他实例上重复上述过程来启动新的CN节点。

## 步骤 4：设置集群

当所有FE、BE节点和CN节点都已成功启动后，即可设置StarRocks集群。

以下过程在MySQL客户端上执行。您必须安装MySQL客户端5.5.0或更高版本。

1. 通过MySQL客户端连接StarRocks。您需要使用初始帐户`root`登录，默认密码为空。

   ```Bash
   # 用Leader FE节点的IP地址（priority_networks）或FQDN替换<fe_address>，并用fe.conf中指定的query_port（默认：9030）替换<query_port>。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下SQL检查Leader FE节点的状态。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   例如：

   ```Plain
   MySQL [(none)]> SHOW PROC '/frontends'\G
   *************************** 1. row ***************************
                Name: x.x.x.x_9010_1686810741121
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: LEADER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1220
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 14:32:28
             Version: 3.0.0-48f4d81
   1 row in set (0.01 sec)
   ```

   - 如果字段`Alive`为`true`，则此FE节点已正确启动并添加到集群中。
   - 如果字段`Role`为`FOLLOWER`，则此FE节点有资格被选为Leader FE节点。
   - 如果字段`Role`为`LEADER`，则此FE节点为Leader FE节点。

3. 将BE节点添加到集群中。

   ```SQL
   -- 用BE节点的IP地址（priority_networks）或FQDN替换<be_address>，并用be.conf中指定的heartbeat_service_port（默认：9050）替换<heartbeat_service_port>。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > 您可以使用上述命令一次添加多个BE节点。每个`<be_address>:<heartbeat_service_port>`对代表一个BE节点。

4. 通过执行以下SQL检查BE节点的状态。

   ```SQL
   SHOW PROC '/backends'\G
   ```

   例如：

   ```Plain
   MySQL [(none)]> SHOW PROC '/backends'\G
   *************************** 1. row ***************************
               BackendId: 10007
                      IP: 172.26.195.67
           HeartbeatPort: 9050
                  BePort: 9060
                HttpPort: 8040
                BrpcPort: 8060
           LastStartTime: 2023-06-15 15:23:08
           LastHeartbeat: 2023-06-15 15:57:30
                   Alive: true
    SystemDecommissioned: false
   ClusterDecommissioned: false
               TabletNum: 30
        DataUsedCapacity: 0.000 
           AvailCapacity: 341.965 GB
           TotalCapacity: 1.968 TB
                 UsedPct: 83.04 %
          MaxDiskUsedPct: 83.04 %
                  ErrMsg: 
                 Version: 3.0.0-48f4d81
                  Status: {"lastSuccessReportTabletsTime":"2023-06-15 15:57:08"}
       DataTotalCapacity: 341.965 GB
             DataUsedPct: 0.00 %
                CpuCores: 16
       NumRunningQueries: 0
              MemUsedPct: 0.01 %
              CpuUsedPct: 0.0 %
   ```

   如果字段`Alive`为`true`，则此BE节点已正确启动并添加到集群中。

5. （可选）将CN节点添加到集群中。

   ```SQL
   -- 用CN节点的IP地址（priority_networks）或FQDN替换<cn_address>，并用cn.conf中指定的heartbeat_service_port（默认：9050）替换<heartbeat_service_port>。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

   > **注意**
   >
   > 您可以使用一个SQL一次添加多个CN节点。每个`<cn_address>:<heartbeat_service_port>`对代表一个CN节点。

6. （可选）通过执行以下SQL检查CN节点的状态。

   ```SQL
   SHOW PROC '/compute_nodes'\G
   ```

   例如：

   ```Plain
   MySQL [(none)]> SHOW PROC '/compute_nodes'\G
   *************************** 1. row ***************************
           ComputeNodeId: 10003
                      IP: x.x.x.x
           HeartbeatPort: 9050
                  BePort: 9060
                HttpPort: 8040
                BrpcPort: 8060
           LastStartTime: 2023-03-13 15:11:13
           LastHeartbeat: 2023-03-13 15:11:13
                   Alive: true
    SystemDecommissioned: false
   ClusterDecommissioned: false
                  ErrMsg: 
                 Version: 2.5.2-c3772fb
   1 row in set (0.00 sec)
   ```

   如果字段`Alive`为`true`，则此CN节点已正确启动并添加到集群中。

   在CN节点正确启动后，如果要在查询期间使用CN节点，请设置系统变量`SET prefer_compute_node = true;`和`SET use_compute_nodes = -1;`。有关详细信息，请参阅[系统变量](../reference/System_variable.md#descriptions-of-variables)。


## 步骤 5：（可选）部署高可用 FE 集群

高可用 FE 集群在 StarRocks 集群中至少需要三个 Follower FE 节点。在 Leader FE 节点成功启动后，您可以启动两个新的 FE 节点来部署高可用 FE 集群。

1. 通过 MySQL 客户端连接 StarRocks。您需要使用初始帐户 `root` 登录，默认密码为空。

   ```Bash
   # 用 Leader FE 节点的 IP 地址（priority_networks）或 FQDN 替换 <fe_address>，用 FE 配置文件 fe.conf 中指定的 query_port（默认值：9030）替换 <query_port>。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下 SQL 将新的 Follower FE 节点添加到集群中。

   ```SQL
   -- 用新的 Follower FE 节点的 IP 地址（priority_networks）或 FQDN 替换 <fe_address>，用 FE 配置文件 fe.conf 中指定的 edit_log_port（默认值：9010）替换 <edit_log_port>。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

   > **注意**
   >
   > - 每次只能使用上述命令添加单个 Follower FE 节点。
   > - 如果要添加 Observer FE 节点，请执行 `ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"=`。有关详细说明，请参阅 [ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)。

3. 在新的 FE 实例上启动终端，创建一个专用目录用于存储元数据，切换到存放 StarRocks FE 部署文件的目录，并修改 FE 配置文件 **fe/conf/fe.conf**。有关更多说明，请参见[步骤 1：启动 Leader FE 节点](#step-1-start-the-leader-fe-node)。基本上，您可以重复步骤 1 中的所有步骤，**除了用于启动 FE 节点的命令**。

   配置 Follower FE 节点后，执行以下 SQL 为 Follower FE 节点分配一个辅助节点，并启动 Follower FE 节点。

   > **注意**
   >
   > 在向集群中添加新的 Follower FE 节点时，必须为新的 Follower FE 节点分配一个辅助节点（实质上是现有的 Follower FE 节点）以同步元数据。

   - 要启动具有 IP 地址访问权限的新 FE 节点，请运行以下命令启动 FE 节点：

     ```Bash
     # 用 Leader FE 节点的 IP 地址（priority_networks）替换 <helper_fe_ip>，用 Leader FE 节点的 edit_log_port（默认值：9010）替换 <helper_edit_log_port>。
     ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     请注意，首次启动节点时只需指定参数 `--helper` 一次。

   - 要启动具有 FQDN 访问权限的新 FE 节点，请运行以下命令启动 FE 节点：

     ```Bash
     # 用 Leader FE 节点的 FQDN 替换 <helper_fqdn>，用 Leader FE 节点的 edit_log_port（默认值：9010）替换 <helper_edit_log_port>。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     请注意，首次启动节点时只需指定参数 `--helper` 和 `--host_type` 一次。

4. 检查 FE 日志，验证 FE 节点是否成功启动。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   如果出现类似“2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020.”的日志记录，则表示 FE 节点已成功启动。

5. 重复上述步骤 2、3 和 4，直到所有新的 Follower FE 节点都正确启动，然后通过从 MySQL 客户端执行以下 SQL 来检查 FE 节点的状态：

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   示例：

   ```Plain
   MySQL [(none)]> SHOW PROC '/frontends'\G
   *************************** 1. row ***************************
                Name: x.x.x.x_9010_1686810741121
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: LEADER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1220
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 14:32:28
             Version: 3.0.0-48f4d81
   *************************** 2. row ***************************
                Name: x.x.x.x_9010_1686814080597
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: FOLLOWER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1219
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 15:38:53
             Version: 3.0.0-48f4d81
   *************************** 3. row ***************************
                Name: x.x.x.x_9010_1686814090833
                  IP: x.x.x.x
         EditLogPort: 9010
            HttpPort: 8030
           QueryPort: 9030
             RpcPort: 9020
                Role: FOLLOWER
           ClusterId: 919351034
                Join: true
               Alive: true
   ReplayedJournalId: 1219
       LastHeartbeat: 2023-06-15 15:39:04
            IsHelper: true
              ErrMsg: 
           StartTime: 2023-06-15 15:37:52
             Version: 3.0.0-48f4d81
   3 rows in set (0.02 sec)
   ```

   - 如果 `Alive` 字段为 `true`，则表示此 FE 节点已成功启动并添加到集群中。
   - 如果 `Role` 字段为 `FOLLOWER`，则表示此 FE 节点有资格成为 Leader FE 节点。
   - 如果 `Role` 字段为 `LEADER`，则表示此 FE 节点是 Leader FE 节点。

## 停止 StarRocks 集群

您可以通过在相应的实例上运行以下命令来停止 StarRocks 集群。

- 停止 FE 节点。

  ```Bash
  ./fe/bin/stop_fe.sh --daemon
  ```

- 停止 BE 节点。

  ```Bash
  ./be/bin/stop_be.sh --daemon
  ```

- 停止 CN 节点。

  ```Bash
  ./be/bin/stop_cn.sh --daemon
  ```

## 故障排除

尝试以下步骤来识别启动 FE、BE 或 CN 节点时发生的错误：

- 如果 FE 节点未能正常启动，您可以通过检查其日志文件 **fe/log/fe.warn.log** 来识别问题。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  确定并解决问题后，您需要先终止当前的 FE 进程，删除现有的 **meta** 目录，创建新的元数据存储目录，然后使用正确的配置重新启动 FE 节点。

- 如果 BE 节点未能正常启动，您可以通过检查其日志文件 **be/log/be.WARNING** 来识别问题。

  ```Bash
  cat be/log/be.WARNING
  ```

  确定并解决问题后，您需要先终止现有的 BE 进程，删除现有的 **storage** 目录，创建新的数据存储目录，然后使用正确的配置重新启动 BE 节点。

- 如果 CN 节点未能正常启动，您可以通过检查其日志文件 **be/log/cn.WARNING** 来识别问题。

  ```Bash
  cat be/log/cn.WARNING
  ```

  在确定并解决问题后，您必须首先终止现有的 CN 进程，然后使用正确的配置重新启动 CN 节点。

## 接下来该做什么

在部署完您的 StarRocks 集群之后，您可以转到 [部署后设置](../deployment/post_deployment_setup.md) 了解有关初始管理措施的说明。