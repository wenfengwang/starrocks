---
displayed_sidebar: English
---

# 显示管理员配置

## 描述

显示当前集群的配置（目前只能显示 FE 配置项）。有关这些配置项的详细说明，请参见[配置](../../../administration/FE_configuration.md#fe-configuration-items)。

如果要设置或修改配置项，请使用 [ADMIN SET CONFIG](ADMIN_SET_CONFIG.md)。

:::提示

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照[GRANT](../account-management/GRANT.md)中的说明授予此权限。

:::

## 语法

```sql
ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"]
```

注意：

返回参数说明：

```plain text
1. Key:        配置项名称
2. Value:      配置项值
3. Type:       配置项类型 
4. IsMutable:  是否可以通过 ADMIN SET CONFIG 命令进行设置
5. MasterOnly: 是否仅适用于主 FE
6. Comment:    配置项描述 
```

## 例子

1. 查看当前 FE 节点的配置。

    ```sql
    ADMIN SHOW FRONTEND CONFIG;
    ```

2. 使用 `like` 谓词搜索当前 FE 节点的配置。 

    ```plain text
    mysql> ADMIN SHOW FRONTEND CONFIG LIKE '%check_java_version%';
    +--------------------+-------+---------+-----------+------------+---------+
    | Key                | Value | Type    | IsMutable | MasterOnly | Comment |
    +--------------------+-------+---------+-----------+------------+---------+
    | check_java_version | true  | boolean | false     | false      |         |
    +--------------------+-------+---------+-----------+------------+---------+
    1 row in set (0.00 sec)
    ```
