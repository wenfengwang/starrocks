---
displayed_sidebar: English
---

# 管理员显示配置

## 描述

展示当前集群的配置情况（目前仅能展示FE配置项）。有关这些配置项的详细描述，请参阅[配置](../../../administration/FE_configuration.md#fe-configuration-items)部分。

若需设置或修改配置项，请使用 [ADMIN SET CONFIG](ADMIN_SET_CONFIG.md) 命令。

:::提示

本操作需要SYSTEM级别的OPERATE权限。您可以遵循[GRANT](../account-management/GRANT.md)命令的指南来授予相应权限。

:::

## 语法

```sql
ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"]
```

注意：

返回参数的描述：

```plain
1. Key:        Configuration item name
2. Value:      Configuration item value
3. Type:       Configuration item type 
4. IsMutable:  Whether it can be set through the ADMIN SET CONFIG command
5. MasterOnly: Whether it only applies to leader FE
6. Comment:    Configuration item description 
```

## 示例

1. 查看当前FE节点的配置。

   ```sql
   ADMIN SHOW FRONTEND CONFIG;
   ```

2. 利用like谓词查询当前FE节点的配置。

   ```plain
   mysql> ADMIN SHOW FRONTEND CONFIG LIKE '%check_java_version%';
   +--------------------+-------+---------+-----------+------------+---------+
   | Key                | Value | Type    | IsMutable | MasterOnly | Comment |
   +--------------------+-------+---------+-----------+------------+---------+
   | check_java_version | true  | boolean | false     | false      |         |
   +--------------------+-------+---------+-----------+------------+---------+
   1 row in set (0.00 sec)
   ```
