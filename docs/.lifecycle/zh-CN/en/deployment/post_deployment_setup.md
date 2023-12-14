---
displayed_sidebar: "Chinese"
---

# 部署后设置

本主题描述了在部署 StarRocks 后应执行的任务。

在将新 StarRocks 集群投入生产之前，您必须确保初始帐户的安全，并设置必要的变量和属性，以确保集群正常运行。

## 确保初始帐户的安全

在创建 StarRocks 集群时，集群的初始`root`用户会自动生成。`root`用户被授予`root`权限，即在集群中拥有所有权限的集合。我们建议您保护此用户帐户，并避免在生产中使用它，以防止被滥用。

当集群创建时，StarRocks 会自动为`root`用户分配一个空密码。按照以下步骤为`root`用户设置新密码：

1. 使用用户名`root`和空密码通过您的 MySQL 客户端连接到 StarRocks。

   ```Bash
   # 用 <fe_address> 替换您连接的 FE 节点的 IP 地址（priority_networks）或 FQDN，用 <query_port> 替换您在 fe.conf 中指定的 query_port（默认值：9030）。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 通过执行以下 SQL 重置`root`用户的密码：

   ```SQL

   -- 用您要分配给root用户的密码替换 <password>。
   SET PASSWORD = PASSWORD('<password>')
   ```


> **注意**
>
> - 重设密码后要妥善保管密码。如果忘记了密码，请参阅[重置丢失的root密码](../administration/User_privilege.md#reset-lost-root-password)获取详细说明。
> - 完成部署后设置后，您可以创建新用户和角色来管理团队内的权限。查看[管理用户权限](../administration/User_privilege.md)获取详细说明。

## 设置必要的系统变量

为了使您的StarRocks集群在生产环境中正常工作，您需要设置以下系统变量：

| **变量名称**                        | **StarRocks 版本** | **推荐值**           | **描述**                                                   |
| ----------------------------------- | ------------------- | -------------------- | ------------------------------------------------------------ |
| is_report_success                   | v2.4或更早版本       | false                | 控制是否发送查询的分析配置文件的布尔开关。默认值为`false`，表示不需要配置文件。将此变量设置为`true`会影响StarRocks的并发性。 |
| enable_profile                      | v2.5或更高版本       | false                | 控制是否发送查询的分析配置文件的布尔开关。默认值为`false`，表示不需要配置文件。将此变量设置为`true`会影响StarRocks的并发性。 |
| enable_pipeline_engine              | v2.3或更高版本       | true                 | 控制是否启用管道执行引擎的布尔开关。`true`表示启用，`false`表示相反。默认值：`true`。 |
| parallel_fragment_exec_instance_num | v2.3或更高版本       | 如果您已启用管道引擎，则可以将此变量设置为`1`。如果您未启用管道引擎，则应将其设置为CPU核心数的一半。 | 用于扫描BE上的节点的实例数。默认值为`1`。 |
| pipeline_dop                        | v2.3、v2.4和v2.5    | 0                    | 管道实例的并行性，用于调整查询的并发性。默认值：`0`，表示系统自动调整每个管道实例的并行性。< br />从 v3.0 开始，StarRocks基于查询并发性自适应地调整此参数。 |

- 在全局范围内将`is_report_success`设置为`false`：

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- 在全局范围内将`enable_profile`设置为`false`：

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- 在全局范围内将`enable_pipeline_engine`设置为`true`：

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- 在全局范围内将`parallel_fragment_exec_instance_num`设置为`1`：

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- 在全局范围内将`pipeline_dop`设置为`0`：

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

有关系统变量的更多信息，请参阅[系统变量](../reference/System_variable.md)。

## 设置用户属性

如果在您的集群中已创建新用户，则需要扩大其最大连接数（例如到`1000`）：

```SQL
-- 用您要扩大最大连接数的用户名替换<username>。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 下一步操作

部署并设置StarRocks集群后，您可以继续设计最适合您场景的表。查看[了解StarRocks表设计](../table_design/Table_design.md)获取有关设计表的详细说明。