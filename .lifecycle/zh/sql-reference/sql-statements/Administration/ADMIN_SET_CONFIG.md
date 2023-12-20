---
displayed_sidebar: English
---

# 管理员设置配置

## 描述

此语句用于设置集群的配置项（目前，只有FE动态配置项可以使用此命令进行设置）。您可以通过 [ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md) 命令来查看这些配置项。

在FE重启之后，配置会被重置为fe.conf文件中的默认值。因此，我们建议您同时修改fe.conf文件中的配置项，以避免所做更改丢失。

:::提示

执行此操作需要**SYSTEM**级别的**OPERATE**权限。您可以遵循[GRANT](../account-management/GRANT.md)命令中的指引来授权此权限。

:::

## 语法

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 示例

1. 将disable_balance设置为true。

   ```sql
   ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
   ```
