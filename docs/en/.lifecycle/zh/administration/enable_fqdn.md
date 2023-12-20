---
displayed_sidebar: English
---

# 启用 FQDN 访问

本主题介绍如何使用**完全限定域名（FQDN）**启用集群访问。FQDN 是可以通过互联网访问的特定实体的完整域名。FQDN 由两部分组成：主机名和域名。

在 2.4 版本之前，StarRocks 仅支持通过 IP 地址访问 FE 和 BE。即使使用 FQDN 将节点添加到集群，最终也会转换为 IP 地址。这给 DBA 带来了极大的不便，因为更改 StarRocks 集群中某些节点的 IP 地址可能会导致无法访问这些节点。在 2.4 版本中，StarRocks 将每个节点与其 IP 地址解耦。现在，您可以仅通过 FQDN 管理 StarRocks 中的节点。

## 先决条件

要为 StarRocks 集群启用 FQDN 访问，请确保满足以下要求：

- 集群中的每台机器都必须有一个主机名。

- 在每台机器上的 **/etc/hosts** 文件中，必须指定集群中其他机器的对应 IP 地址和 FQDN。

- **/etc/hosts** 文件中的 IP 地址必须是唯一的。

## 设置具有 FQDN 访问的新集群

默认情况下，新集群中的 FE 节点通过 IP 地址访问启动。要启动具有 FQDN 访问的新集群，您必须在**首次启动集群时**运行以下命令来启动 FE 节点：

```Shell
./bin/start_fe.sh --host_type FQDN --daemon
```

属性 `--host_type` 指定用于启动节点的访问方法。有效值包括 `FQDN` 和 `IP`。您只需在首次启动节点时指定该属性一次。

有关如何安装 StarRocks 的详细说明，请参阅[部署 StarRocks](../deployment/deploy_manually.md)。

每个 BE 节点使用在 FE 元数据中定义的 `BE Address` 来标识自己。因此，启动 BE 节点时不需要指定 `--host_type`。如果 `BE Address` 使用 FQDN 定义了 BE 节点，则 BE 节点将使用此 FQDN 来标识自己。

## 在现有集群中启用 FQDN 访问

要在之前通过 IP 地址启动的现有集群中启用 FQDN 访问，您必须首先将 StarRocks 升级到 2.4.0 版本或更高版本。

### 为 FE 节点启用 FQDN 访问

您需要先为所有非领导者 Follower FE 节点启用 FQDN 访问，然后再为领导者 Leader FE 节点启用。

> **警告**
> 在为 FE 节点启用 FQDN 访问之前，请确保集群至少有三个 Follower FE 节点。

#### 为非领导者 Follower FE 节点启用 FQDN 访问

1. 进入 FE 节点的部署目录，执行以下命令停止 FE 节点：

   ```Shell
   ./bin/stop_fe.sh --daemon
   ```

2. 通过 MySQL 客户端执行以下语句，检查您已停止的 FE 节点的 `Alive` 状态。等待直到 `Alive` 状态变为 `false`。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

3. 执行以下语句将 IP 地址替换为 FQDN。

   ```SQL
   ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
   ```

4. 运行以下命令以启动具有 FQDN 访问权限的 FE 节点。

   ```Shell
   ./bin/start_fe.sh --host_type FQDN --daemon
   ```

   属性 `--host_type` 指定用于启动节点的访问方法。有效值包括 `FQDN` 和 `IP`。在您修改节点后重新启动节点时，只需指定此属性一次。

5. 检查 FE 节点的 `Alive` 状态。等待直到 `Alive` 状态变为 `true`。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

6. 当当前 FE 节点的 `Alive` 状态为 `true` 时，重复上述步骤，依次为其他非领导者 Follower FE 节点启用 FQDN 访问。

#### 为领导者 Leader FE 节点启用 FQDN 访问

在所有非领导者 FE 节点都已修改并成功重启后，您现在可以为领导者 Leader FE 节点启用 FQDN 访问。

> **注意**
> 在领导者 Leader FE 节点启用 FQDN 访问之前，用于向集群添加节点的 FQDN 仍会转换为相应的 IP 地址。在集群选出启用 FQDN 访问的领导者 Leader FE 节点后，FQDN 将不会转换为 IP 地址。

1. 进入领导者 Leader FE 节点的部署目录，执行以下命令停止领导者 Leader FE 节点。

   ```Shell
   ./bin/stop_fe.sh --daemon
   ```

2. 通过 MySQL 客户端执行以下语句，检查集群是否已选举出新的领导者 Leader FE 节点。

   ```SQL
   SHOW PROC '/frontends'\G
   ```

   任何状态为 `Alive` 且 `isMaster` 为 `true` 的 FE 节点都是正在运行的领导者 Leader FE。

3. 执行以下语句将 IP 地址替换为 FQDN。

   ```SQL
   ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
   ```

4. 运行以下命令以启动具有 FQDN 访问权限的领导者 Leader FE 节点。

   ```Shell
   ./bin/start_fe.sh --host_type FQDN --daemon
   ```

   属性 `--host_type` 指定用于启动节点的访问方法。有效值包括 `FQDN` 和 `IP`。在您修改节点后重新启动节点时，只需指定此属性一次。

5. 检查领导者 Leader FE 节点的 `Alive` 状态。

   ```Plain
   SHOW PROC '/frontends'\G
   ```

如果 `Alive` 状态变为 `true`，则领导者 Leader FE 节点修改成功，并作为 Follower FE 节点添加到集群中。

### 为 BE 节点启用 FQDN 访问

通过 MySQL 客户端执行以下语句，将 IP 地址替换为 FQDN，以启用 BE 节点的 FQDN 访问。

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_ip>" TO "<be_hostname>";
```

> **注意**
> 启用 FQDN 访问后，**无需**重新启动 BE 节点。

## 回滚

要将支持 FQDN 访问的 StarRocks 集群回滚到不支持 FQDN 访问的早期版本，必须首先为集群中的所有节点启用 IP 地址访问。您可以参考[在现有集群中启用 FQDN 访问](#enable-fqdn-access-in-an-existing-cluster)的一般指导，但需要将 SQL 命令更改为以下命令：

- 为 FE 节点启用 IP 地址访问：

```SQL
ALTER SYSTEM MODIFY FRONTEND HOST "<fe_hostname>" TO "<fe_ip>";
```

- 为 BE 节点启用 IP 地址访问：

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_hostname>" TO "<be_ip>";
```

集群成功重启后，修改将生效。

## 常见问题解答

**问：当我为 FE 节点启用 FQDN 访问时，出现错误：“需要 1 个副本。但该主节点没有活动的副本”。我应该怎么办？**

答：在为 FE 节点启用 FQDN 访问之前，请确保集群至少有三个 Follower FE 节点。

**问：我可以使用 IP 地址向已启用 FQDN 访问的集群添加新节点吗？**

答：可以。