---
displayed_sidebar: English
---

# 显示表状态

## 描述

此语句用于查看表中的部分信息。

:::提示

此操作无需权限。

:::

## 语法

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
>
> 该语句主要兼容 MySQL 语法。目前，它只显示一些信息，比如评论。

## 例子

1. 查看当前数据库中所有表的信息。

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 查看名称中包含 example 的表，并且在指定数据库下的所有表的信息。

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```
