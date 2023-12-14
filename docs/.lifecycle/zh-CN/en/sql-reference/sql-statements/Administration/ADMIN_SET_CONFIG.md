---
displayed_sidebar: "Chinese"
---

# 管理员设置配置

## 描述

此语句用于设置集群的配置项（目前，仅可使用此命令设置FE动态配置项）。您可以使用[ADMIN SHOW FRONTEND CONFIG](ADMIN_SET_CONFIG.md)命令查看这些配置项。

配置项将在FE重新启动后恢复到`fe.conf`文件中的默认值。因此，我们建议您也修改`fe.conf`中的配置项，以防止修改的丢失。

## 语法

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value")
```

## 示例

1. 将`disable_balance`设置为`true`。

    ```sql
    ADMIN SET FRONTEND CONFIG ("disable_balance" = "true");
    ```