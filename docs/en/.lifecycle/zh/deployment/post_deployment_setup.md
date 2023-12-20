---
displayed_sidebar: English
---

# 部署后设置

本主题介绍部署 StarRocks 后应执行的任务。

在将新的 StarRocks 集群投入生产之前，您必须保护初始账户并设置必要的变量和属性，以确保集群能够正常运行。

## 保护初始账户

StarRocks 集群创建后，会自动生成集群的初始 `root` 用户。`root` 用户被授予 `root` 权限，这是集群内所有权限的集合。我们建议您保护此用户账户并避免在生产环境中使用它，以防止滥用。

创建集群时，StarRocks 会自动为 `root` 用户分配一个空密码。请按照以下步骤为 `root` 用户设置新密码：

1. 使用用户名 `root` 和空密码通过 MySQL 客户端连接到 StarRocks。

   ```Bash
   # 将 <fe_address> 替换为您连接的 FE 节点的 IP 地址（priority_networks）或 FQDN，
   # 并将 <query_port> 替换为您在 fe.conf 中指定的查询端口（默认值：9030）。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 执行以下 SQL 重置 `root` 用户的密码：

   ```SQL
   -- 将 <password> 替换为您想要分配给 root 用户的密码。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注意**
- 重置密码后请妥善保管。如果您忘记了密码，请参阅[重置丢失的 root 密码](../administration/User_privilege.md#reset-lost-root-password)以获取详细说明。
- 完成部署后设置后，您可以创建新用户和角色来管理团队内的权限。请参阅[管理用户权限](../administration/User_privilege.md)以获取详细说明。

## 设置必要的系统变量

为了让您的 StarRocks 集群在生产环境中正常工作，您需要设置以下系统变量：

|**变量名称**|**StarRocks 版本**|**推荐值**|**描述**|
|---|---|---|---|
|is_report_success|v2.4 或更早版本|false|控制是否发送查询 profile 以进行分析的布尔开关。默认值为 `false`，这意味着不需要 profile。将此变量设置为 `true` 会影响 StarRocks 的并发性。|
|enable_profile|v2.5 或更高版本|false|控制是否发送查询 profile 以进行分析的布尔开关。默认值为 `false`，这意味着不需要 profile。将此变量设置为 `true` 会影响 StarRocks 的并发性。|
|enable_pipeline_engine|v2.3 或更高版本|true|控制是否启用 pipeline 执行引擎的布尔开关。`true` 表示启用，`false` 表示相反。默认值：`true`。|
|parallel_fragment_exec_instance_num|v2.3 或更高版本|如果启用了 pipeline 引擎，则可以将此变量设置为 `1`。如果未启用 pipeline 引擎，则应将其设置为 CPU 核心数的一半。|用于扫描每个 BE 上的节点的实例数。默认值为 `1`。|
|pipeline_dop|v2.3、v2.4、v2.5|0|pipeline 实例的并行度，用于调整查询并发度。默认值：`0`，表示系统自动调整每个 pipeline 实例的并行度。<br />从 v3.0 起，StarRocks 会根据查询并发度自适应调整此参数。|

- 全局将 `is_report_success` 设置为 `false`：

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- 全局将 `enable_profile` 设置为 `false`：

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- 全局将 `enable_pipeline_engine` 设置为 `true`：

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- 全局将 `parallel_fragment_exec_instance_num` 设置为 `1`：

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- 全局将 `pipeline_dop` 设置为 `0`：

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

有关系统变量的更多信息，请参阅[系统变量](../reference/System_variable.md)。

## 设置用户属性

如果您在集群中创建了新用户，您需要增加他们的最大连接数（例如，设置为 `1000`）：

```SQL
-- 将 <username> 替换为您想要增加最大连接数的用户名。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 下一步做什么

在部署和设置 StarRocks 集群之后，您可以继续设计最适合您场景的表。有关设计表的详细说明，请参阅[理解 StarRocks 表设计](../table_design/Table_design.md)。