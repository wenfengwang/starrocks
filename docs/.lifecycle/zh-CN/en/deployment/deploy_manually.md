---
displayed_sidebar: "Chinese"
---

# 手动部署 StarRocks

本主题介绍如何手动部署共享无存储 StarRocks（其中 BE 负责存储和计算）。有关其他安装模式，请参见[部署概述](../deployment/deployment_overview.md)。

要部署共享数据 StarRocks 集群（存储和计算分离），请参见[部署和使用共享数据 StarRocks](../deployment/shared_data/s3.md)。

## 步骤 1：启动主 FE 节点

以下过程在 FE 实例上执行。

1. 为元数据存储创建专用目录。我们建议将元数据存储在与 FE 部署文件分开的目录中。确保该目录存在，并您具有写访问权限。

   ```YAML
   # 用所需的元数据目录替换 <meta_dir>。
   mkdir -p <meta_dir>
   ```

2. 转到存储[StarRocks FE 部署文件](../deployment/prepare_deployment_files.md)的目录，并修改 FE 配置文件 **fe/conf/fe.conf**。

   a. 在配置项 `meta_dir` 中指定元数据目录。

      ```YAML
      # 用您创建的元数据目录替换 <meta_dir>。
      meta_dir = <meta_dir>
      ```

   b. 如果[环境配置清单](../deployment/environment_configurations.md#fe-ports)中提到的任何 FE 端口已被占用，必须在 FE 配置文件中分配有效的替代端口。

      ```YAML
      http_port = aaaa        # 默认值：8030
      rpc_port = bbbb         # 默认值：9020
      query_port = cccc       # 默认值：9030
      edit_log_port = dddd    # 默认值：9010
      ```

      > **警告**
      >
      > 如果要在集群中部署多个 FE 节点，必须为每个 FE 节点分配相同的 `http_port`。

   c. 如果要为集群启用 IP 地址访问，必须在配置文件中添加配置项 `priority_networks` 并为 FE 节点分配一个专用 IP 地址（CIDR 格式）。如果要为集群启用[FQDN 访问](../administration/enable_fqdn.md)，可以忽略此配置项。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > 您可以在终端中运行 `ifconfig` 以查看实例拥有的 IP 地址（们）。

   d. 如果实例上安装了多个 JDK，并且您希望使用与环境变量 `JAVA_HOME` 中指定的 JDK 不同的特定 JDK，则必须在配置文件中通过添加配置项 `JAVA_HOME` 指定所选 JDK 安装的路径。

      ```YAML
      # 用所选 JDK 安装路径替换 <path_to_JDK>。
      JAVA_HOME = <path_to_JDK>
      ```

   f. 有关高级配置项的信息，请参见[参数配置 - FE 配置项](../administration/Configuration.md#fe-configuration-items)

3. 启动 FE 节点。

   - 若要为集群启用 IP 地址访问，请运行以下命令启动 FE 节点：

     ```Bash
     ./fe/bin/start_fe.sh --daemon
     ```

   - 若要为集群启用 FQDN 访问，请运行以下命令启动 FE 节点：

     ```Bash
     ./fe/bin/start_fe.sh --host_type FQDN --daemon
     ```

     注意，当您首次启动节点时，只需指定参数 `--host_type` 一次。

     > **警告**
     >
     > 在启用 FQDN 访问后启动 FE 节点之前，请确保您已为**/etc/hosts**中的所有实例分配了主机名。有关更多信息，请参见[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。

4. 检查 FE 日志，以验证 FE 节点是否已成功启动。

   ```Bash
   cat fe/log/fe.log | grep thrift
   ```

   类似 "2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020." 的日志记录表示 FE 节点已正确启动。

## 步骤 2：启动 BE 服务

以下过程在 BE 实例上执行。

1. 为数据存储创建专用目录。我们建议将数据存储在与 BE 部署目录分开的目录中。确保该目录存在，并您具有写访问权限。

   ```YAML
   # 用所需的数据存储目录替换 <storage_root_path>。
   mkdir -p <storage_root_path>
   ```

2. 转到存储[StarRocks BE 部署文件](../deployment/prepare_deployment_files.md)的目录，并修改 BE 配置文件 **be/conf/be.conf**。

   a. 在配置项 `storage_root_path` 中指定数据目录。

      ```YAML
      # 用您创建的数据目录替换 <storage_root_path>。
      storage_root_path = <storage_root_path>
      ```

   b. 如果[环境配置清单](../deployment/environment_configurations.md#be-ports)中提到的任何 BE 端口已被占用，必须在 BE 配置文件中分配有效的替代端口。

      ```YAML
      be_port = vvvv                   # 默认值：9060
      be_http_port = xxxx              # 默认值：8040
      heartbeat_service_port = yyyy    # 默认值：9050
      brpc_port = zzzz                 # 默认值：8060
      ```

   c. 如果要为集群启用 IP 地址访问，必须在配置文件中添加配置项 `priority_networks` 并为 BE 节点分配一个专用 IP 地址（CIDR 格式）。如果要为集群启用 FQDN 访问，可以忽略此配置项。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > 您可以在终端中运行 `ifconfig` 以查看实例拥有的 IP 地址（们）。

   d. 如果实例上安装了多个 JDK，并且您希望使用与环境变量 `JAVA_HOME` 中指定的 JDK 不同的特定 JDK，则必须在配置文件中通过添加配置项 `JAVA_HOME` 指定所选 JDK 安装的路径。

      ```YAML
      # 用所选 JDK 安装路径替换 <path_to_JDK>。
      JAVA_HOME = <path_to_JDK>
      ```

   有关高级配置项的信息，请参见[参数配置 - BE 配置项](../administration/Configuration.md#be-configuration-items)。

3. 启动 BE 节点。

      ```Bash
      ./be/bin/start_be.sh --daemon
      ```

      > **警告**
      >
      > - 在启用 FQDN 访问后启动 BE 节点之前，请确保您已为**/etc/hosts**中的所有实例分配了主机名。有关更多信息，请参见[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)。
      > - 在启动 BE 节点时，不需要指定 `--host_type` 参数。

4. 检查 BE 日志，以验证 BE 节点是否已成功启动。

      ```Bash
      cat be/log/be.INFO | grep heartbeat
      ```

      类似 "I0810 16:18:44.487284 3310141 task_worker_pool.cpp:1387] Waiting to receive first heartbeat from frontend" 的日志记录表示 BE 节点已正确启动。

5. 您可以通过在其他 BE 实例上重复上述过程来启动新的 BE 节点。

> **注意**
>
> 至少部署并添加到 StarRocks 集群中的三个 BE 节点后，将自动形成一个高可用的 BE 集群。

## 步骤 3：（可选）启动 CN 服务

计算节点（CN）是无状态计算服务，不维护数据本身。您可以选择将 CN 节点添加到集群，以为查询提供额外的计算资源。可以使用 BE 部署文件部署 CN 节点。从 v2.4 版本开始支持计算节点。

1. 转到存储[StarRocks BE 部署文件](../deployment/prepare_deployment_files.md)的目录，并修改 CN 配置文件 **be/conf/cn.conf**。

   a. 如果[环境配置清单](../deployment/environment_configurations.md)中提到的任何 CN 端口已被占用，必须在 CN 配置文件中分配有效的替代端口。

      ```YAML
      be_port = vvvv                   # 默认值：9060
      be_http_port = xxxx              # 默认值：8040
      heartbeat_service_port = yyyy    # 默认值：9050
      brpc_port = zzzz                 # 默认值：8060
      ```

b. 如果您想要为您的集群启用IP地址访问，您必须在配置文件中添加配置项`priority_networks`并将专用IP地址（CIDR格式）分配给CN节点。如果您希望为集群启用FQDN访问，您可以忽略此配置项。

      ```YAML
      priority_networks = x.x.x.x/x
      ```

      > **注意**
      >
      > 您可以在终端中运行`ifconfig`来查看实例拥有的IP地址。

c. 如果您的实例上安装了多个JDK，并且您希望使用与环境变量`JAVA_HOME`中指定的不同的特定JDK，则必须通过在配置文件中添加配置项`JAVA_HOME`来指定所选JDK安装的路径。

      ```YAML
      # 用所选JDK安装的路径替换<path_to_JDK>。
      JAVA_HOME = <path_to_JDK>
      ```

有关高级配置项的信息，请参阅[参数配置 - BE配置项](../administration/Configuration.md#be-configuration-items)，因为大多数CN参数都是从BE继承的。

2. 启动CN节点。

   ```Bash
   ./be/bin/start_cn.sh --daemon
   ```

   > **注意**
   >
   > - 在启用FQDN访问的情况下启动CN节点之前，请确保您已为**/etc/hosts**中的所有实例分配了主机名。请参阅[环境配置清单 - 主机名](../deployment/environment_configurations.md#hostnames)了解更多信息。
   > - 在启动CN节点时，您无需指定参数`--host_type`。

3. 检查CN日志，以验证CN节点是否已成功启动。

   ```Bash
   cat be/log/cn.INFO | grep heartbeat
   ```

   类似"I0313 15:03:45.820030 412450 thrift_server.cpp:375] heartbeat has started listening port on 9050"的日志记录表明CN节点已成功启动。

4. 您可以通过在其他实例上重复上述过程来启动新的CN节点。

## 步骤 4：设置集群

在所有FE、BE节点和CN节点都正确启动后，您可以设置StarRocks集群。

以下过程在MySQL客户端上执行。您必须安装MySQL客户端5.5.0或更高版本。

1. 通过您的MySQL客户端连接到StarRocks。您需要使用初始帐户`root`登录，默认情况下密码为空。

   ```Bash
   # 用Leader FE节点的IP地址（priority_networks）或FQDN替换<fe_address>，将查询端口（默认值：9030）替换为您在fe.conf中指定的query_port。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下SQL来检查Leader FE节点的状态。

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
   1 row in set (0.01 sec)
   ```

   - 如果`Alive`字段的值为`true`，则该FE节点已正确启动并已添加到集群中。
   - 如果`Role`字段的值为`FOLLOWER`，则该FE节点有资格被选举为Leader FE节点。
   - 如果`Role`字段的值为`LEADER`，则该FE节点是Leader FE节点。

3. 将BE节点添加到集群。

   ```SQL
   -- 用BE节点的IP地址（priority_networks）或FQDN替换<be_address>，用在be.conf中指定的heartbeat_service_port（默认值：9050）替换<heartbeat_service_port>。
   ALTER SYSTEM ADD BACKEND "<be_address>:<heartbeat_service_port>";
   ```

   > **注意**

   >
   > 您可以使用上述命令一次添加多个BE节点。每个`<be_address>:<heartbeat_service_port>`对代表一个BE节点。


4. 通过执行以下SQL来检查BE节点的状态。

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

   如果`Alive`字段的值为`true`，则该BE节点已正确启动并已添加到集群中。

5. （可选）将CN节点添加到集群。

   ```SQL
   -- 用CN节点的IP地址（priority_networks）或FQDN替换<cn_address>，用在cn.conf中指定的heartbeat_service_port（默认值：9050）替换<heartbeat_service_port>。
   ALTER SYSTEM ADD COMPUTE NODE "<cn_address>:<heartbeat_service_port>";
   ```

   > **注意**

   >
   > 您可以使用上述命令一次添加多个CN节点。每个`<cn_address>:<heartbeat_service_port>`对代表一个CN节点。


6. （可选）通过执行以下SQL来检查CN节点的状态。

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

   如果`Alive`字段的值为`true`，则该CN节点已正确启动并已添加到集群中。

   在CN正确启动并且您要在查询期间使用CN时，请设置系统变量`SET prefer_compute_node = true;`和`SET use_compute_nodes = -1;`。有关更多信息，请参阅[系统变量](../reference/System_variable.md#descriptions-of-variables)。

## 步骤 5：（可选）部署高可用FE集群

高可用FE集群需要StarRocks集群中至少有三个Follower FE节点。成功启动Leader FE节点后，您可以启动两个新的FE节点来部署高可用FE集群。

1. 通过您的MySQL客户端连接到StarRocks。您需要使用初始帐户`root`登录，默认情况下密码为空。

   ```Bash
   # 用Leader FE节点的IP地址（priority_networks）或FQDN替换<fe_address>，将查询端口（默认值：9030）替换为您在fe.conf中指定的query_port。
   ```

```markdown
   mysql -h <fe_address> -P<query_port> -uroot

   ```

2. 通过执行以下SQL将新的Follower前端节点添加到集群中。

   ```SQL
   -- 使用下面的SQL语句将<fe_address>替换为新的Follower前端节点的IP地址（priority_networks）或FQDN，并将<edit_log_port>替换为您在fe.conf中指定的edit_log_port（默认值为：9010）。
   ALTER SYSTEM ADD FOLLOWER "<fe2_address>:<edit_log_port>";
   ```

   > **注意**
   >
   > - 每次只能使用上述命令添加一个单个Follower前端节点。
   > - 如果要添加Observer前端节点，请执行`ALTER SYSTEM ADD OBSERVER "<fe_address>:<edit_log_port>"=`。有关详细说明，请参阅[ALTER SYSTEM - FE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)。

3. 在新的前端实例上启动终端，为元数据存储创建一个专用目录，转到存储StarRocks前端部署文件的目录，并修改前端配置文件**fe/conf/fe.conf**。有关更多说明，请参阅[Step 1: Start the Leader FE node](#step-1-start-the-leader-fe-node)。基本上，您可以重复执行Step 1中的流程**除了用于启动FE节点的命令**。

   在配置好Follower前端节点之后，执行以下SQL以为Follower前端节点分配帮助节点并启动Follower前端节点。

   > **注意**
   >
   > 在向集群添加新的Follower前端节点时，必须为新的Follower前端节点分配一个帮助节点（实质上是现有的Follower前端节点）来同步元数据。

   - 要使用IP地址访问启动新的FE节点，请运行以下命令以启动FE节点：

     ```Bash
     # 将<helper_fe_ip>替换为主Leader FE节点的IP地址（priority_networks），将<helper_edit_log_port>（默认值为：9010）替换为主Leader FE节点的edit_log_port。
     ./fe/bin/start_fe.sh --helper <helper_fe_ip>:<helper_edit_log_port> --daemon
     ```

     请注意，第一次启动节点时，只需要指定一次参数`--helper`。

   - 要使用FQDN访问启动新的FE节点，请运行以下命令以启动FE节点：

     ```Bash

     # 将<helper_fqdn>替换为主Leader FE节点的FQDN，将<helper_edit_log_port>（默认值为：9010）替换为主Leader FE节点的edit_log_port。
     ./fe/bin/start_fe.sh --helper <helper_fqdn>:<helper_edit_log_port> \
           --host_type FQDN --daemon
     ```

     请注意，第一次启动节点时，只需要指定参数`--helper`和`--host_type`一次。

4. 检查FE日志以验证FE节点是否已成功启动。

   ```Bash

   cat fe/log/fe.log | grep thrift
   ```


   类似于“2022-08-10 16:12:29,911 INFO (UNKNOWN x.x.x.x_9010_1660119137253(-1)|1) [FeServer.start():52] thrift server started with port 9020.”的日志记录表明FE节点已成功启动。

5. 重复执行上述步骤2、3和4，直到所有新的Follower前端节点正确启动，然后通过从MySQL客户端执行以下SQL来检查FE节点的状态：

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

   - 如果字段`Alive`是`true`，表示此FE节点已正确启动并添加到集群中。
   - 如果字段`Role`是`FOLLOWER`，表示此FE节点有资格被选为Leader FE节点。
   - 如果字段`Role`是`LEADER`，表示此FE节点是Leader FE节点。

## 停止StarRocks集群

您可以通过在相应实例上运行以下命令来停止StarRocks集群。

- 停止一个FE节点。

  ```Bash
  ./fe/bin/stop_fe.sh --daemon
  ```

- 停止一个BE节点。

  ```Bash
  ./be/bin/stop_be.sh --daemon
  ```

- 停止一个CN节点。

  ```Bash
  ./be/bin/stop_cn.sh --daemon
  ```

## 故障排除

尝试以下步骤来识别启动FE、BE或CN节点时出现的错误：

- 如果一个FE节点未能正确启动，您可以通过检查其位于**fe/log/fe.warn.log**中的日志来识别问题。

  ```Bash
  cat fe/log/fe.warn.log
  ```

  识别并解决问题后，您必须首先终止当前的FE进程，删除现有的**meta**目录，创建一个新的元数据存储目录，然后使用正确的配置重新启动FE节点。

- 如果一个BE节点未能正确启动，您可以通过检查其位于**be/log/be.WARNING**中的日志来识别问题。

  ```Bash
  cat be/log/be.WARNING
  ```

  识别并解决问题后，您必须首先终止现有的BE进程，删除现有的**storage**目录，创建一个新的数据存储目录，然后使用正确的配置重新启动BE节点。

- 如果一个CN节点未能正确启动，您可以通过检查其位于**be/log/cn.WARNING**中的日志来识别问题。

  ```Bash
  cat be/log/cn.WARNING
  ```

  识别并解决问题后，您必须首先终止现有的CN进程，然后使用正确的配置重新启动CN节点。

## 下一步

部署好StarRocks集群后，您可以继续执行[Post-deployment Setup](../deployment/post_deployment_setup.md)中的说明，进行初始管理操作。