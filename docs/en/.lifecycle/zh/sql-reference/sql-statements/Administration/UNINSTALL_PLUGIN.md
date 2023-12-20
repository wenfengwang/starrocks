---
displayed_sidebar: English
---

# 卸载插件

## 描述

此语句用于卸载插件。

:::tip

此操作需要 SYSTEM 级别的 PLUGIN 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 中的指南来授予此权限。

:::

## 语法

```SQL
UNINSTALL PLUGIN <plugin_name>
```

可以通过 SHOW PLUGINS 命令查看 plugin_name。

只有非内置插件才能被卸载。

## 示例

1. 卸载一个插件：

   ```SQL
   UNINSTALL PLUGIN auditdemo;
   ```