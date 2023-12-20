---
displayed_sidebar: English
---

# 启用 FQDN 访问

本主题介绍如何通过使用**完全限定域名（FQDN）**来启用集群访问。FQDN 是一个特定实体的**完整域名**，可以通过互联网访问。FQDN 由两部分组成：主机名和域名。

在 2.4 版本之前，StarRocks 仅支持通过 IP 地址来访问 FE（前端）和 BE（后端）。即便是使用 FQDN 添加节点到集群，最终它也会被转换成 IP 地址。这给数据库管理员（DBA）带来了极大的不便，因为更改 StarRocks 集群中某些节点的 IP 地址可能会导致无法访问这些节点。在 2.4 版本中，StarRocks 将节点与其 IP 地址解耦。您现在可以只通过 FQDN 来管理 StarRocks 中的节点。

## 先决条件

要为 StarRocks 集群启用 FQDN 访问，需确保满足以下条件：

- 集群中的每台机器都必须有一个主机名。

- 在每台机器的 **/etc/hosts** 文件中，必须指定集群中其他机器的对应 IP 地址和 FQDN。

- /etc/hosts 文件中的 IP 地址必须是**唯一的**。

## 设置启用 FQDN 访问的新集群

默认情况下，新集群中的 FE 节点是通过 IP 地址访问来启动的。要启动启用 FQDN 访问的新集群，您必须在首次启动集群时运行以下命令**当您首次启动集群时**来启动 FE 节点：

```Shell
./bin/start_fe.sh --host_type FQDN --daemon
```

属性 --host_type 指定了启动节点时使用的访问方法。有效值包括 FQDN 和 IP。您仅需在首次启动节点时指定此属性一次。

查看[部署 StarRocks](../deployment/deploy_manually.md)，了解如何安装 StarRocks 的详细说明。

每个 BE 节点通过 FE 元数据中定义的 BE 地址来识别自身。因此，启动 BE 节点时不需要指定 --host_type。如果 BE 地址定义了一个使用 FQDN 的 BE 节点，那么 BE 节点将以此 FQDN 来识别自己。

## 在现有集群中启用 FQDN 访问

要在原先通过 IP 地址启动的现有集群中启用 FQDN访问，您必须首先将**StarRocks**升级至 2.4.0 版本或更新版本。

### 为 FE 节点启用 FQDN 访问

您需要先为所有非领导者 Follower FE 节点启用 FQDN 访问，然后再为领导者 Leader FE 节点启用 FQDN 访问。

> **注意**
> 在为 FE 节点启用 FQDN访问之前，请确保集群中至少有三个**Follower FE**节点。

#### 为非领导者 Follower FE 节点启用 FQDN 访问

1. 进入 FE 节点的部署目录，执行以下命令以停止 FE 节点：

   ```Shell
   ./bin/stop_fe.sh --daemon
   ```

2. 通过 MySQL 客户端执行以下语句，检查您已停止的 FE 节点的 Alive 状态。等待直到 Alive 状态变为 false。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

3. 执行以下语句以将 IP 地址替换为 FQDN。

   ```SQL
   ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
   ```

4. 运行以下命令以 FQDN 访问方式启动 FE 节点。

   ```Shell
   ./bin/start_fe.sh --host_type FQDN --daemon
   ```

   属性 --host_type 指定了启动节点时使用的访问方法。有效值包括 FQDN 和 IP。在修改节点后重新启动节点时，您只需指定此属性一次。

5. 检查 FE 节点的 Alive 状态。等待直到 Alive 状态变为 true。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

6. 当当前 FE 节点的 Alive 状态为 true 时，重复上述步骤，逐个为其他非领导者 Follower FE 节点启用 FQDN 访问。

#### 为领导者 Leader FE 节点启用 FQDN 访问

在所有非领导者 FE 节点都已修改并成功重启后，您现在可以为领导者 Leader FE 节点启用 FQDN 访问。

> **备注**
> 在领导者 **Leader FE** 节点启用 **FQDN** 访问之前，用于添加到集群的节点的 **FQDN** 仍然会被转换成相应的 IP 地址。在集群选出启用了 **FQDN** 访问的领导者 **Leader FE** 节点后，**FQDN** 将不再被转换成 IP 地址。

1. 进入领导者 Leader FE 节点的部署目录，执行以下命令以停止领导者 Leader FE 节点。

   ```Shell
   ./bin/stop_fe.sh --daemon
   ```

2. 通过 MySQL 客户端执行以下语句，检查集群是否已选出新的领导者 Leader FE 节点。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   任何状态为 Alive 并且 isMaster 为 true 的 FE 节点即为正在运行的领导者 Leader FE。

3. 执行以下语句以将 IP 地址替换为 FQDN。

   ```SQL
   ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
   ```

4. 运行以下命令以 FQDN 访问方式启动 FE 节点。

   ```Shell
   ./bin/start_fe.sh --host_type FQDN --daemon
   ```

   属性 --host_type 指定了启动节点时使用的访问方法。有效值包括 FQDN 和 IP。在修改节点后重新启动节点时，您只需指定此属性一次。

5. 检查 FE 节点的 Alive 状态。

   ```Plain
   SHOW PROC '/frontends'\G
   ```

如果 Alive 状态变为 true，则表示 FE 节点已成功修改，并作为 Follower FE 节点被添加到集群中。

### 为 BE 节点启用 FQDN 访问

通过 MySQL 客户端执行以下语句，将 IP 地址替换为 FQDN，以启用 BE 节点的 FQDN 访问。

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_ip>" TO "<be_hostname>";
```

> **备注**
> 启用 FQDN访问后，**无需重启** BE节点。

## 回滚操作

若需将启用了 **FQDN** 访问的 StarRocks 集群回滚至不支持 **FQDN** 访问的早期版本，您必须首先为集群中的所有节点启用 IP 地址访问。您可以参考 [在现有集群中启用 FQDN 访问](#enable-fqdn-access-in-an-existing-cluster) 的一般指导，但需要将 SQL 命令更改为以下内容：

- 为 FE 节点启用 IP 地址访问：

```SQL
ALTER SYSTEM MODIFY FRONTEND HOST "<fe_hostname>" TO "<fe_ip>";
```

- 为 BE 节点启用 IP 地址访问：

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_hostname>" TO "<be_ip>";
```

修改在集群成功重启后生效。

## 常见问题解答

**问: 当我为 FE 节点启用 FQDN 访问时出现错误: "需要 1 个副本。但该主节点没有活动的副本"。我该怎么办？**

答：在为 FE 节点启用 FQDN 访问之前，请确保集群至少有三个 Follower FE 节点。

**问: 我可以通过使用 IP 地址向已启用 FQDN 访问的集群添加新节点吗?**

答：可以。
