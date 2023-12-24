---
displayed_sidebar: English
---

# 启用 FQDN 访问

本主题描述如何通过使用完全限定域名（FQDN）来启用集群访问。FQDN 是指可以通过互联网访问的特定实体的完整域名。FQDN 由两部分组成：主机名和域名。

在 2.4 版本之前，StarRocks仅支持通过 IP 地址访问 FE 和 BE。即使使用 FQDN 将节点添加到集群中，最终也会被转换为 IP 地址。这给数据库管理员带来了很大的不便，因为更改StarRocks集群中某些节点的IP地址可能会导致节点访问失败。在 2.4 版本中，StarRocks将每个节点与其IP地址解耦。现在，您可以仅通过它们的FQDN来管理StarRocks中的节点。

## 先决条件

要为StarRocks集群启用FQDN访问，请确保满足以下要求：

- 集群中的每台机器都必须有一个主机名。

- 在每台机器的 **/etc/hosts** 文件中，必须指定集群中其他机器的相应IP地址和FQDN。

- **/etc/hosts** 文件中的IP地址必须是唯一的。

## 使用具有FQDN访问权限的新集群

默认情况下，新集群中的FE节点是通过IP地址访问启动的。若要启动具有FQDN访问权限的新集群，您必须在首次启动集群时通过运行以下命令来启动FE节点：

```Shell
./bin/start_fe.sh --host_type FQDN --daemon
```

属性 `--host_type` 指定了用于启动节点的访问方法。有效值包括 `FQDN` 和 `IP`。首次启动节点时，只需指定此属性一次。

有关如何安装StarRocks的详细说明，请参见[部署StarRocks](../deployment/deploy_manually.md)。

每个BE节点都使用FE元数据中定义的BE地址来标识自己。因此，在启动BE节点时，无需指定 `--host_type`。如果`BE Address`定义了具有FQDN的BE节点，则该BE节点将使用此FQDN来标识自己。

## 在现有集群中启用FQDN访问

要在之前通过IP地址启动的现有集群中启用FQDN访问，您首先需要将StarRocks升级到2.4.0或更高版本。

### 为FE节点启用FQDN访问

在启用FE节点的FQDN访问之前，您需要先为所有非Leader Follower FE节点启用FQDN访问，然后再为Leader FE节点启用FQDN访问。

> **注意**
>
> 在为FE节点启用FQDN访问之前，请确保集群至少具有三个Follower FE节点。

#### 为非Leader Follower FE节点启用FQDN访问

1. 进入FE节点的部署目录，并运行以下命令停止FE节点：

    ```Shell
    ./bin/stop_fe.sh --daemon
    ```

2. 通过MySQL客户端执行以下语句，检查您已停止的FE节点的 `Alive` 状态。等待 `Alive` 状态变为 `false`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

3. 执行以下语句，将IP地址替换为FQDN。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 运行以下命令以使用FQDN访问权限启动FE节点。

    ```Shell
    ./bin/start_fe.sh --host_type FQDN --daemon
    ```

    属性 `--host_type` 指定了用于启动节点的访问方法。有效值包括 `FQDN` 和 `IP`。在修改节点后重新启动节点时，只需指定此属性一次。

5. 检查FE节点的 `Alive` 状态。等待 `Alive` 状态变为 `true`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

6. 当当前FE节点的 `Alive` 状态为 `true` 时，依次为其他非Leader Follower FE节点启用FQDN访问，重复上述步骤。

#### 为Leader FE节点启用FQDN访问

成功修改并重新启动所有非Leader FE节点后，您现在可以为Leader FE节点启用FQDN访问。

> **注意**
>
> 在Leader FE节点启用FQDN访问之前，用于向集群添加节点的FQDN仍会转换为相应的IP地址。为集群选择启用了FQDN访问的Leader FE节点后，FQDN将不会转换为IP地址。

1. 进入Leader FE节点的部署目录，并运行以下命令停止Leader FE节点。

    ```Shell
    ./bin/stop_fe.sh --daemon
    ```

2. 通过MySQL客户端执行以下语句，检查集群是否选择了新的Leader FE节点。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

    任何具有 `Alive` 状态和 `isMaster` 为 `true` 的FE节点都是正在运行的Leader FE。

3. 执行以下语句，将IP地址替换为FQDN。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 运行以下命令以使用FQDN访问权限启动FE节点。

    ```Shell
    ./bin/start_fe.sh --host_type FQDN --daemon
    ```

    属性 `--host_type` 指定了用于启动节点的访问方法。有效值包括 `FQDN` 和 `IP`。在修改节点后重新启动节点时，只需指定此属性一次。

5. 检查FE节点的 `Alive` 状态。

    ```Plain
    SHOW PROC '/frontends'\G
    ```

    如果 `Alive` 状态变为 `true`，则FE节点修改成功，并作为Follower FE节点添加到集群中。

### 为BE节点启用FQDN访问

通过MySQL客户端执行以下语句，将IP地址替换为FQDN，以启用BE节点的FQDN访问。

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_ip>" TO "<be_hostname>";
```

> **注意**
>
> 启用FQDN访问后，无需重新启动BE节点。

## 回滚

要将启用了FQDN访问的StarRocks集群回滚到不支持FQDN访问的早期版本，您需要先为集群中的所有节点启用IP地址访问。您可以参考[在现有集群中启用FQDN访问](#enable-fqdn-access-in-an-existing-cluster)中的一般指南，只需将SQL命令更改为以下命令：

- 为FE节点启用IP地址访问：

```SQL
ALTER SYSTEM MODIFY FRONTEND HOST "<fe_hostname>" TO "<fe_ip>";
```

- 为BE节点启用IP地址访问：

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_hostname>" TO "<be_ip>";
```

修改将在集群成功重启后生效。

## 常见问题

**问：为FE节点启用FQDN访问时出现错误：“需要1个副本。但没有人与这位大师一起活跃”。我该怎么办？**

答：在为FE节点启用FQDN访问之前，请确保集群至少具有三个Follower FE节点。

**问：在启用了FQDN访问的集群中，我可以使用IP地址将新节点添加到集群吗？**

答：可以。
