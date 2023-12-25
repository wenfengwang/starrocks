---
displayed_sidebar: Chinese
---

# bitmap_to_base64

## 機能

bitmap を Base64 文字列に変換します。この関数はバージョン 2.5 からサポートされています。

## 文法

```Haskell
VARCHAR bitmap_to_base64(BITMAP bitmap)
```

## パラメータ説明

`bitmap`: 変換する bitmap。必須です。入力値の形式が不正な場合は、エラーが返されます。

## 戻り値の説明

VARCHAR 型の値を返します。

## 例

例1：この関数を他の bitmap 関数と組み合わせて使用します。

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

例2：データテーブルの BITMAP 列を Base64 文字列に変換します。

1. BITMAP 列を持つ集約テーブルを作成します。`visit_users` 列は集約列で、列の型は BITMAP、集約関数は bitmap_union() を使用します。

    ```SQL
    CREATE TABLE `page_uv`
        (`page_id` INT NOT NULL COMMENT 'ページID',
        `visit_date` datetime NOT NULL COMMENT '訪問日時',
        `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '訪問ユーザーID'
        ) ENGINE=OLAP
        AGGREGATE KEY(`page_id`, `visit_date`)
        DISTRIBUTED BY HASH(`page_id`)
        PROPERTIES (
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
        );
    ```

2. テーブルにデータをインポートします。

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

3. `visit_users` 列の各行の `bitmap` 値を Base64 文字列に変換します。

    ```Plain
        select page_id, bitmap_to_base64(visit_users)
        from page_uv;
        +---------+------------------------------------------+
        | page_id | bitmap_to_base64(visit_users)            |
        +---------+------------------------------------------+
        |       1 | CgMAAAANAAAAAAAAABcAAAAAAAAAIQAAAAAAAAA= |
        |       1 | AQ0AAAA=                                 |
        |       2 | ARcAAAA=                                 |
        +---------+------------------------------------------+
    ```
