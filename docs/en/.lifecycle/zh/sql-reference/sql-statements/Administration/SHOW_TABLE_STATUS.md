---
displayed_sidebar: English
---

# 显示表状态

## 描述

此语句用于查看表中的一些信息。

:::tip

此操作不需要权限。

:::

## 语法

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
> 此语句主要与MySQL语法兼容。目前，它只显示一些信息，例如Comment。

## 示例

1. 查看当前数据库下所有表的信息。

   ```SQL
   SHOW TABLE STATUS;
   ```

2. 查看指定数据库下，名称中包含example的所有表的信息。

   ```SQL
   SHOW TABLE STATUS FROM db LIKE "%example%";
   ```