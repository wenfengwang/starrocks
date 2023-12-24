---
displayed_sidebar: English
---

# 显示插件

## 描述

该语句用于查看已安装的插件。

:::提示

此操作需要 SYSTEM 级 PLUGIN 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```sql
SHOW PLUGINS
```

该命令将显示所有内置插件和自定义插件。

## 例子

1. 显示已安装的插件：

    ```sql
    SHOW PLUGINS;
    ```
