---
displayed_sidebar: English
---

# 设置管理员配置

## 描述

此语句用于设置集群的配置项（目前只能使用该命令设置 FE 动态配置项）。您可以使用[ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md)命令查看这些配置项。

FE 重新启动后，配置将恢复为`fe.conf`文件中的默认值。因此，我们建议您同时修改`fe.conf`中的配置项，以防止修改丢失。

:::提示

此操作需要SYSTEM级别的OPERATE权限。您可以按照[GRANT](../account-management/GRANT.md)中的说明授予此权限。

:::

## 语法

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 例子

1. 将`disable_balance`设置为`true`。

    ```sql
    ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
    ```
