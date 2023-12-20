---
displayed_sidebar: English
---

# 显示插件

## 描述

此语句用于查看已安装的插件。

:::tip

此操作需要 SYSTEM 级别的 PLUGIN 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的指南来授予此权限。

:::

## 语法

```sql
SHOW PLUGINS
```

此命令将显示所有内置插件和自定义插件。

## 示例

1. 显示已安装的插件：

   ```sql
   SHOW PLUGINS;
   ```