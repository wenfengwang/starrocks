---
displayed_sidebar: English
---

# 部署后设置

本主题描述了部署 StarRocks 后应执行的任务。

在将新的 StarRocks 集群投入生产之前，您必须保护初始帐户并设置必要的变量和属性，以允许集群正常运行。

## 保护初始帐户

在创建 StarRocks 集群时，集群的初始`root`用户会自动生成。`root`用户被授予`root`权限，即在集群中拥有所有权限的集合。我们建议您保护此用户帐户，并避免在生产环境中使用它，以防止误用。

在创建集群时，StarRocks会自动为`root`用户分配空密码。请按以下步骤为`root`用户设置新密码：

1. 使用用户名`root`和空密码通过您的 MySQL 客户端连接到 StarRocks。

   ```Bash
   # 用您连接到的 FE 节点的 IP 地址（priority_networks）或 FQDN 替换 <fe_address>，并替换 <query_port> 为 fe.conf 中指定的查询端口（默认值：9030）。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下 SQL 重置`root`用户的密码：

   ```SQL
   -- 用您要分配给 root 用户的密码替换 <password>。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注意**
>
> - 重置密码后，请妥善保管密码。如果忘记密码，请参阅[重置丢失的 root 密码](../administration/User_privilege.md#reset-lost-root-password)获取详细说明。
> - 完成部署后设置后，您可以创建新用户和角色来管理团队中的权限。有关详细说明，请参阅[管理用户权限](../administration/User_privilege.md)。

## 设置必要的系统变量

为了使您的 StarRocks 集群在生产环境中正常工作，您需要设置以下系统变量：

| **变量名称**                   | **StarRocks 版本** | **推荐值**                                        | **描述**                                              |
| ----------------------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| is_report_success                   | v2.4 或更早版本       | false                                                        | 控制是否发送查询配置文件进行分析的布尔开关。默认值为 `false`，表示不需要配置文件。将此变量设置为 `true`会影响 StarRocks 的并发性。 |
| enable_profile                      | v2.5 或更新版本         | false                                                        | 控制是否发送查询配置文件进行分析的布尔开关。默认值为 `false`，表示不需要配置文件。将此变量设置为 `true`会影响 StarRocks 的并发性。 |
| enable_pipeline_engine              | v2.3 或更新版本         | true                                                         | 控制是否启用管道执行引擎的布尔开关。`true`表示已启用，`false`表示相反。默认值：`true`。 |
| parallel_fragment_exec_instance_num | v2.3 或更新版本         | 如果已启用管道引擎，则可以将此变量设置为 `1`。如果尚未启用管道引擎，则应将其设置为 CPU 核心数的一半。 | 用于扫描每个 BE 上的节点的实例数。默认值为 `1`。 |
| pipeline_dop                        | v2.3、v2.4 和 v2.5  | 0                                                            | 流水线实例的并行度，用于调整查询并发。默认值： `0`，表示系统自动调整每个流水线实例的并行度。<br />从 v3.0 开始，StarRocks 会根据查询并行度自适应调整该参数。 |

- 全局将`is_report_success`设置为`false`：

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- 全局将`enable_profile`设置为`false`：

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- 全局将`enable_pipeline_engine`设置为`true`：

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- 全局将`parallel_fragment_exec_instance_num`设置为`1`：

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- 全局将`pipeline_dop`设置为`0`：

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

有关系统变量的更多信息，请参阅[系统变量](../reference/System_variable.md)。

## 设置用户属性

如果在集群中创建了新用户，您需要将其最大连接数扩大（例如，到`1000`）：

```SQL
-- 用要扩大最大连接数的用户名替换 <username>。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 下一步操作

部署和设置 StarRocks 集群后，您可以继续设计最适合您场景的表。有关设计表的详细说明，请参阅[了解 StarRocks 表设计](../table_design/Table_design.md)。
