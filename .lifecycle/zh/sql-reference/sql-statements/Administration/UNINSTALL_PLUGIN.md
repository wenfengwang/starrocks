---
displayed_sidebar: English
---

# 卸载插件

## 描述

此语句用于卸载插件。

:::提示

此操作需要 **SYSTEM** 级别的 **PLUGIN** 权限。您可以根据 [GRANT](../account-management/GRANT.md) 命令中的指导来授予相应权限。

:::

## 语法

```SQL
UNINSTALL PLUGIN <plugin_name>
```

可以通过 SHOW PLUGINS 命令查看插件名称（plugin_name）

只有非内建的插件可以被卸载。

## 示例

1. 卸载一个插件：

   ```SQL
   UNINSTALL PLUGIN auditdemo;
   ```
