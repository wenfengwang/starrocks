---
displayed_sidebar: English
---

# 部署后的设置

本主题将介绍在部署StarRocks之后需要执行的任务。

在您的新StarRocks集群投入生产使用之前，您必须确保初始账户的安全，并设置必要的变量和属性，以确保集群能够正确运行。

## 保护初始账户

在创建StarRocks集群时，会自动生成集群的初始root用户。root用户被授予了所有权限，即集群内所有权限的集合。我们建议您保护好这个用户账户，并避免在生产环境中使用它，以防止被滥用。

当集群创建时，StarRocks会自动为root用户分配一个空密码。请按照以下步骤为root用户设置一个新密码：

1. 使用root用户名和空密码通过MySQL客户端连接到StarRocks。

   ```Bash
   # Replace <fe_address> with the IP address (priority_networks) or FQDN 
   # of the FE node you connect to, and replace <query_port> 
   # with the query_port (Default: 9030) you specified in fe.conf.
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 执行以下SQL命令来重置root用户的密码：

   ```SQL
   -- Replace <password> with the password you want to assign to the root user.
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注意**
- 重置密码后请妥善保存。如果您忘记了密码，可参考[重置遗失的root密码](../administration/User_privilege.md#reset-lost-root-password)获取详细步骤。
- 在完成部署后的设置之后，您可以创建新的用户和角色，以便管理团队内的权限。详细操作请参见[管理用户权限](../administration/User_privilege.md)。

## 设置必要的系统变量

为了确保您的StarRocks集群在生产环境中能够正常工作，您需要设置以下系统变量：

|变量名称|StarRocks版本|推荐值|描述|
|---|---|---|---|
|is_report_success|v2.4 或更早版本|false|控制是否发送查询配置文件以进行分析的布尔开关。默认值为 false，这意味着不需要配置文件。将此变量设置为 true 会影响 StarRocks 的并发性。|
|enable_profile|v2.5 或更高版本|false|控制是否发送查询配置文件以进行分析的布尔开关。默认值为 false，这意味着不需要配置文件。将此变量设置为 true 会影响 StarRocks 的并发性。|
|enable_pipeline_engine|v2.3 或更高版本|true|控制是否启用管道执行引擎的布尔开关。 true 表示启用， false 表示相反。默认值：true。|
|parallel_fragment_exec_instance_num|v2.3 或更高版本|如果启用了管道引擎，则可以将此变量设置为 1。如果未启用管道引擎，则应将其设置为 CPU 核心数的一半。|用于扫描每个 BE 上的节点的实例。默认值为 1。|
|pipeline_dop|v2.3、v2.4、v2.5|0|管道实例的并行度，用于调整查询并发度。默认值：0，表示系统自动调整每个管道实例的并行度。从v3.0开始，StarRocks会根据查询并行度自适应调整该参数。|

- 全局设置is_report_success为false：

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- 全局设置enable_profile为false：

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- 全局设置enable_pipeline_engine为true：

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- 全局设置parallel_fragment_exec_instance_num为1：

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- 全局设置pipeline_dop为0：

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

有关系统变量的更多信息，请参见[System variables](../reference/System_variable.md)。

## 设置用户属性

如果您在集群中创建了新用户，需要增加他们的最大连接数（例如，设置为1000）：

```SQL
-- Replace <username> with the username you want to enlarge the maximum connection number for.
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 下一步

在部署和设置好您的StarRocks集群之后，您接下来可以着手设计最适合您使用场景的表格。有关设计表格的详细说明，请参见[理解StarRocks表格设计](../table_design/Table_design.md)。
