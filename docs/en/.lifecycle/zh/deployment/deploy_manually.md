---
displayed_sidebar: English
---

# 手动部署 StarRocks

本文介绍如何手动部署无共享架构的 StarRocks（其中 BE 负责存储和计算）。其他安装方式请参见[部署概述](../deployment/deployment_overview.md)。

要部署一个共享数据的 StarRocks 集群（存储与计算解耦），请参阅[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)。

## 步骤 1：启动 Leader FE 节点

以下流程在 FE 实例上执行。

1. 创建一个用于元数据存储的专用目录。我们建议将元数据存储在与 FE 部署文件分开的目录中。确保该目录存在并且您有写入权限。

   ```YAML
   # 用您想要创建的元数据目录替换 <meta_dir>。
   mkdir -p <meta_dir>
   ```

2. 进入您之前准备的 [StarRocks FE 部署文件](../deployment/prepare_deployment_files.md) 所在目录，修改 FE 配置文件 **fe/conf/fe.conf**。

   a. 在配置项 `meta_dir` 中指定元数据目录。

   ```YAML
   # 用您已创建的元数据目录替换 <meta_dir>。
   meta_dir = <meta_dir>
   ```

   b. 如果 [环境配置清单](../deployment/environment_configurations.md#fe-ports) 中提到的任何 FE 端口被占用，您必须在 FE 配置文件中分配有效的替代端口。

   ```YAML
   http_port = aaaa        # 默认值: 8030
   rpc_port = bbbb         # 默认值: 9020
   query_port = cccc       # 默认值: 9030
   edit_log_port = dddd    # 默认值: 9010
   ```

      > **警告**
      > 如果您要在集群中部署多个 FE 节点，必须为每个 FE 节点分配相同的 `http_port`。

   c. 如果您想为集群启用 IP 地址访问，必须在配置文件中添加配置项 `priority_networks` 并为 FE 节点分配专用 IP 地址（CIDR 格式）。如果您想为集群启用 [FQDN 访问](../administration/enable_fqdn.md)，可以忽略此配置项。

   ```YAML
   priority_networks = x.x.x.x/x
   ```

      > **注意**
      > 您可以在终端中运行 `ifconfig` 来查看实例拥有的 IP 地址。

   d. 如果实例上安装了多个 JDK，并且您想要使用与环境变量 `JAVA_HOME` 指定的 JDK 不同的特定 JDK，您必须在配置文件中通过添加配置项 `JAVA_HOME` 来指定所选 JDK 的安装路径。

   ```YAML
   # 用所选 JDK 的安装路径替换 <path_to_JDK>。
   JAVA_HOME = <path_to_JDK>
   ```

   f. 有关高级配置项的信息，请参阅[参数配置 - FE 配置项](../administration/FE_configuration.md#fe-configuration-items)。

3. 启动 FE 节点。

-    要为集群启用 IP 地址访问，请运行以下命令来启动 FE 节点：

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

-    要为集群启用 FQDN 访问，请运行以下命令来启动 FE 节点：

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     请注意，首次启动节点时只需指定参数 `--host_type` 一次。

          > **警告**
          > 在启用 FQDN 访问的情况下启动 FE 节点之前，请确保已在 **/etc/hosts** 中为所有实例分配了主机名。有关详细信息，请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。

4. 检查 FE 日志，验证 FE 节点是否启动成功。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   日志记录类似于 "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift 服务器已在端口 9020 上启动。" 表明 FE 节点已正确启动。

## 步骤 2：启动 BE 服务

以下过程在 BE 实例上执行。

1. 创建一个专用于数据存储的目录。我们建议将数据存储在与 BE 部署目录分开的目录中。确保该目录存在并且您有写入权限。

   ```YAML
   # 用您想要创建的数据存储目录替换 <storage_root_path>。
   mkdir -p <storage_root_path>
   ```

2. 进入您之前准备的 [StarRocks BE 部署文件](../deployment/prepare_deployment_files.md) 所在目录，修改 BE 配置文件 **be/conf/be.conf**。

   a. 在配置项 `storage_root_path` 中指定数据目录。

   ```YAML
   # 用您已创建的数据目录替换 <storage_root_path>。
   storage_root_path = <storage_root_path>
   ```

   b. 如果 [环境配置清单](../deployment/environment_configurations.md#be-ports) 中提到的任何 BE 端口被占用，您必须在 BE 配置文件中分配有效的替代端口。

   ```YAML
   be_port = vvvv                   # 默认值: 9060
   be_http_port = xxxx              # 默认值: 8040
   heartbeat_service_port = yyyy    # 默认值: 9050
   brpc_port = zzzz                 # 默认值: 8060
   ```

   c. 如果您希望集群开启 IP 地址访问功能，需要在配置文件中添加 `priority_networks` 配置项，并为 BE 节点分配专用 IP 地址（CIDR 格式）。如果您想为集群启用 FQDN 访问，可以忽略此配置项。

   ```YAML
   priority_networks = x.x.x.x/x
   ```

      > **注意**
      > 您可以在终端中运行 `ifconfig` 来查看实例拥有的 IP 地址。

   d. 如果实例上安装了多个 JDK，并且您想要使用与环境变量 `JAVA_HOME` 指定的 JDK 不同的特定 JDK，您必须在配置文件中通过添加配置项 `JAVA_HOME` 来指定所选 JDK 的安装路径。

   ```YAML
   # 用所选 JDK 的安装路径替换 <path_to_JDK>。
   JAVA_HOME = <path_to_JDK>
   ```

   有关高级配置项的信息，请参阅[参数配置 - BE 配置项](../administration/BE_configuration.md#be-configuration-items)。

3. 启动 BE 节点。

   ```Bash
   ./be/bin/start_be.sh --daemon
   ```

      > **警告**
   - 在启用 FQDN 访问的情况下启动 BE 节点之前，请确保已在 **/etc/hosts** 中为所有实例分配了主机名。有关详细信息，请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。
   - 启动 BE 节点时不需要指定参数 `--host_type`。

4. 检查 BE 日志，验证 BE 节点是否启动成功。

   ```Bash
   cat be/log/be.INFO | grep heartbeat
   ```

   日志记录如 "I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend" 表明 BE 节点已正确启动。

5. 您可以通过在其他 BE 实例上重复上述过程来启动新的 BE 节点。

> **注意**
> 当至少部署三个 BE 节点并将其添加到 StarRocks 集群时，会自动形成高可用性 BE 集群。

## 步骤 3：（可选）启动 CN 服务

计算节点（CN）是一种无状态计算服务，本身不维护数据。您可以选择将 CN 节点添加到集群中，以为查询提供额外的计算资源。您可以使用 BE 部署文件来部署 CN 节点。自 v2.4 版本起支持计算节点。

1. 进入您之前准备的 [StarRocks BE 部署文件](../deployment/prepare_deployment_files.md) 所在目录，修改 CN 配置文件 **be/conf/cn.conf**。

   a. 如果 [环境配置清单](../deployment/environment_configurations.md) 中提到的任何 CN 端口被占用，您必须在 CN 配置文件中分配有效的替代端口。

   ```YAML
   be_port = vvvv                   # 默认值: 9060
   be_http_port = xxxx              # 默认值: 8040
   heartbeat_service_port = yyyy    # 默认值: 9050
   brpc_port = zzzz                 # 默认值: 8060
   ```

   b. 如果您希望集群开启 IP 地址访问功能，需要在配置文件中添加 `priority_networks` 配置项，并为 CN 节点分配专用 IP 地址（CIDR 格式）。如果您想为集群启用 FQDN 访问，可以忽略此配置项。

   ```YAML
   priority_networks = x.x.x.x/x
   ```

      > **注意**
      > 您可以在终端中运行 `ifconfig` 来查看实例拥有的 IP 地址。

   c. 如果实例上安装了多个 JDK，并且您想要使用与环境变量 `JAVA_HOME` 指定的 JDK 不同的特定 JDK，您必须在配置文件中通过添加配置项 `JAVA_HOME` 来指定所选 JDK 的安装路径。

   ```YAML
   # 用所选 JDK 的安装路径替换 <path_to_JDK>。
   JAVA_HOME = <path_to_JDK>
   ```

   有关高级配置项的信息，请参阅[参数配置 - BE 配置项](../administration/BE_configuration.md#be-configuration-items)，因为大部分 CN 的参数都是继承自 BE。

2. 启动 CN 节点。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

      > **警告**
   - 在启用 FQDN 访问的情况下启动 CN 节点之前，请确保已在 **/etc/hosts** 中为所有实例分配了主机名。有关详细信息，请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。
   - 启动 CN 节点时不需要指定参数 `--host_type`。

3. 检查 CN 日志，验证 CN 节点是否启动成功。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   日志记录如 "I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening on port 9050" 表明 CN 节点已正确启动。

4. 您可以通过在其他实例上重复上述过程来启动新的 CN 节点。

## 步骤 4：设置集群

当所有 FE、BE 和 CN 节点正常启动后，您可以开始设置 StarRocks 集群。

以下过程在 MySQL 客户端上执行。您必须安装 MySQL 客户端 5.5.0 或更高版本。

1. 通过 MySQL 客户端连接到 StarRocks。您需要使用初始账户 `root` 登录，密码默认为空。

   ```Bash
   # 用 Leader FE 节点的 IP 地址（priority_networks）或 FQDN 替换 <fe_address>，
   # 并用您在 fe.conf 中指定的 query_port 替换 <query_port>（默认值：9030）。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下 SQL 检查 Leader FE 节点的状态。

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
```
- 如果`Alive`字段为`true`，则该FE节点已正确启动并添加到集群中。
   - 如果`Role`字段为`FOLLOWER`，则该FE节点有资格被选举为Leader FE节点。
   - 如果`Role`字段为`LEADER`，则该FE节点为Leader FE节点。

3. 将BE节点添加到集群。

   ```SQL
   -- 将<be_address>替换为BE节点的IP地址（priority_networks）
   -- 或FQDN，并将<heartbeat_service_port>替换为
   -- 在be.conf中指定的heartbeat_service_port（默认：9050）。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

      > **注意**
      > 您可以使用上述命令一次添加多个BE节点。每对`<be_address>:<heartbeat_service_port>`代表一个BE节点。

4. 通过执行以下SQL检查BE节点的状态。

   ```SQL
   SHOW PROC '/backends'\G
   ```

   示例：

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

   如果`Alive`字段为`true`，则该BE节点已正确启动并添加到集群中。

5. （可选）向集群添加CN节点。

   ```SQL
   -- 将<cn_address>替换为CN节点的IP地址（priority_networks）
   -- 或FQDN，并将<heartbeat_service_port>替换为
   -- 在cn.conf中指定的heartbeat_service_port（默认：9050）。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

      > **注意**
      > 您可以使用一个SQL命令添加多个CN节点。每对`<cn_address>:<heartbeat_service_port>`代表一个CN节点。

6. （可选）执行以下SQL检查CN节点的状态。

   ```SQL
   SHOW PROC '/compute_nodes'\G
   ```

   示例：

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

   如果`Alive`字段为`true`，则该CN节点已正确启动并添加到集群中。

   CN节点正常启动后，如果您想在查询时使用CN节点，请设置系统变量 `SET prefer_compute_node = true;` 并设置 `SET use_compute_nodes = -1;`。有关详细信息，请参阅[系统变量](../reference/System_variable.md#descriptions-of-variables)。

## 步骤5：（可选）部署高可用FE集群

高可用性FE集群至少需要StarRocks集群中的三个Follower FE节点。在Leader FE节点成功启动后，您可以启动两个新的FE节点来部署高可用FE集群。

1. 通过MySQL客户端连接到StarRocks。您需要使用初始账户`root`登录，密码默认为空。

   ```Bash
   # 将<fe_address>替换为Leader FE节点的IP地址（priority_networks）或FQDN，
   # 并将<query_port>（默认：9030）替换为您在fe.conf中指定的query_port。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下SQL将新的Follower FE节点添加到集群中。

   ```SQL
   -- 将<fe_address>替换为新的Follower FE节点的IP地址（priority_networks）
   -- 或FQDN，并将<edit_log_port>替换为您在fe.conf中指定的edit_log_port（默认：9010）。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

      > **注意**
   - 您可以使用上述命令每次添加一个Follower FE节点。
   - 如果您想添加Observer FE节点，请执行 `ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>";`。有关详细说明，请参阅[ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)。

3. 在新的FE实例上启动终端，创建元数据存储的专用目录，导航到存储StarRocks FE部署文件的目录，并修改FE配置文件**fe/conf/fe.conf**。有关更多说明，请参阅[步骤1：启动Leader FE节点](#step-1-start-the-leader-fe-node)。基本上，您可以重复步骤1中的过程，**除了用于启动FE节点的命令**。

   配置完Follower FE节点后，执行以下SQL为Follower FE节点分配一个helper节点，并启动Follower FE节点。

      > **注意**
      > 当向集群中添加新的Follower FE节点时，您必须为新的Follower FE节点分配一个helper节点（本质上是现有的Follower FE节点），以同步元数据。

-    要启动具有IP地址访问权限的新FE节点，请运行以下命令来启动FE节点：

     ```Bash
     # 将<helper_fe_ip>替换为Leader FE节点的IP地址（priority_networks），
     # 并将<helper_edit_log_port>（默认：9010）替换为
     # Leader FE节点的edit_log_port。
     ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     请注意，首次启动节点时只需指定`--helper`参数一次。

-    要启动具有FQDN访问权限的新FE节点，请运行以下命令来启动FE节点：

     ```Bash
     # 将<helper_fqdn>替换为Leader FE节点的FQDN，
     # 并将<helper_edit_log_port>（默认：9010）替换为
     # Leader FE节点的edit_log_port。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     请注意，首次启动节点时只需指定`--helper`和`--host_type`参数一次。

4. 检查FE日志，验证FE节点是否启动成功。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   类似“2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift服务器已使用端口9020启动。”的日志记录表明FE节点启动正常。

5. 重复上述步骤2、3、4，直到所有新的Follower FE节点正常启动，然后在MySQL客户端执行以下SQL检查FE节点的状态：

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

   - 如果`Alive`字段为`true`，则该FE节点已正确启动并添加到集群中。
   - 如果`Role`字段为`FOLLOWER`，则该FE节点有资格被选举为Leader FE节点。
   - 如果`Role`字段为`LEADER`，则该FE节点为Leader FE节点。

## 停止StarRocks集群

您可以通过在相应实例上运行以下命令来停止StarRocks集群。

- 停止FE节点。

  ```Bash
  ./fe/bin/stop_fe.sh --daemon
  ```

- 停止BE节点。

  ```Bash
  ./be/bin/stop_be.sh --daemon
  ```

- 停止CN节点。

  ```Bash
  ./be/bin/stop_cn.sh --daemon
  ```

## 故障排除

尝试以下步骤来确定启动FE、BE或CN节点时发生的错误：

- 如果FE节点没有正确启动，您可以通过检查其日志在**fe/log/fe.warn.log**中来确定问题。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  确定并解决问题后，您必须首先终止当前的FE进程，删除现有的**meta**目录，创建新的元数据存储目录，然后使用正确的配置重新启动FE节点。

- 如果BE节点启动不正常，您可以通过查看**be/log/be.WARNING**中的日志来确定问题。

  ```Bash
  cat be/log/be.WARNING
  ```

  确定并解决问题后，您必须首先终止现有的BE进程，删除现有的**storage**目录，创建新的数据存储目录，然后使用正确的配置重新启动BE节点。

- 如果CN节点没有正确启动，您可以通过检查其日志在**be/log/cn.WARNING**中来确定问题。

  ```Bash
  cat be/log/cn.WARNING
  ```
```markdown
确定并解决问题后，必须首先终止当前的 CN 进程，然后使用正确的配置重新启动 CN 节点。

## 下一步该做什么

部署 StarRocks 集群后，您可以继续阅读[部署后设置](../deployment/post_deployment_setup.md)以了解初始管理措施的指导。