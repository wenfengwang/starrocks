wget https://github.com/wangtianyi2004/starrocks-controller/raw/main/stargo-pkg.tar.gz
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/sr-c1.yaml
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/repo.yaml

```

授予 **sr-ctl** 访问权限。

```shell

chmod 751 sr-ctl

```

## 部署StarRocks集群

您可以使用StarGo部署StarRocks集群。

### 先决条件


- 要部署的集群必须至少包含一个中央控制节点和三个部署节点。所有节点都可以部署在一台机器上。
- 您需要在中央控制节点上部署StarGo。
- 您需要在中央控制节点和三个部署节点之间建立互相的SSH身份验证。

以下示例建立中央控制节点sr-dev@r0和三个部署节点starrocks@r1、starrocks@r2和starrocks@r3之间的互相身份验证。

```plain text

## 建立sr-dev@r0和starrocks@r1, 2, 3之间的互相身份验证。
[sr-dev@r0 ~]$ ssh-keygen
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r1

[sr-dev@r0 ~]$ ssh-copy-id starrocks@r2

[sr-dev@r0 ~]$ ssh-copy-id starrocks@r3

## 验证sr-dev@r0和starrocks@r1, 2, 3之间的互相身份验证。

[sr-dev@r0 ~]$ ssh starrocks@r1 date
[sr-dev@r0 ~]$ ssh starrocks@r2 date
[sr-dev@r0 ~]$ ssh starrocks@r3 date

```

### 创建配置文件

根据以下YAML模板创建StarRocks部署拓扑文件。详细信息请参阅 [配置](../administration/Configuration.md)。

```yaml
global:

    user: "starrocks"   ## 当前操作系统用户。
    ssh_port: 22

fe_servers:
  - host: 192.168.XX.XX

    ssh_port: 22

    http_port: 8030

    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe

    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.XX.XX/24 # 当机器具有多个IP地址时，指定当前节点的唯一IP。
    config:
      sys_log_level: "INFO"
  - host: 192.168.XX.XX
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.XX.XX/24 # 当机器具有多个IP地址时，指定当前节点的唯一IP。
    config:
      sys_log_level: "INFO"
  - host: 192.168.XX.XX
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.XX.XX/24 # 当机器具有多个IP地址时，指定当前节点的唯一IP。
    config:
      sys_log_level: "INFO"
be_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器具有多个IP地址时，指定当前节点的唯一IP。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器具有多个IP地址时，指定当前节点的唯一IP。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器具有多个IP地址时，指定当前节点的唯一IP。
    config:
      create_tablet_worker_count: 3
```

### 创建部署目录（可选）

如果要部署StarRocks的路径不存在，并且您有权限创建这些路径，则无需创建这些路径，StarGo将根据配置文件为您创建这些路径。如果路径已存在，请确保您有对它们的写入访问权限。您还可以通过以下命令在每个节点上创建必要的部署目录。

- 在FE节点上创建 **meta** 目录。

```shell

mkdir -p StarRocks/fe/meta

```

- 在BE节点上创建 **storage** 目录。

```shell
mkdir -p StarRocks/be/storage
```

> 注意

> 确保上述路径与配置文件中的 `meta_dir` 和 `storage_dir` 配置项相同。

### 部署StarRocks

运行以下命令部署StarRocks集群。


```shell

./sr-ctl cluster deploy <cluster_name> <version> <topology_file>

```

|参数|描述|

|----|----|
|cluster_name|要部署的集群名称。|
|version|StarRocks的版本。|
|topology_file|配置文件的名称。|


如果部署成功，集群将自动启动。当 beStatus 和 feStatus 为 true 时，表示集群已成功启动。

示例：

```plain text
[sr-dev@r0 ~]$ ./sr-ctl cluster deploy sr-c1 v2.0.1 sr-c1.yaml
[20220301-234817  输出] 部署集群 [clusterName = sr-c1, clusterVersion = v2.0.1, metaFile = sr-c1.yaml]
[20220301-234836  输出] 部署环境预检查:
预检查 FE:
IP                    ssh auth         meta dir                   deploy dir                 http port        rpc port         query port       edit log port
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS

预检查 BE:
IP                    ssh auth         storage dir                deploy dir                 webSer port      heartbeat port   brpc port        be port
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS
192.168.xx.xx         PASS             PASS                       PASS                       PASS             PASS             PASS             PASS


[20220301-234836  输出] 预检查成功。 RESPECT
```
[20220301-234836  OUTPUT] 创建 deploy 文件夹...
[20220301-234838  OUTPUT] 下载 StarRocks 软件包 & jdk ...
[20220302-000515    INFO] 文件 starrocks-2.0.1-quickstart.tar.gz [1227406189] 下载成功
[20220302-000515  OUTPUT] 下载完成。
[20220302-000515  OUTPUT] 解压 StarRocks 软件包 & jdk ...
[20220302-000520    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz 已解压至 /home/sr-dev/.starrocks-controller/download
[20220302-000547    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz 已解压至 /home/sr-dev/.starrocks-controller/download
[20220302-000556    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz 已解压至 /home/sr-dev/.starrocks-controller/download
[20220302-000556  OUTPUT] 分发 FE 目录...
[20220302-000603    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 至 feTargetDir = [StarRocks/fe]，在 FeHost = [192.168.xx.xx]
[20220302-000615    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 至 JDKTargetDir = [StarRocks/fe/jdk]，在 FeHost = [192.168.xx.xx]
[20220302-000615    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx]，文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000622    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 至 feTargetDir = [StarRocks/fe]，在 FeHost = [192.168.xx.xx]
[20220302-000634    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 至 JDKTargetDir = [StarRocks/fe/jdk]，在 FeHost = [192.168.xx.xx]
[20220302-000634    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx]，文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 至 feTargetDir = [StarRocks/fe]，在 FeHost = [192.168.xx.xx]
[20220302-000652    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 至 JDKTargetDir = [StarRocks/fe/jdk]，在 FeHost = [192.168.xx.xx]
[20220302-000652    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx]，文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  OUTPUT] 分发 BE 目录...
[20220302-000728    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 至 BeTargetDir = [StarRocks/be]，在 BeHost = [192.168.xx.xx]
[20220302-000752    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 至 BeTargetDir = [StarRocks/be]，在 BeHost = [192.168.xx.xx]
[20220302-000815    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 至 BeTargetDir = [StarRocks/be]，在 BeHost = [192.168.xx.xx]
[20220302-000815  OUTPUT] 修改 FE 节点 & BE 节点的配置...
############################################# 启动 FE 集群 #############################################
############################################# 启动 FE 集群 #############################################
[20220302-000816    INFO] 启动Leader FE节点 [主机 = 192.168.xx.xx, editLogPort = 9010]
[20220302-000836    INFO] FE节点启动成功 [主机 = 192.168.xx.xx, queryPort = 9030]
[20220302-000836    INFO] 启动Follower FE节点 [主机 = 192.168.xx.xx, editLogPort = 9010]
[20220302-000857    INFO] FE节点启动成功 [主机 = 192.168.xx.xx, queryPort = 9030]
[20220302-000857    INFO] 启动Follower FE节点 [主机 = 192.168.xx.xx, editLogPort = 9010]
[20220302-000918    INFO] FE节点启动成功 [主机 = 192.168.xx.xx, queryPort = 9030]
[20220302-000918    INFO] 列出所有 FE 状态:
                                        fe主机 = 192.168.xx.xx       fe查询端口 = 9030     fe状态 = true
                                        fe主机 = 192.168.xx.xx       fe查询端口 = 9030     fe状态 = true
                                        fe主机 = 192.168.xx.xx       fe查询端口 = 9030     fe状态 = true

############################################# 启动 BE 集群 #############################################
############################################# 启动 BE 集群 #############################################
[20220302-000918    INFO] 启动 BE 节点 [Be主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220302-000939    INFO] BE节点启动成功 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220302-000939    INFO] 启动 BE 节点 [Be主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220302-001000    INFO] BE节点启动成功 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220302-001000    INFO] 启动 BE 节点 [Be主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220302-001020    INFO] BE节点启动成功 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220302-001020  OUTPUT] 列出所有 BE 状态:
                                        be主机 = 192.168.xx.xx       be心跳服务端口 = 9050      be状态 = true
                                        be主机 = 192.168.xx.xx       be心跳服务端口 = 9050      be状态 = true
                                        be主机 = 192.168.xx.xx       be心跳服务端口 = 9050      be状态 = true
```
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
```

## 启动集群

您可以通过StarGo启动StarRocks集群。

### 启动集群中的所有节点

通过运行以下命令启动集群中的所有节点。

```shell
./sr-ctl cluster start <cluster-name>
```

示例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1
[20220303-190404  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-190404    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 启动特定角色的节点

- 启动集群中的所有FE节点。

```shell
./sr-ctl cluster start <cluster_name> --role FE
```

- 启动集群中的所有BE节点。

```shell
./sr-ctl cluster start <cluster_name> --role BE
```

示例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-191529    INFO] Starting FE cluster ....
[20220303-191529    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] Starting FE node [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-194215    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] Starting BE node [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] Starting BE cluster ...
```

### 启动特定节点

在集群中启动特定节点。目前，仅支持启动BE节点。

```shell
./sr-ctl cluster start <cluster_name> --node <node_ID>
```

您可以通过[查看特定集群的信息](#view-the-information-of-a-specific-cluster)来查找特定节点的ID。

示例：

```plain text
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --node 192.168.xx.xx:9060
[20220303-194714  OUTPUT] Start cluster [clusterName = sr-c1]
[20220303-194714    INFO] Start BE node. [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## 停止集群

您可以通过StarGo停止StarRocks集群。

### 停止集群中的所有节点

通过运行以下命令停止集群中的所有节点。

```shell
./sr-ctl cluster stop <cluster_name>
```

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1
[20220302-180140  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180140  OUTPUT] Stop cluster sr-c1
[20220302-180140    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180143  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180143    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180145  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180145    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180148  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180148  OUTPUT] Stop cluster sr-c1
[20220302-180148    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180148    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180148    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180149    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 停止特定角色的节点

- 停止集群中的所有FE节点。

```shell
./sr-ctl cluster stop <cluster_name> --role FE
```

- 停止集群中的所有BE节点。

```shell
./sr-ctl cluster stop <cluster_name> --role BE
```

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role BE
[20220302-180624  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180624  OUTPUT] Stop cluster sr-c1
[20220302-180624    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180624    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180624    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] Waiting for stoping BE node [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] The BE node stop succefully [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] Stopping BE cluster ...

###########################################################################

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] Stop cluster [clusterName = sr-c1]
[20220302-180849    INFO] Stopping FE cluster ....
[20220302-180849  OUTPUT] Stop cluster sr-c1
[20220302-180849    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
[20220302-180851  OUTPUT] The FE node stop succefully [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180851    INFO] Waiting for stoping FE node [FeHost = 192.168.xx.xx]
```markdown
[20220302-180854  OUTPUT] FE节点成功停止 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220302-180854    INFO] 等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180856  OUTPUT] FE节点成功停止 [主机 = 192.168.xx.xx, 查询端口 = 9030]
```

### 停止指定节点

停止集群中的特定节点。

```shell
./sr-ctl 集群 停止 <集群名称> --节点 <节点ID>
```

您可以通过[查看特定集群的信息](#查看特定集群的信息)来检查特定节点的ID。

示例:

```plain text
[root@nd1 sr-controller]# ./sr-ctl 集群 显示 sr-c1
[20220303-185400  OUTPUT] 显示集群 [集群名称 = sr-c1]
集群名称 = sr-c1
[20220303-185400    WARN] 所有 FE 节点都已停止，请启动 FE 节点并再次显示集群状态。
ID                          角色    主机                  端口             状态        数据目录                                             部署目录
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        关闭        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        关闭        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        关闭        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        关闭        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        关闭        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        关闭        StarRocks/be                                   /dataStarRocks/be/storage

[root@nd1 sr-controller]# ./sr-ctl 集群 停止 sr-c1 --节点 192.168.xx.xx:9060
[20220303-185510  OUTPUT] 停止集群 [集群名称 = sr-c1]
[20220303-185510    INFO] 正在停止 BE 节点。[BeHost = 192.168.xx.xx]
[20220303-185510  INFO] 等待停止 BE 节点 [BeHost = 192.168.xx.xx]
```

## 扩展集群

您可以通过StarGo扩展集群。

### 创建配置文件

根据以下模板创建扩展任务拓扑文件。您可以根据需求指定文件以添加 FE 和/或 BE 节点。有关详细信息，请参阅[配置](../administration/Configuration.md)。

```yaml
# 添加一个 FE 节点。
fe_servers:
  - host: 192.168.xx.xx # 新 FE 节点的 IP 地址。
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.xx.xx/24 # 当机器拥有多个 IP 地址时，请指定当前节点的唯一 IP。
    config:
      sys_log_level: "INFO"
      sys_log_delete_age: "1d"

# 添加一个 BE 节点。
be_servers:
  - host: 192.168.xx.xx # 新 BE 节点的 IP 地址。
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    config:
      create_tablet_worker_count: 3
```

### 建立SSH互相认证

如果要将新节点添加到集群中，您必须在新节点和中央控制节点之间建立互相认证。有关详细说明，请参阅[先决条件](#先决条件)。

### 创建部署目录（可选）

如果要部署的新节点路径不存在，并且您有权限创建该路径，则无需创建这些路径，StarGo 将根据配置文件为您创建这些路径。如果路径已存在，请确保您具有对其的写权限。您还可以通过运行以下命令在每个节点上创建必要的部署目录。

- 在 FE 节点上创建**meta**目录。

```shell
mkdir -p StarRocks/fe/meta
```

- 在 BE 节点上创建**storage**目录。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 确保上述路径与配置文件中的 `meta_dir` 和 `storage_dir` 项目相同。

### 扩展集群

通过运行以下命令扩展集群。

```shell
./sr-ctl 集群 扩展 <集群名称> <拓扑文件>
```

示例:

```plain text
# 扩容前集群状态。
[root@nd1 sr-controller]# ./sr-ctl 集群 显示 sr-test       
[20220503-210047  OUTPUT] 显示集群 [集群名称 = sr-test]
集群名称 = sr-test
集群版本 = v2.0.1
ID                          角色    主机                  端口             状态        数据目录                                             部署目录
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        运行        /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        运行        /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    

# 扩容集群。
[sr-dev@nd1 sr-controller]$ ./sr-ctl 集群 扩展 sr-test sr-out.yaml
[20220503-213725  OUTPUT] 扩容集群。[集群名称 = sr-test]
[20220503-213731  OUTPUT] 预检查部署环境:
预检查 FE:
IP                    SSH认证         Meta 目录                     部署目录                      Http 端口        RPC 端口         查询端口       编辑日志端口  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过             通过                            通过                            通过             通过             通过             通过           

预检查 BE:
IP                    SSH认证         Storage 目录                     部署目录                      WebSer 端口      Heartbeat 端口   brpc端口        BE 端口        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过             通过                            通过                            通过             通过             通过             通过           


[20220503-213731  OUTPUT] 预检查成功。符合预期
[20220503-213731  OUTPUT] 创建部署文件夹...
[20220503-213732  OUTPUT] 下载 StarRocks 包 & JDK...
[20220503-213732    INFO] 包已经存在 [文件名 = starrocks-2.0.1-quickstart.tar.gz, 文件大小 = 1227406189, 文件修改时间 = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  OUTPUT] 下载完成。
[20220503-213732  OUTPUT] 解压 StarRocks 包 & JDK...
[20220503-213741    INFO] 压缩文件 /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220503-213837    INFO] 压缩文件 /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220503-213837    INFO] 压缩文件 /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220503-213837  OUTPUT] 分发 FE 目录...
[20220503-213845    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 至 feTargetDir = [StarRocks/fe] 于 FE 主机 = [192.168.xx.xx]
[20220503-213857    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 到 JDKTargetDir = [StarRocks/fe/jdk] 到 FeHost = [192.168.xx.xx]
[20220503-213857    INFO] 修改 JAVA_HOME: host = [192.168.xx.xx], filePath = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857  OUTPUT] 分发 BE 目录...
[20220503-213924    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 到 BeTargetDir = [StarRocks/be] 到 BeHost = [192.168.xx.xx]
[20220503-213924  OUTPUT] 修改 FE 节点和 BE 节点的配置...
############################################# 扩展 FE 集群 #############################################
############################################# 扩展 FE 集群 #############################################
[20220503-213925    INFO] 启动从属 FE 节点 [主机 = 192.168.xx.xx，编辑日志端口 = 9010]
[20220503-213945    INFO] FE 节点成功启动 [主机 = 192.168.xx.xx，查询端口 = 9030]
[20220503-213945    INFO] 列出所有 FE 状态:
                                        fe 主机 = 192.168.xx.xx       fe 查询端口 = 9030     fe 状态 = true

############################################# 启动 BE 集群 #############################################
############################################# 启动 BE 集群 #############################################
[20220503-213945    INFO] 启动 BE 节点 [Be 主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220503-214016    INFO] BE 节点成功启动 [主机 = 192.168.xx.xx，心跳服务端口 = 9050]
[20220503-214016  OUTPUT] 列出所有 BE 状态:
                                        be 主机 = 192.168.xx.xx       be 心跳服务端口 = 9050      be 状态 = true

# 扩容后集群的状态。
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-test 
[20220503-214302  OUTPUT] 显示集群 [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   StarRocks/be/storage                         
```

## 缩减集群

通过运行以下命令来从集群中删除一个节点。

```shell
./sr-ctl cluster scale-in <cluster_name> --node <node_id>
```

您可以通过[查看特定集群的信息](#view-the-information-of-a-specific-cluster)来查看特定节点的 ID。

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220505-145649  OUTPUT] 显示集群 [clusterName = sr-c1]
clusterName = sr-c1
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-in sr-c1 --node 192.168.88.83:9010
[20220621-010553  OUTPUT] 缩减集群 [clusterName = sr-c1, nodeId = 192.168.88.83:9010]
[20220621-010553    INFO] 等待停止 FE 节点 [Fe 主机 = 192.168.88.83]
[20220621-010606  OUTPUT] 成功缩减 FE 节点。[clusterName = sr-c1, nodeId = 192.168.88.83:9010]

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-c1
[20220621-010623  OUTPUT] 显示集群 [clusterName = sr-c1]
clusterName = sr-c1
clusterVerison = 
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.88.84:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.85:9010          FE      192.168.xx.xx         9010/9030        UP/L        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.83:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.84:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.85:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage              
```

## 升级或降级集群

您可以通过 StarGo 升级或降级集群。

- 升级集群。

```shell
./sr-ctl cluster upgrade <cluster_name>  <target_version>
```

- 降级集群。

```shell
./sr-ctl cluster downgrade <cluster_name>  <target_version>
```

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-195827  OUTPUT] 列出所有集群
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 19:35:36        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster upgrade sr-test2 v2.1.3
[20220515-200358  OUTPUT] 列出所有集群
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.1.3      test222     2022-05-15 20:03:01        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster downgrade sr-test2 v2.0.1 
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster list
[20220515-200915  OUTPUT] 列出所有集群
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```

## 相关命令

|Command|Description|
|----|----|
|deploy|Deploy a cluster.|
|start|Start a cluster.|
|stop|Stop a cluster.|
|scale-in|Scale in a cluster.|
|scale-out|Scale out a cluster.|
|upgrade|Upgrade a cluster.|
|downgrade|Downgrade a cluster|
|display|View the information of a specific cluater.|
|list|View all clusters.|