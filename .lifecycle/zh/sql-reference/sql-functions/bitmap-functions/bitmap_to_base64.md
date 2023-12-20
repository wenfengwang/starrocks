---
displayed_sidebar: English
---

# 位图转换为Base64

## 描述

此功能将位图转换成Base64编码的字符串。该函数自v2.5版本起提供支持。

## 语法

```Haskell
VARCHAR bitmap_to_base64(BITMAP bitmap)
```

## 参数

bitmap：需要转换的位图。该参数为必填项。如果输入的值无效，将返回错误信息。

## 返回值

返回的值为VARCHAR类型。

## 示例

示例1：将该函数与其他位图函数结合使用。

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

示例2：将BITMAP列中的每个值转换成Base64编码的字符串。

1. 创建一个聚合表page_uv，其聚合键为（page_id，visit_date）。该表包含一个BITMAP类型的列visit_users，需要对其值进行聚合。

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

3. 将visit_users列中的每个值转换成Base64编码的字符串。

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
