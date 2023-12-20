---
displayed_sidebar: English
---

# 查看插件

## 说明

此语句用于查看已安装的插件。

:::提示

执行此操作需要 **SYSTEM** 级别的 **PLUGIN** 权限。您可以根据 [GRANT](../account-management/GRANT.md) 命令中的指引来授予相应权限。

:::

## 语法

```sql
SHOW PLUGINS
```

此命令会显示所有内置插件和自定义插件。

## 示例

1. 查看已安装的插件：

   ```sql
   SHOW PLUGINS;
   ```
