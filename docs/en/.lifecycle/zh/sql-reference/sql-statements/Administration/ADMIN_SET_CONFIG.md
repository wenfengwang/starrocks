---
displayed_sidebar: English
---

# ADMIN SET CONFIG

## 描述

此语句用于设置集群的配置项（目前，只有FE的动态配置项可以使用此命令进行设置）。您可以使用 [ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md) 命令查看这些配置项。

FE重启后，配置将会恢复到 `fe.conf` 文件中的默认值。因此，我们建议您也在 `fe.conf` 文件中修改配置项，以防止修改后的配置丢失。

:::tip

此操作需要 SYSTEM 级别的 OPERATE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的指南来授予此权限。

:::

## 语法

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 示例

1. 将 `disable_balance` 设置为 `true`。

   ```sql
   ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
   ```