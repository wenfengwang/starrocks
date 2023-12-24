---
displayed_sidebar: English
---

# bitmap_to_binary

## 描述

将 Bitmap 值转换为二进制字符串。

bitmap_to_binary 主要用于导出 Bitmap 数据。其压缩效果优于 [bitmap_to_base64](./bitmap_to_base64.md)。

如果您计划直接将数据导出到 Parquet 等二进制文件中，则建议使用此函数。

此函数从 v3.0 版本开始支持。

## 语法

```Haskell
VARBINARY bitmap_to_binary(BITMAP bitmap)
```

## 参数

`bitmap`：要转换的 Bitmap，必填。如果输入值无效，则会返回错误。

## 返回值

返回 VARBINARY 类型的值。

## 例子

示例 1：将此函数与其他 Bitmap 函数一起使用。

```Plain
mysql> select hex(bitmap_to_binary(bitmap_from_string("0, 1, 2, 3")));
+---------------------------------------------------------+
| hex(bitmap_to_binary(bitmap_from_string('0, 1, 2, 3'))) |
+---------------------------------------------------------+
| 023A3000000100000000000300100000000000010002000300      |
+---------------------------------------------------------+
1 行受影响 (用时 0.01 秒)

mysql> select hex(bitmap_to_binary(to_bitmap(1)));
+-------------------------------------+
| hex(bitmap_to_binary(to_bitmap(1))) |
+-------------------------------------+
| 0101000000                          |
+-------------------------------------+
1 行受影响 (用时 0.01 秒)

mysql> select hex(bitmap_to_binary(bitmap_empty()));
+---------------------------------------+
| hex(bitmap_to_binary(bitmap_empty())) |
+---------------------------------------+
| 00                                    |
+---------------------------------------+
1 行受影响 (用时 0.01 秒)
```

示例 2：将 BITMAP 列中的每个值转换为二进制字符串。

1. 创建一个聚合表 `page_uv`，其中 `AGGREGATE KEY` 为 (`page_id`, `visit_date`)。该表包含一个 BITMAP 列 `visit_users`，其值将被聚合。

    ```SQL
        CREATE TABLE `page_uv`
        (`page_id` INT NOT NULL,
        `visit_date` datetime NOT NULL,
        `visit_users` BITMAP BITMAP_UNION NOT NULL
        ) ENGINE=OLAP
        AGGREGATE KEY(`page_id`, `visit_date`)
        DISTRIBUTED BY HASH(`page_id`)
        PROPERTIES (
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
        );
    ```

2. 向该表中插入数据。

    ```SQL
      insert into page_uv values
      (1, '2020-06-23 01:30:30', to_bitmap(13)),
      (1, '2020-06-23 01:30:30', to_bitmap(23)),
      (1, '2020-06-23 01:30:30', to_bitmap(33)),
      (1, '2020-06-23 02:30:30', to_bitmap(13)),
      (2, '2020-06-23 01:30:30', to_bitmap(23));
      
      select * from page_uv order by page_id;
      +---------+---------------------+-------------+
      | page_id | visit_date          | visit_users |
      +---------+---------------------+-------------+
      |       1 | 2020-06-23 01:30:30 | NULL        |
      |       1 | 2020-06-23 02:30:30 | NULL        |
      |       2 | 2020-06-23 01:30:30 | NULL        |
      +---------+---------------------+-------------+
    ```

3. 将 `visit_users` 列中的每个值转换为二进制编码的字符串。

    ```Plain
       mysql> select page_id, hex(bitmap_to_binary(visit_users)) from page_uv;
       +---------+------------------------------------------------------------+
       | page_id | hex(bitmap_to_binary(visit_users))                         |
       +---------+------------------------------------------------------------+
       |       1 | 0A030000000D0000000000000017000000000000002100000000000000 |
       |       1 | 010D000000                                                 |
       |       2 | 0117000000                                                 |
       +---------+------------------------------------------------------------+
       3 行受影响 (用时 0.01 秒)
    ```
    
