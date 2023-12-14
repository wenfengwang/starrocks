---
displayed_sidebar: "Chinese"
---

# 卸载插件

## 描述

此语句用于卸载插件。

语法：

```SQL
UNINSTALL PLUGIN <插件名称>
```

插件名称可通过SHOW PLUGINS命令查看。

只能卸载非内置插件。

## 例子

1. 卸载一个插件：

    ```SQL
    UNINSTALL PLUGIN auditdemo;
    ```