---
displayed_sidebar: "Chinese"
---

# 启用 FQDN 访问

本主题介绍了如何通过使用完全限定域名（FQDN）启用集群访问。FQDN 是指可通过互联网访问的特定实体的**完整域名**。FQDN 由两部分组成：主机名和域名。

在 2.4 版本之前，StarRocks 仅支持通过 IP 地址访问 FE 和 BE。即使在向集群添加节点时使用 FQDN，最终也会将其转换为 IP 地址。这给数据库管理员带来了很大的不便，因为更改 StarRocks 集群中某些节点的 IP 地址可能会导致无法访问这些节点。在 2.4 版本中，StarRocks 将每个节点与其 IP 地址解耦。现在，您可以仅通过其 FQDN 管理 StarRocks 中的节点。

## 先决条件

要为 StarRocks 集群启用 FQDN 访问，请确保满足以下要求：

- 集群中的每台机器必须具有主机名。

- 在每台机器的 **/etc/hosts** 文件中，您必须指定集群中其他机器的对应 IP 地址和 FQDN。

- **/etc/hosts** 文件中的 IP 地址必须是唯一的。

## 使用 FQDN 访问设置新集群

默认情况下，新集群中的 FE 节点是通过 IP 地址访问的。要以 FQDN 访问启动新集群，您必须在第一次启动集群时运行以下命令来启动 FE 节点：

```Shell
./bin/start_fe.sh --host_type FQDN --daemon
```

属性 `--host_type` 指定启动节点所使用的访问方法。有效值包括 `FQDN` 和 `IP`。当您首次启动节点时只需要指定此属性一次。

有关如何安装 StarRocks 的详细说明，请参见 [部署 StarRocks](../deployment/deploy_manually.md)。

每个 BE 节点使用 FE 元数据中定义的 `BE Address` 进行标识。因此，在启动 BE 节点时，**无需**指定 `--host_type`。如果 `BE Address` 定义了具有 FQDN 的 BE 节点，则该 BE 节点将以该 FQDN 进行标识。

## 在现有集群中启用 FQDN 访问

要在先前通过 IP 地址启动的现有集群中启用 FQDN 访问，您首先必须**升级** StarRocks 版本至 2.4.0 或更高版本。

### 为 FE 节点启用 FQDN 访问

在启用 Leader FE 节点的 FQDN 访问之前，您需要先为所有非 Leader Follower FE 节点启用 FQDN 访问。

> **注意**
>
> 确保在为 FE 节点启用 FQDN 访问之前，集群至少具有三个 Follower FE 节点。

#### 为非 Leader Follower FE 节点启用 FQDN 访问

1. 转到 FE 节点的部署目录，并运行以下命令停止 FE 节点：

    ```Shell
    ./bin/stop_fe.sh --daemon
    ```

2. 通过 MySQL 客户端执行以下语句，检查您已停止的 FE 节点的 `Alive` 状态。等待直到 `Alive` 状态变为 `false`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

3. 执行以下语句，将 IP 地址替换为 FQDN。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 运行以下命令以启用具有 FQDN 访问的 FE 节点。

    ```Shell
    ./bin/start_fe.sh --host_type FQDN --daemon
    ```

    属性 `--host_type` 指定启动节点所使用的访问方法。有效值包括 `FQDN` 和 `IP`。您只需要在修改节点后重新启动节点时指定此属性一次。

5. 检查 FE 节点的 `Alive` 状态。等待直到 `Alive` 状态变为 `true`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

6. 在当前 FE 节点的 `Alive` 状态为 `true` 时，依次为其他非 Leader Follower FE 节点启用 FQDN 访问，即重复上述步骤。

#### 为 Leader FE 节点启用 FQDN 访问

在所有非 Leader FE 节点已成功修改和重新启动后，您现在可以为 Leader FE 节点启用 FQDN 访问。

> **注意**
>
> 在为 Leader FE 节点启用 FQDN 访问之前，用于向集群添加节点的 FQDN 仍然会转换为相应的 IP 地址。启用 FQDN 访问后选举出用于集群的 Leader FE 节点，此时 FQDN 将不再转换为 IP 地址。

1. 转到 Leader FE 节点的部署目录，并运行以下命令停止 Leader FE 节点。

    ```Shell
    ./bin/stop_fe.sh --daemon
    ```

2. 通过 MySQL 客户端执行以下语句，以检查集群是否已选举出新的 Leader FE 节点。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

    具有 `Alive` 和 `isMaster` 状态为 `true` 的任何 FE 节点都是正在运行的 Leader FE。

3. 执行以下语句，将 IP 地址替换为 FQDN。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 运行以下命令以启用具有 FQDN 访问的 FE 节点。

    ```Shell
    ./bin/start_fe.sh --host_type FQDN --daemon
    ```

    属性 `--host_type` 指定启动节点所使用的访问方法。有效值包括 `FQDN` 和 `IP`。您只需要在修改节点后重新启动节点时指定此属性一次。

5. 检查 FE 节点的 `Alive` 状态。

    ```Plain
    SHOW PROC '/frontends'\G
    ```

  如果 `Alive` 状态变为 `true`，则 FE 节点已成功修改并作为 Follower FE 节点添加到集群中。

### 为 BE 节点启用 FQDN 访问

通过 MySQL 客户端执行以下语句，将 IP 地址替换为 FQDN 以启用 BE 节点的 FQDN 访问。

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_ip>" TO "<be_hostname>";
```

> **注意**
>
> 在启用 FQDN 访问后，无需重新启动 BE 节点。

## 回滚

要将启用了 FQDN 访问的 StarRocks 集群回滚至不支持 FQDN 访问的早期版本，您必须先为集群中的所有节点启用 IP 地址访问。您可以参考[为现有集群启用 FQDN 访问](#为现有集群启用-fqdn-访问)获得一般指导，只是需要将 SQL 命令更改为以下命令：

- 为 FE 节点启用 IP 地址访问：

```SQL
ALTER SYSTEM MODIFY FRONTEND HOST "<fe_hostname>" TO "<fe_ip>";
```

- 为 BE 节点启用 IP 地址访问：

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_hostname>" TO "<be_ip>";
```

修改在集群成功重新启动后生效。

## 常见问题

**Q: 为 FE 节点启用 FQDN 访问时出现错误："required 1 replica. But none were active with this master"。应该怎么办？**

A: 在为 FE 节点启用 FQDN 访问之前，请确保集群至少具有三个 Follower FE 节点。

**Q: 在启用了 FQDN 访问的集群中，我可以使用 IP 地址向集群添加新节点吗？**

A: 可以。