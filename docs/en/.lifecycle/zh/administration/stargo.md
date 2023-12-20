---
displayed_sidebar: English
---

# 使用 StarGo 部署和管理 StarRocks

本主题介绍如何使用 StarGo 部署和管理 StarRocks 集群。

StarGo 是一个用于管理多个 StarRocks 集群的命令行工具。您可以通过 StarGo 轻松部署、检查、升级、降级、启动和停止多个集群。

## 安装 StarGo

将以下文件下载到您的中央控制节点：

- **sr-ctl**：StarGo 的二进制文件。下载后无需安装。
- **sr-c1.yaml**：部署配置文件的模板。
- **repo.yaml**：StarRocks 安装程序下载路径的配置文件。

> 注意
> 您可以访问 `http://cdn-thirdparty.starrocks.com` 获取相应的安装索引文件和安装程序。

```shell
wget https://github.com/wangtianyi2004/starrocks-controller/raw/main/stargo-pkg.tar.gz
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/sr-c1.yaml
wget https://github.com/wangtianyi2004/starrocks-controller/blob/main/repo.yaml
```

授予 **sr-ctl** 访问权限。

```shell
chmod 751 sr-ctl
```

## 部署 StarRocks 集群

您可以使用 StarGo 部署 StarRocks 集群。

### 先决条件

- 待部署的集群至少有一个中央控制节点和三个部署节点。所有节点都可以部署在一台机器上。
- 您需要在中央控制节点上部署 StarGo。
- 您需要在中央控制节点和三个部署节点之间建立相互的 SSH 认证。

以下示例在中央控制节点 sr-dev@r0 和三个部署节点 starrocks@r1、starrocks@r2 和 starrocks@r3 之间建立相互认证。

```plain
## 在 sr-dev@r0 和 starrocks@r1、2、3 之间建立相互认证。
[sr-dev@r0 ~]$ ssh-keygen
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r1
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r2
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r3

## 验证 sr-dev@r0 和 starrocks@r1、2、3 之间的相互认证。
[sr-dev@r0 ~]$ ssh starrocks@r1 date
[sr-dev@r0 ~]$ ssh starrocks@r2 date
[sr-dev@r0 ~]$ ssh starrocks@r3 date
```

### 创建配置文件

根据以下 YAML 模板创建 StarRocks 部署拓扑文件。详细信息请参阅[配置](../administration/FE_configuration.md)。

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
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
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
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
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
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
    config:
      sys_log_level: "INFO"
be_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    be_http_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir: StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
    config:
      create_tablet_worker_count: 3
```

### 创建部署目录（可选）

如果 StarRocks 需要部署的路径不存在，而您有创建该路径的权限，则您无需创建这些路径，StarGo 会根据配置文件为您创建。如果路径已存在，请确保您具有对它们的写入权限。您还可以通过运行以下命令在每个节点上创建必要的部署目录。

- 在 FE 节点上创建 **meta** 目录。

```shell
mkdir -p StarRocks/fe/meta
```

- 在 BE 节点上创建 **storage** 目录。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
> 请确保上述路径与配置文件中的配置项 `meta_dir` 和 `storage_dir` 一致。

### 部署 StarRocks

通过运行以下命令部署 StarRocks 集群。

```shell
./sr-ctl cluster deploy <cluster_name> <version> <topology_file>
```

| 参数 | 说明 |
| --- | --- |
| cluster_name | 要部署的集群的名称。 |
| version | StarRocks 版本。 |
| topology_file | 配置文件的名称。 |

如果部署成功，集群会自动启动。当 beStatus 和 feStatus 为 true 时，集群启动成功。

示例：

```plain
[sr-dev@r0 ~]$ ./sr-ctl cluster deploy sr-c1 v2.0.1 sr-c1.yaml
[20220301-234817  OUTPUT] 部署集群 [clusterName = sr-c1, clusterVersion = v2.0.1, metaFile = sr-c1.yaml]
[20220301-234836  OUTPUT] 预检查部署环境：
预检查 FE：
IP                    ssh 认证         meta 目录                   部署目录                 http 端口        rpc 端口         查询端口       编辑日志端口
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过             通过                       通过                       通过             通过             通过             通过
192.168.xx.xx         通过             通过                       通过                       通过             通过             通过             通过
192.168.xx.xx         通过             通过                       通过                       通过             通过             通过             通过

预检查 BE：
IP                    ssh 认证         存储目录                    部署目录                   web 服务端口      心跳服务端口     brpc 端口        be 端口
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过             通过                       通过                       通过             通过             通过             通过
192.168.xx.xx         通过             通过                       通过                       通过             通过             通过             通过
192.168.xx.xx         通过             通过                       通过                       通过             通过             通过             通过


[20220301-234836  OUTPUT] 预检查成功。尊重
[20220301-234836  OUTPUT] 创建部署文件夹...
[20220301-234838  OUTPUT] 下载 StarRocks 包 & jdk...
[20220302-000515    INFO] 文件 starrocks-2.0.1-quickstart.tar.gz [1227406189] 下载成功
[20220302-000515  OUTPUT] 下载完成。
[20220302-000515  OUTPUT] 解压 StarRocks 包 & jdk...
[20220302-000520    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220302-000547    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220302-000556    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220302-000556  OUTPUT] 分发 FE 目录...
[20220302-000603    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 至 feTargetDir = [StarRocks/fe] 在 FeHost = [192.168.xx.xx]
[20220302-000615    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 至 JDKTargetDir = [StarRocks/fe/jdk] 在 FeHost = [192.168.xx.xx]
[20220302-000615    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
```
```plain
[20220302-000622    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 到 feTargetDir = [StarRocks/fe] 在 FeHost = [192.168.xx.xx]
[20220302-000634    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 到 JDKTargetDir = [StarRocks/fe/jdk] 在 FeHost = [192.168.xx.xx]
[20220302-000634    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 到 feTargetDir = [StarRocks/fe] 在 FeHost = [192.168.xx.xx]
[20220302-000652    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 到 JDKTargetDir = [StarRocks/fe/jdk] 在 FeHost = [192.168.xx.xx]
[20220302-000652    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  OUTPUT] 分发 BE 目录...
[20220302-000728    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 到 BeTargetDir = [StarRocks/be] 在 BeHost = [192.168.xx.xx]
[20220302-000752    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 到 BeTargetDir = [StarRocks/be] 在 BeHost = [192.168.xx.xx]
[20220302-000815    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 到 BeTargetDir = [StarRocks/be] 在 BeHost = [192.168.xx.xx]
[20220302-000815  OUTPUT] 修改 FE 节点和 BE 节点的配置...
############################################# 启动 FE 集群 #############################################
############################################# 启动 FE 集群 #############################################
[20220302-000816    INFO] 启动领导者 FE 节点 [主机 = 192.168.xx.xx, editLogPort = 9010]
[20220302-000836    INFO] FE 节点成功启动 [主机 = 192.168.xx.xx, queryPort = 9030]
[20220302-000836    INFO] 启动追随者 FE 节点 [主机 = 192.168.xx.xx, editLogPort = 9010]
[20220302-000857    INFO] FE 节点成功启动 [主机 = 192.168.xx.xx, queryPort = 9030]
[20220302-000857    INFO] 启动追随者 FE 节点 [主机 = 192.168.xx.xx, editLogPort = 9010]
[20220302-000918    INFO] FE 节点成功启动 [主机 = 192.168.xx.xx, queryPort = 9030]
[20220302-000918    INFO] 列出所有 FE 状态:
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# 启动 BE 集群 #############################################
############################################# 启动 BE 集群 #############################################
[20220302-000918    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-000939    INFO] BE 节点成功启动 [主机 = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-000939    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001000    INFO] BE 节点成功启动 [主机 = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-001000    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx HeartbeatServicePort = 9050]
[20220302-001020    INFO] BE 节点成功启动 [主机 = 192.168.xx.xx, heartbeatServicePort = 9050]
[20220302-001020  OUTPUT] 列出所有 BE 状态:
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true
```

您可以通过[查看集群信息](#查看集群信息)来测试集群。

您也可以通过 MySQL 客户端连接集群进行测试。

```shell
mysql -h 127.0.0.1 -P9030 -uroot
```

## 查看集群信息

您可以查看 StarGo 管理的集群信息。

### 查看所有集群信息

执行以下命令查看所有集群信息。

```shell
./sr-ctl cluster list
```

例子：

```shell
[sr-dev@r0 ~]$ ./sr-ctl cluster list
[20220302-001640  OUTPUT] 列出所有集群
ClusterName      User        CreateDate                 MetaPath                                                      PrivateKey
---------------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-c1            starrocks   2022-03-02 00:08:15        /home/sr-dev/.starrocks-controller/cluster/sr-c1              /home/sr-dev/.ssh/id_rsa
```

### 查看特定集群的信息

运行以下命令查看特定集群的信息。

```shell
./sr-ctl cluster display <cluster_name>
```

例子：

```plain
[sr-dev@r0 ~]$ ./sr-ctl cluster display sr-c1
[20220302-002310  OUTPUT] 显示集群 [clusterName = sr-c1]
clusterName = sr-c1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage
```

## 启动集群

您可以通过 StarGo 启动 StarRocks 集群。

### 启动集群中所有节点

通过运行以下命令启动集群中的所有节点。

```shell
./sr-ctl cluster start <cluster-name>
```

例子：

```plain
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1
[20220303-190404  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-190404    INFO] 启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] 启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] 启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 启动特定角色的节点

- 启动集群中的所有 FE 节点。

```shell
./sr-ctl cluster start <cluster_name> --role FE
```

- 启动集群中的所有 BE 节点。

```shell
./sr-ctl cluster start <cluster_name> --role BE
```

例子：

```plain
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-191529    INFO] 启动 FE 集群....
[20220303-191529    INFO] 启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] 启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] 启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-194215    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] 启动 BE 集群...
```

### 启动特定节点

启动集群中的特定节点。目前仅支持 BE 节点。

```shell
./sr-ctl cluster start <cluster_name> --node <node_ID>
```

您可以通过[查看特定集群的信息](#查看特定集群的信息)来查看特定节点的 ID。

例子：

```plain
[root@nd1 sr-controller]# ./sr-ctl cluster start sr-c1 --node 192.168.xx.xx:9060
```
```plain
[20220303-194714  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-194714    INFO] 启动 BE 节点。[BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## 停止集群

您可以通过 StarGo 停止 StarRocks 集群。

### 停止集群中的所有节点

通过运行以下命令停止集群中的所有节点。

```shell
./sr-ctl cluster stop <cluster_name>
```

示例：

```plain
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1
[20220302-180140  OUTPUT] 停止集群 [clusterName = sr-c1]
[20220302-180140  OUTPUT] 停止集群 sr-c1
[20220302-180140    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180143  OUTPUT] FE 节点已成功停止 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180143    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180145  OUTPUT] FE 节点已成功停止 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180145    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180148  OUTPUT] FE 节点已成功停止 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180148  OUTPUT] 停止集群 sr-c1
[20220302-180148    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180148    INFO] BE 节点已成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180148    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] BE 节点已成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180149    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] BE 节点已成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 停止特定角色的节点

- 停止集群中的所有 FE 节点。

```shell
./sr-ctl cluster stop <cluster_name> --role FE
```

- 停止集群中的所有 BE 节点。

```shell
./sr-ctl cluster stop <cluster_name> --role BE
```

示例：

```plain
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role BE
[20220302-180624  OUTPUT] 停止集群 [clusterName = sr-c1]
[20220302-180624  OUTPUT] 停止集群 sr-c1
[20220302-180624    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180624    INFO] BE 节点已成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180624    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] BE 节点已成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] BE 节点已成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] 正在停止 BE 集群 ...

###########################################################################

[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] 停止集群 [clusterName = sr-c1]
[20220302-180849    INFO] 正在停止 FE 集群 ....
[20220302-180849  OUTPUT] 停止集群 sr-c1
[20220302-180849    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180851  OUTPUT] FE 节点已成功停止 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180851    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180854  OUTPUT] FE 节点已成功停止 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180854    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180856  OUTPUT] FE 节点已成功停止 [host = 192.168.xx.xx, queryPort = 9030]
```

### 停止特定节点

停止集群中的特定节点。

```shell
./sr-ctl cluster stop <cluster_name> --node <node_ID>
```

您可以通过[查看特定集群的信息](#查看特定集群的信息)来查看特定节点的 ID。

示例：

```plain
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-c1
[20220303-185400  OUTPUT] 显示集群 [clusterName = sr-c1]
clusterName = sr-c1
[20220303-185400    WARN] 所有 FE 节点都已关闭，请启动 FE 节点并再次显示集群状态。
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        DOWN        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        DOWN        StarRocks/be                                   /dataStarRocks/be/storage

[root@nd1 sr-controller]# ./sr-ctl cluster stop sr-c1 --node 192.168.xx.xx:9060
[20220303-185510  OUTPUT] 停止集群 [clusterName = sr-c1]
[20220303-185510    INFO] 正在停止 BE 节点。[BeHost = 192.168.xx.xx]
[20220303-185510    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
```

## 横向扩展集群

您可以通过 StarGo 横向扩展集群。

### 创建配置文件

根据以下模板创建横向扩展任务拓扑文件。您可以根据需要指定文件来添加 FE 和/或 BE 节点。详细信息请参见[配置](../administration/FE_configuration.md)。

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
    priority_networks: 192.168.xx.xx/24 # 当机器有多个 IP 地址时，指定当前节点的唯一 IP。
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

### 构建 SSH 相互认证

如果您要向集群添加新节点，则必须在新节点和中央控制节点之间建立相互认证。详细说明请参阅[先决条件](#先决条件)。

### 创建部署目录（可选）

如果新节点的部署路径不存在，并且您有权限创建该路径，则无需创建这些路径，StarGo 将根据配置文件为您创建。如果路径已存在，请确保您有权访问它们。您还可以通过运行以下命令在每个节点上创建必要的部署目录。

- 在 FE 节点上创建 meta 目录。

```shell
mkdir -p StarRocks/fe/meta
```

- 在 BE 节点上创建 storage 目录。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
请确保上述路径与配置文件中的配置项 `meta_dir` 和 `storage_dir` 相匹配。

### 横向扩展集群

通过运行以下命令横向扩展集群。

```shell
./sr-ctl cluster scale-out <cluster_name> <topology_file>
```

示例：

```plain
# 横向扩展前的集群状态。
[root@nd1 sr-controller]# ./sr-ctl cluster display sr-test       
[20220503-210047  OUTPUT] 显示集群 [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          ROLE    HOST                  PORT             STAT        DATADIR                                             DEPLOYDIR                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        UP          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        UP          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage
# 扩展集群
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster scale-out sr-test sr-out.yaml
[20220503-213725  OUTPUT] 扩展集群。[ClusterName = sr-test]
[20220503-213731  OUTPUT] 预检部署环境：
预检 FE：
IP                    ssh 认证         meta 目录                        部署目录                      http 端口        rpc 端口         查询端口       编辑日志端口  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过             通过                            通过                            通过             通过             通过             通过           

预检 BE：
IP                    ssh 认证         存储目录                     部署目录                      web 服务端口      心跳端口   brpc 端口        be 端口        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过             通过                            通过                            通过             通过             通过             通过           


[20220503-213731  OUTPUT] 预检成功。尊重
[20220503-213731  OUTPUT] 创建部署文件夹...
[20220503-213732  OUTPUT] 下载 StarRocks 包 & jdk...
[20220503-213732    INFO] 包已经存在 [文件名 = starrocks-2.0.1-quickstart.tar.gz, 文件大小 = 1227406189, 文件修改时间 = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  OUTPUT] 下载完成。
[20220503-213732  OUTPUT] 解压 StarRocks 包 & jdk...
[20220503-213741    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220503-213837    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220503-213837    INFO] tar 文件 /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 下解压
[20220503-213837  OUTPUT] 分发 FE 目录...
[20220503-213845    INFO] 上传目录 feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 到 feTargetDir = [StarRocks/fe] 在 FeHost = [192.168.xx.xx]
[20220503-213857    INFO] 上传目录 JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 到 JDKTargetDir = [StarRocks/fe/jdk] 在 FeHost = [192.168.xx.xx]
[20220503-213857    INFO] 修改 JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857  OUTPUT] 分发 BE 目录...
[20220503-213924    INFO] 上传目录 BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 到 BeTargetDir = [StarRocks/be] 在 BeHost = [192.168.xx.xx]
[20220503-213924  OUTPUT] 修改 FE 节点 & BE 节点的配置...
############################################# 扩展 FE 集群 #############################################
############################################# 扩展 FE 集群 #############################################
[20220503-213925    INFO] 启动从属 FE 节点 [主机 = 192.168.xx.xx, 编辑日志端口 = 9010]
[20220503-213945    INFO] FE 节点成功启动 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220503-213945    INFO] 列出所有 FE 状态：
                                        feHost = 192.168.xx.xx       feQueryPort = 9030     feStatus = true

############################################# 启动 BE 集群 #############################################
############################################# 启动 BE 集群 #############################################
[20220503-213945    INFO] 启动 BE 节点 [BeHost = 192.168.xx.xx 心跳服务端口 = 9050]
[20220503-214016    INFO] BE 节点成功启动 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220503-214016  OUTPUT] 列出所有 BE 状态：
                                        beHost = 192.168.xx.xx       beHeartbeatServicePort = 9050      beStatus = true

# 扩展后的集群状态。
[sr-dev@nd1 sr-controller]$ ./sr-ctl cluster display sr-test 
[20220503-214302  OUTPUT] 显示集群 [clusterName = sr-test]
clusterName = sr-test
clusterVerison = v2.0.1
ID                          角色    主机                  端口             状态        数据目录                                             部署目录                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        运行中          /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        运行中          StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        运行中          /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        运行中          StarRocks/be                                   StarRocks/be/storage                         
```
```plain
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```

## 相关命令

|命令|描述|
|---|---|
|deploy|部署集群。|
|start|启动集群。|
|stop|停止集群。|
|scale-in|缩容集群。|
|scale-out|扩容集群。|
|upgrade|升级集群。|
|downgrade|降级集群|
|display|查看特定集群的信息。|
|list|查看所有集群。|