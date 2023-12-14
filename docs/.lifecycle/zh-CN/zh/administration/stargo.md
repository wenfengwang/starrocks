wget https://raw.githubusercontent.com/wangtianyi2004/starrocks-controller/main/stargo-pkg.tar.gz
tar -xzvf stargo-pkg.tar.gz
```

安装包包含以下文件。

- **stargo**：StarGo 二进制文件，无需安装。

- **deploy-template.yaml**：部署配置文件模板。
- **repo.yaml**：指定 StarRocks 安装包下载库的配置文件。


## 部署集群

您可以使用 StarGo 部署 StarRocks 集群。

### 前提条件

- 待部署集群至少需要一个中控机节点和三个部署机节点，所有节点可以混合部署于同一台机器。

- 中控机上需部署 StarGo。
- 中控机与部署机间需创建 SSH 互信。

以下示例创建了中控机 sr-dev@r0 与部署机 starrocks@r1、starrocks@r2 以及 starrocks@r3 间的 SSH 互信。

```plain text

## 创建 sr-dev@r0 到 starrocks@r1、r2、r3 的 ssh 互信。
[sr-dev@r0 ~]$ ssh-keygen
[sr-dev@r0 ~]$ ssh-copy-id starrocks@r1

[sr-dev@r0 ~]$ ssh-copy-id starrocks@r2

[sr-dev@r0 ~]$ ssh-copy-id starrocks@r3

## 验证 sr-dev@r0 到 starrocks@r1、r2、r3 的 ssh 互信。

[sr-dev@r0 ~]$ ssh starrocks@r1 date
[sr-dev@r0 ~]$ ssh starrocks@r2 date
[sr-dev@r0 ~]$ ssh starrocks@r3 date

```

### 创建配置文件

根据以下 YAML 模板，创建部署 StarRocks 集群的拓扑文件。具体配置项参考[参数配置](../administration/Configuration.md)。

```yaml
global:

    user: "starrocks"   # 请修改为当前操作系统用户。
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
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 时，请在当前配置项中为当前节点指定唯一 IP。
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
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 时，在当前配置项中为当前节点指定唯一 IP。
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
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 时，在当前配置项中为当前节点指定唯一 IP。
    config:
      sys_log_level: "INFO"
be_servers:
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 时，在当前配置项中为当前节点指定唯一 IP。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 时，在当前配置项中为当前节点指定唯一 IP。
    config:
      create_tablet_worker_count: 3
  - host: 192.168.XX.XX
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    priority_networks: 192.168.XX.XX/24 # 当机器有多个 IP 时，在当前配置项中为当前节点指定唯一 IP。
    config:
      create_tablet_worker_count: 3
```

### 创建部署目录（可选）

如果您在配置文件中设定的部署路径不存在，且您有创建该路径的权限，StarGo 将根据配置文件自动创建部署目录。如果路径已存在，请确保您有在该路径下拥有写入的权限。您也可以通过以下命令，在各部署节点分别创建部署路径。

- 在 FE 节点安装目录下上创建 **meta** 路径。

```shell

mkdir -p StarRocks/fe/meta

```

- 在 BE 节点安装目录下上创建 **storage** 路径。

```shell
mkdir -p StarRocks/be/storage
```

> 注意

>
> 请确保以上创建的路径与配置文件中的 `meta_dir` 和 `storage_dir` 相同。


### 部署 StarRocks

通过以下命令部署 StarRocks 集群。

```shell

./stargo cluster deploy <cluster_name> <version> <topology_file>

```

|参数|描述|

|----|----|
|cluster_name|创建的集群名|
|version|StarRocks 的版本|
|topology_file|配置文件名|


创建成功后，集群将会自动启动。当返回 beStatus 和feStatus 为 true 时，集群部署启动成功。

示例：

```plain text
[sr-dev@r0 ~]$ ./stargo cluster deploy sr-c1 v2.0.1 sr-c1.yaml
[20220301-234817  输出] 部署集群 [clusterName = sr-c1, clusterVersion = v2.0.1, metaFile = sr-c1.yaml]
[20220301-234836  输出] 部署环境预检查:
预检查FE:
IP                    ssh授权           元数据目录                 部署目录                 http端口           rpc端口            查询端口          编辑日志端口
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过              通过                        通过                        通过              通过              通过              通过
192.168.xx.xx         通过              通过                        通过                        通过              通过              通过              通过
192.168.xx.xx         通过              通过                        通过                        通过              通过              通过              通过

预检查BE:
IP                    ssh授权           存储目录                    部署目录                 web服务端口       心跳端口            brpc端口         be端口
--------------------  ---------------  -------------------------  -------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过              通过                        通过                        通过              通过              通过              通过
192.168.xx.xx         通过              通过                        通过                        通过              通过              通过              通过
192.168.xx.xx         通过              通过                        通过                        通过              通过              通过              通过


[20220301-234836  输出] 预检查成功。敬上
[20220301-234836  输出] 创建部署文件夹 ...
[20220301-234838  输出] 下载StarRocks软件包 & jdk ...
[20220302-000515    信息] 文件 starrocks-2.0.1-quickstart.tar.gz [1227406189] 下载成功
[20220302-000515  输出] 下载完成。
[20220302-000515  输出] 解压StarRocks软件包 & jdk ...
[20220302-000520    信息] tar文件 /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 目录下解压
[20220302-000547    信息] tar文件 /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 目录下解压
[20220302-000556    信息] tar文件 /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz 已在 /home/sr-dev/.starrocks-controller/download 目录下解压
[20220302-000556  输出] 分发FE目录 ...
[20220302-000603    信息] 将目录feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 上传至feTargetDir = [StarRocks/fe] 在FeHost = [192.168.xx.xx]
[20220302-000615    信息] 将目录JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 上传至JDKTargetDir = [StarRocks/fe/jdk] 在FeHost = [192.168.xx.xx]
[20220302-000615    信息] 修改JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000622    信息] 将目录feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 上传至feTargetDir = [StarRocks/fe] 在FeHost = [192.168.xx.xx]
[20220302-000634    信息] 将目录JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 上传至JDKTargetDir = [StarRocks/fe/jdk] 在FeHost = [192.168.xx.xx]
[20220302-000634    信息] 修改JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000640    信息] 将目录feSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 上传至feTargetDir = [StarRocks/fe] 在FeHost = [192.168.xx.xx]
[20220302-000652    信息] 将目录JDKSourceDir = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 上传至JDKTargetDir = [StarRocks/fe/jdk] 在FeHost = [192.168.xx.xx]
[20220302-000652    信息] 修改JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220302-000652  输出] 分发BE目录 ...
[20220302-000728    信息] 将目录BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 上传至BeTargetDir = [StarRocks/be] 在BeHost = [192.168.xx.xx]
[20220302-000752    信息] 将目录BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 上传至BeTargetDir = [StarRocks/be] 在BeHost = [192.168.xx.xx]
[20220302-000815    信息] 将目录BeSourceDir = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 上传至BeTargetDir = [StarRocks/be] 在BeHost = [192.168.xx.xx]
[20220302-000815  输出] 修改FE节点 & BE节点配置 ...
############################################# 启动FE集群 #############################################
############################################# 启动FE集群 #############################################
[20220302-000816    信息] 启动领导者FE节点 [主机 = 192.168.xx.xx, 编辑日志端口 = 9010]
[20220302-000836    信息] FE节点成功启动 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220302-000836    信息] 启动追随者FE节点 [主机 = 192.168.xx.xx, 编辑日志端口 = 9010]
[20220302-000857    信息] FE节点成功启动 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220302-000857    信息] 启动追随者FE节点 [主机 = 192.168.xx.xx, 编辑日志端口 = 9010]
[20220302-000918    信息] FE节点成功启动 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220302-000918    信息] 列出所有FE状态:
                                        fe主机 = 192.168.xx.xx       fe查询端口 = 9030     fe状态 = true
                                        fe主机 = 192.168.xx.xx       fe查询端口 = 9030     fe状态 = true
                                        fe主机 = 192.168.xx.xx       fe查询端口 = 9030     fe状态 = true

############################################# 启动BE集群 #############################################
############################################# 启动BE集群 #############################################
[20220302-000918    信息] 启动BE节点 [Be主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220302-000939    信息] BE节点成功启动 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220302-000939    信息] 启动BE节点 [Be主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220302-001000    信息] BE节点成功启动 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220302-001000    信息] 启动BE节点 [Be主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220302-001020    信息] BE节点成功启动 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220302-001020  输出] 列出所有BE状态:
                                        be主机 = 192.168.xx.xx       be心跳服务端口 = 9050      be状态 = true
                                        be主机 = 192.168.xx.xx       be心跳服务端口 = 9050      be状态 = true
                                        be主机 = 192.168.xx.xx       be心跳服务端口 = 9050      be状态 = true
```

您可以通过[查看指定集群信息](#查看指定集群信息) 查看各节点是否部署成功。

您也可以通过连接 MySQL 客户端测试集群是否部署成功。

```shell
mysql -h 127.0.0.1 -P9030 -uroot
```

## 查看集群信息

您可以通过 StarGo 查看其管理的集群信息。

### 查看所有集群信息

通过以下命令查看其管理的所有集群信息。

```shell
./stargo cluster list
```

示例：

```shell
[sr-dev@r0 ~]$ ./stargo cluster list
[20220302-001640  OUTPUT] 列出所有集群
ClusterName      User        CreateDate                 MetaPath                                                      PrivateKey
---------------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-c1            starrocks   2022-03-02 00:08:15        /home/sr-dev/.starrocks-controller/cluster/sr-c1              /home/sr-dev/.ssh/id_rsa
```

### 查看指定集群信息

通过以下命令查看指定集群的信息。

```shell
./stargo cluster display <cluster_name>
```

示例：

```plain text
[sr-dev@r0 ~]$ ./stargo cluster display sr-c1
[20220302-002310  OUTPUT] 显示集群信息 [clusterName = sr-c1]
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

您可以通过 StarGo 启动其管理的集群。

### 启动集群所有节点

通过以下命令启动特定集群所有节点。

```shell
./stargo cluster start <cluster-name>
```

示例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster start sr-c1
[20220303-190404  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-190404    INFO] 正在启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190435    INFO] 正在启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190446    INFO] 正在启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-190457    INFO] 正在启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] 正在启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-190458    INFO] 正在启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 启动集群中特定角色节点

- 通过以下命令启动特定集群中 FE 节点。

```shell
./stargo cluster start <cluster_name> --role FE
```

- 通过以下命令启动特定集群中 BE 节点。

```shell
./stargo cluster start <cluster_name> --role BE
```

示例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster start sr-c1 --role FE
[20220303-191529  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-191529    INFO] 正在启动 FE 集群 ....
[20220303-191529    INFO] 正在启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191600    INFO] 正在启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]
[20220303-191610    INFO] 正在启动 FE 节点 [FeHost = 192.168.xx.xx, EditLogPort = 9010]

[root@nd1 sr-controller]# ./stargo cluster start sr-c1 --role BE
[20220303-194215  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-194215    INFO] 正在启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194216    INFO] 正在启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] 正在启动 BE 节点 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220303-194217    INFO] 正在启动 BE 集群 ...
```

### 启动集群中特定节点

通过以下命令启动集群的某一个节点。目前只支持启动 BE 的节点。

```shell
./stargo cluster start <cluster_name> --node <node_ID>
```

您可以通过[查看指定集群信息](#查看指定集群信息)查看集群中特定节点的 ID。

示例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster start sr-c1 --node 192.168.xx.xx:9060
[20220303-194714  OUTPUT] 启动集群 [clusterName = sr-c1]
[20220303-194714    INFO] 启动 BE 节点. [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

## 停止集群

您可以通过 StarGo 停止其管理的集群。

### 停止集群所有节点

通过以下命令停止特定集群所有节点。

```shell
./stargo cluster stop <cluster_name>
```

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster stop sr-c1
[20220302-180140  OUTPUT] 停止集群 [clusterName = sr-c1]
[20220302-180140  OUTPUT] 停止集群 sr-c1
[20220302-180140    INFO] 正在等待 FE 节点关闭 [FeHost = 192.168.xx.xx]
[20220302-180143  OUTPUT] FE 节点已成功关闭 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180143    INFO] 正在等待 FE 节点关闭 [FeHost = 192.168.xx.xx]
[20220302-180145  OUTPUT] FE 节点已成功关闭 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180145    INFO] 正在等待 FE 节点关闭 [FeHost = 192.168.xx.xx]
[20220302-180148  OUTPUT] FE 节点已成功关闭 [host = 192.168.xx.xx, queryPort = 9030]
[20220302-180148  OUTPUT] 停止集群 sr-c1
[20220302-180148    INFO] 正在等待 BE 节点关闭 [BeHost = 192.168.xx.xx]
[20220302-180148    INFO] BE 节点已成功关闭 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180148    INFO] 正在等待 BE 节点关闭 [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] BE 节点已成功关闭 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180149    INFO] 正在等待 BE 节点关闭 [BeHost = 192.168.xx.xx]
[20220302-180149    INFO] BE 节点已成功关闭 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
```

### 停止集群中特定角色节点

- 通过以下命令停止特定集群中 FE 节点。

```shell
./stargo cluster stop <cluster_name> --role FE
```

- 通过以下命令停止特定集群中 BE 节点。

```shell
./stargo cluster stop <cluster_name> --role BE
```

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster stop sr-c1 --role BE
[20220302-180624  OUTPUT] 停止集群 [集群名称 = sr-c1]
[20220302-180624  OUTPUT] 停止集群 sr-c1
[20220302-180624    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180624    INFO] BE 节点成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180624    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] BE 节点成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
[20220302-180625    INFO] BE 节点成功停止 [BeHost = 192.168.xx.xx, HeartbeatServicePort = 9050]
[20220302-180625    INFO] 正在停止 BE 集群 ...

###########################################################################

[sr-dev@nd1 sr-controller]$ ./stargo cluster stop sr-c1 --role FE
[20220302-180849  OUTPUT] 停止集群 [集群名称 = sr-c1]
[20220302-180849    INFO] 正在停止 FE 集群 ....
[20220302-180849  OUTPUT] 停止集群 sr-c1
[20220302-180849    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180851  OUTPUT] FE 节点成功停止 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220302-180851    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180854  OUTPUT] FE 节点成功停止 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220302-180854    INFO] 正在等待停止 FE 节点 [FeHost = 192.168.xx.xx]
[20220302-180856  OUTPUT] FE 节点成功停止 [主机 = 192.168.xx.xx, 查询端口 = 9030]
```

### 停止集群中特定节点

通过以下命令停止集群的某一个节点。

```shell
./stargo cluster stop <集群名称> --node <节点_ID>
```

您可以通过[查看指定集群信息](#查看指定集群信息)查看集群中特定节点的 ID。

示例：

```plain text
[root@nd1 sr-controller]# ./stargo cluster display sr-c1
[20220303-185400  OUTPUT] 显示集群 [集群名称 = sr-c1]
集群名称 = sr-c1
[20220303-185400    WARN] 所有 FE 节点都已关闭，请启动 FE 节点并再次显示集群状态。
ID                          角色    主机                  端口             状态        数据目录                                           部署目录
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        关闭        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        关闭        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        关闭        StarRocks/fe                                   /dataStarRocks/fe/meta
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        关闭        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        关闭        StarRocks/be                                   /dataStarRocks/be/storage
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        关闭        StarRocks/be                                   /dataStarRocks/be/storage

[root@nd1 sr-controller]# ./stargo cluster stop sr-c1 --node 192.168.xx.xx:9060
[20220303-185510  OUTPUT] 停止集群 [集群名称 = sr-c1]
[20220303-185510    INFO] 正在停止 BE 节点。[BeHost = 192.168.xx.xx]
[20220303-185510    INFO] 正在等待停止 BE 节点 [BeHost = 192.168.xx.xx]
```

## 扩容集群

您可以通过 StarGo 扩容其管理的集群。

### 创建配置文件

根据以下 YAML 模板，创建扩容 StarRocks 集群的拓扑文件。您可以根据需求配置相应的 EF 和/或 BE 节点。具体配置项参考[参数配置](../administration/Configuration.md)。

```yaml
# 扩容 FE 节点。
fe_servers:
  - host: 192.168.xx.xx # 扩容 FE 节点的 IP 地址。
    ssh_port: 22
    http_port: 8030
    rpc_port: 9020
    query_port: 9030
    edit_log_port: 9010
    deploy_dir: StarRocks/fe
    meta_dir: StarRocks/fe/meta
    log_dir: StarRocks/fe/log
    priority_networks: 192.168.xx.xx/24 # 当机器有多个 IP 时，在当前配置项中为当前节点指定唯一 IP。
    config:
      sys_log_level: "INFO"
      sys_log_delete_age: "1d"

# 扩容 BE 节点。
be_servers:
  - host: 192.168.xx.xx # 扩容 BE 节点的 IP 地址。
    ssh_port: 22
    be_port: 9060
    webserver_port: 8040
    heartbeat_service_port: 9050
    brpc_port: 8060
    deploy_dir : StarRocks/be
    storage_dir: StarRocks/be/storage
    log_dir: StarRocks/be/log
    config:
      create_tablet_worker_count: 3
```

### 创建 SSH 互信

如果扩容节点是新节点，您需要为其配置 SSH 节点互信。详细详细操作参考[前提条件](#前提条件)。

### 创建部署目录（可选）

如果您在配置文件中设定的部署路径不存在，且您有创建该路径的权限，StarGo 将根据配置文件自动创建部署目录。如果路径已存在，请确保您有在该路径下拥有写入的权限。您也可以通过以下命令，在各部署节点分别创建部署路径。

- 在新增 FE 节点安装目录下上创建 **meta** 路径。

```shell
mkdir -p StarRocks/fe/meta
```

- 在新增 BE 节点安装目录下上创建 **storage** 路径。

```shell
mkdir -p StarRocks/be/storage
```

> 注意
>
> 请确保以上创建的路径与配置文件中的 `meta_dir` 和 `storage_dir` 相同。

### 扩容 StarRocks 集群

通过以下命令扩容集群。

```shell
./stargo cluster scale-out <集群名称> <拓扑文件>
```

示例：

```plain text
# 当前集群状态。
[root@nd1 sr-controller]# ./stargo cluster display sr-test       
[20220503-210047  OUTPUT] 显示集群 [集群名称 = sr-test]
集群名称 = sr-test
集群版本 = v2.0.1
ID                          角色    主机                  端口             状态        数据目录                                             部署目录                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          FE      192.168.xx.xx         9010/9030        运行中      /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9060          BE      192.168.xx.xx         9060/9050        运行中      /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    

# 扩容集群。
[sr-dev@nd1 sr-controller]$ ./stargo cluster scale-out sr-test sr-out.yaml
[20220503-213725  OUTPUT] 正在扩容集群。[集群名称 = sr-test]
[20220503-213731  输出] 预检查部署环境：
预检查前端：
IP                    ssh认证        元数据目录                       部署目录                         http端口         rpc端口          查询端口         编辑日志端口  
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过            通过                             通过                             通过            通过            通过            通过           

预检查后端：
IP                    ssh认证        存储目录                         部署目录                          网络服务端口       心跳端口          brpc端口         后端端口        
--------------------  ---------------  ------------------------------  ------------------------------  ---------------  ---------------  ---------------  ---------------
192.168.xx.xx         通过            通过                             通过                             通过            通过            通过            通过           


[20220503-213731  输出] 预检查成功。尊重
[20220503-213731  输出] 创建部署文件夹 ...
[20220503-213732  输出] 下载 StarRocks 包 & jdk ...
[20220503-213732    信息] 包已经存在 [文件名 = starrocks-2.0.1-quickstart.tar.gz, 文件大小 = 1227406189, 文件修改时间 = 2022-05-03 17:32:03.478661923 +0800 CST]
[20220503-213732  输出] 下载完成。
[20220503-213732  输出] 解压 StarRocks 包 & jdk ...
[20220503-213741    信息] tar文件 /home/sr-dev/.starrocks-controller/download/starrocks-2.0.1-quickstart.tar.gz 已解压至 /home/sr-dev/.starrocks-controller/download
[20220503-213837    信息] tar文件 /home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1.tar.gz 已解压至 /home/sr-dev/.starrocks-controller/download
[20220503-213837    信息] tar文件 /home/sr-dev/.starrocks-controller/download/jdk-8u301-linux-x64.tar.gz 已解压至 /home/sr-dev/.starrocks-controller/download
[20220503-213837  输出] 分发前端目录 ...
[20220503-213845    信息] 上传目录前端源目录 = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/fe] 至前端目标目录 = [StarRocks/fe] 至前端主机 = [192.168.xx.xx]
[20220503-213857    信息] 上传目录 JDK源目录 = [/home/sr-dev/.starrocks-controller/download/jdk1.8.0_301] 至 JDK目标目录 = [StarRocks/fe/jdk] 至前端主机 = [192.168.xx.xx]
[20220503-213857    信息] 修改 JAVA_HOME: 主机 = [192.168.xx.xx], 文件路径 = [StarRocks/fe/bin/start_fe.sh]
[20220503-213857  输出] 分发后端目录 ...
[20220503-213924    信息] 上传目录后端源目录 = [/home/sr-dev/.starrocks-controller/download/StarRocks-2.0.1/be] 至后端目标目录 = [StarRocks/be] 至后端主机 = [192.168.xx.xx]
[20220503-213924  输出] 修改前端和后端节点的配置 ...
############################################# 扩展前端集群 #############################################
############################################# 扩展前端集群 #############################################
[20220503-213925    信息] 启动追随者前端节点 [主机 = 192.168.xx.xx, 编辑日志端口 = 9010]
[20220503-213945    信息] 前端节点成功启动 [主机 = 192.168.xx.xx, 查询端口 = 9030]
[20220503-213945    信息] 列出所有前端状态：
                                        前端主机 = 192.168.xx.xx       前端查询端口 = 9030     前端状态 = 真

############################################# 启动后端集群 #############################################
############################################# 启动后端集群 #############################################
[20220503-213945    信息] 启动后端节点 [后端主机 = 192.168.xx.xx 心跳服务端口 = 9050]
[20220503-214016    信息] 后端节点成功启动 [主机 = 192.168.xx.xx, 心跳服务端口 = 9050]
[20220503-214016  输出] 列出所有后端状态：
                                        后端主机 = 192.168.xx.xx       后端心跳服务端口 = 9050      后端状态 = 真

# 扩容后集群状态。
[sr-dev@nd1 sr-controller]$ ./stargo cluster display sr-test 
[20220503-214302  输出] 显示集群 [集群名称 = sr-test]
集群名称 = sr-test
集群版本 = v2.0.1
ID                          角色    主机                  端口             状态        数据目录                                              部署目录                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          前端      192.168.xx.xx         9010/9030        运行中        /opt/starrocks-test/fe                              /opt/starrocks-test/fe/meta                       
192.168.xx.xx:9010          前端      192.168.xx.xx         9010/9030        运行中        StarRocks/fe                                   StarRocks/fe/meta                            
192.168.xx.xx:9060          后端      192.168.xx.xx         9060/9050        运行中        /opt/starrocks-test/be                              /opt/starrocks-test/be/storage                    
192.168.xx.xx:9060          后端      192.168.xx.xx         9060/9050        运行中        StarRocks/be                                   StarRocks/be/storage                         
```

## 缩容集群

通过以下命令缩容集群中特定节点。

```shell
./stargo cluster scale-in <cluster_name> --node <node_id>
```

您可以通过[查看指定集群信息](#查看指定集群信息)查看集群中特定节点的 ID。

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster display sr-c1
[20220505-145649  输出] 显示集群 [集群名称 = sr-c1]
集群名称 = sr-c1
集群版本 = v2.0.1
ID                          角色    主机                  端口             状态        数据目录                                              部署目录                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.xx.xx:9010          前端      192.168.xx.xx         9010/9030        运行中        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          前端      192.168.xx.xx         9010/9030        运行中        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9010          前端      192.168.xx.xx         9010/9030        运行中        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.xx.xx:9060          后端      192.168.xx.xx         9060/9050        运行中        StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          后端      192.168.xx.xx         9060/9050        运行中        StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.xx.xx:9060          后端      192.168.xx.xx         9060/9050        运行中        StarRocks/be                                   /dataStarRocks/be/storage                        
[sr-dev@nd1 sr-controller]$ ./stargo cluster scale-in sr-c1 --node 192.168.88.83:9010
[20220621-010553  输出] 缩容集群 [集群名称 = sr-c1, 节点 ID = 192.168.88.83:9010]
[20220621-010553    信息] 等待停止前端节点 [前端主机 = 192.168.88.83]
[20220621-010606  输出] 前端节点成功缩容。[集群名称 = sr-c1, 节点 ID = 192.168.88.83:9010]

[sr-dev@nd1 sr-controller]$ ./stargo cluster display sr-c1
[20220621-010623  输出] 显示集群 [集群名称 = sr-c1]
集群名称 = sr-c1
集群版本 = 
ID                          角色    主机                  端口             状态        数据目录                                              部署目录                                         
--------------------------  ------  --------------------  ---------------  ----------  --------------------------------------------------  --------------------------------------------------
192.168.88.84:9010          前端      192.168.xx.xx         9010/9030        运行中        StarRocks/fe                                   /dataStarRocks/fe/meta                           
192.168.88.85:9010          前端      192.168.xx.xx         9010/9030        运行中/负载中  StarRocks/fe                                   /dataStarRocks/fe/meta                           
```
192.168.88.83:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.84:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage                        
192.168.88.85:9060          BE      192.168.xx.xx         9060/9050        UP          StarRocks/be                                   /dataStarRocks/be/storage              
```

## 升级和降级集群

您可以通过 StarGo 升级或降级其管理的集群。

- 升级集群的命令如下。

```shell
./stargo cluster upgrade <cluster_name>  <target_version>
```

- 降级集群的命令如下。

```shell
./stargo cluster downgrade <cluster_name>  <target_version>
```

示例：

```plain text
[sr-dev@nd1 sr-controller]$ ./stargo cluster list
[20220515-195827  输出] 列出所有集群
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 19:35:36        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          
[sr-dev@nd1 sr-controller]$ ./stargo cluster upgrade sr-test2 v2.1.3
[20220515-200358  输出] 列出所有集群
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.1.3      test222     2022-05-15 20:03:01        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                          

[sr-dev@nd1 sr-controller]$ ./stargo cluster downgrade sr-test2 v2.0.1 
[sr-dev@nd1 sr-controller]$ ./stargo cluster list
[20220515-200915  输出] 列出所有集群
ClusterName      Version     User        CreateDate                 MetaPath                                                      PrivateKey                                        
---------------  ----------  ----------  -------------------------  ------------------------------------------------------------  --------------------------------------------------
sr-test2         v2.0.1      test222     2022-05-15 20:08:40        /home/sr-dev/.starrocks-controller/cluster/sr-test2           /home/sr-dev/.ssh/id_rsa                
```

## 相关命令

|命令|描述|
|----|----|
|deploy|部署集群。|
|start|启动集群。|
|stop|停止集群。|
|scale-in|集群缩容。|
|scale-out|集群扩容。|
|upgrade|升级集群。|
|downgrade|降级集群。|
|display|查看特定集群。|
|list|查看所有集群。|