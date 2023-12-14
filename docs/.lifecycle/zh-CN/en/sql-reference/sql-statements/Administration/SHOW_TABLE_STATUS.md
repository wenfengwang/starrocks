---
displayed_sidebar: "Chinese"
---

# 显示表格状态

## 描述

此语句用于查看表格中的部分信息。

语法:

```sql
SHOW TABLE STATUS
[FROM db] [LIKE "pattern"]
```

> 注意
>
> 此语句主要兼容于MySQL语法。目前，它只展示少量信息，比如注释信息。

## 例子

1. 查看当前数据库下所有表格的信息。

    ```SQL
    SHOW TABLE STATUS;
    ```

2. 查看指定数据库中表格名称包含“example”的所有表格的信息。

    ```SQL
    SHOW TABLE STATUS FROM db LIKE "%example%";
    ```