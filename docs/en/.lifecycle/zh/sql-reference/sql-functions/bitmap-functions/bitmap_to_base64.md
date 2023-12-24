---
displayed_sidebar: English
---

# bitmap_to_base64

## 描述

将位图转换为 Base64 编码的字符串。从 v2.5 开始支持此功能。

## 语法

```Haskell
VARCHAR bitmap_to_base64(BITMAP bitmap)
```

## 参数

`bitmap`：要转换的位图。此参数是必需的。如果输入值无效，则会返回错误。

## 返回值

返回 VARCHAR 类型的值。

## 例子

示例 1：将此函数与其他位图函数一起使用。

```Plain
select bitmap_to_base64(bitmap_from_string("0, 1, 2, 3"));
+----------------------------------------------------+
| bitmap_to_base64(bitmap_from_string('0, 1, 2, 3')) |
+----------------------------------------------------+
| AjowAAABAAAAAAADABAAAAAAAAEAAgADAA==               |
+----------------------------------------------------+
1 row in set (0.00 sec)


select bitmap_to_base64(to_bitmap(1));
+--------------------------------+
| bitmap_to_base64(to_bitmap(1)) |
+--------------------------------+
| AQEAAAA=                       |
+--------------------------------+
1 row in set (0.00 sec)


select bitmap_to_base64(bitmap_empty());
+----------------------------------+
| bitmap_to_base64(bitmap_empty()) |
+----------------------------------+
| AA==                             |
+----------------------------------+
1 row in set (0.00 sec)
```

示例 2：将 BITMAP 列中的每个值转换为 Base64 编码的字符串。

1. 创建一个聚合表 `page_uv` ，其中 `AGGREGATE KEY` 是 （`page_id`， `visit_date`）。此表包含一个 BITMAP 列 `visit_users` ，其值将进行聚合。

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

2. 在此表中插入数据。

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

3. 将 `visit_users` 列中的每个值转换为 Base64 编码的字符串。

    ```Plain
      select page_id, bitmap_to_base64(visit_users) from page_uv;
      +---------+------------------------------------------+
      | page_id | bitmap_to_base64(visit_users)            |
      +---------+------------------------------------------+
      |       1 | CgMAAAANAAAAAAAAABcAAAAAAAAAIQAAAAAAAAA= |
      |       1 | AQ0AAAA=                                 |
      |       2 | ARcAAAA=                                 |
      +---------+------------------------------------------+
    ```
