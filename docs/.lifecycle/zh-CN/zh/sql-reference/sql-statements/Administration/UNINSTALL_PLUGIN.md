```
---
displayed_sidebar: "Chinese"
---

# 卸载插件

## 功能

此语句用于卸载一个插件。

## 语法

```SQL
UNINSTALL PLUGIN plugin_name
```

插件名称可以通过 `SHOW PLUGINS;` 命令查看。

只能卸载非内置的插件。

## 示例

1. 卸载一个插件：

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```