---
displayed_sidebar: English
---

# 卸载插件

## 描述

该语句用于卸载插件。

:::提示

此操作需要 SYSTEM 级 PLUGIN 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```SQL
UNINSTALL PLUGIN <plugin_name>
```

可以通过 SHOW PLUGINS 命令查看 plugin_name

只能卸载非内置插件。

## 例子

1. 卸载插件：

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```
