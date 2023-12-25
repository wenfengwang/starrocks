---
displayed_sidebar: Chinese
---

# bitmap_to_binary

## 機能

Bitmap を StarRocks が定義するルールに従ってバイナリ文字列に変換します。

この関数は主に Bitmap データのエクスポートに使用され、[bitmap_to_base64](./bitmap_to_base64.md)よりも圧縮効果が高いです。Parquet などのバイナリファイルに Bitmap データをエクスポートする場合は、bitmap_to_binary 関数の使用を推奨します。

この関数はバージョン 3.0 からサポートされています。

## 文法

```Haskell
VARBINARY bitmap_to_binary(BITMAP bitmap)
```

## パラメータ説明

`bitmap`: 変換する bitmap、必須。

## 戻り値の説明

VARBINARY 型の値を返します。

## 例

例 1: この関数を他の Bitmap 関数と組み合わせて使用します。

```Plain
mysql> select hex(bitmap_to_binary(bitmap_from_string("0, 1, 2, 3")));
+---------------------------------------------------------+
| hex(bitmap_to_binary(bitmap_from_string('0, 1, 2, 3'))) |
+---------------------------------------------------------+
| 023A3000000100000000000300100000000000010002000300      |
+---------------------------------------------------------+
1 row in set (0.01 sec)

mysql> select hex(bitmap_to_binary(to_bitmap(1)));
+-------------------------------------+
| hex(bitmap_to_binary(to_bitmap(1))) |
+-------------------------------------+
| 0101000000                          |
+-------------------------------------+
1 row in set (0.01 sec)

mysql> select hex(bitmap_to_binary(bitmap_empty()));
+---------------------------------------+
| hex(bitmap_to_binary(bitmap_empty())) |
+---------------------------------------+
| 00                                    |
+---------------------------------------+
1 row in set (0.01 sec)
```

例 2: テーブル内の Bitmap 列を Varbinary 文字列に変換します。

1. BITMAP 列を持つ集約テーブルを作成します。`visit_users` 列は集約列で、列の型は BITMAP で、集約関数 bitmap_union() を使用します。

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

3. `visit_users` 列の各行の Bitmap 値を Varbinary 文字列に変換します。hex 関数は不可視の文字列をクライアントに表示するためだけのもので、エクスポート時には使用しなくても構いません。

    ```Plain
       mysql> select page_id, hex(bitmap_to_binary(visit_users)) from page_uv;
       +---------+------------------------------------------------------------+
       | page_id | hex(bitmap_to_binary(visit_users))                         |
       +---------+------------------------------------------------------------+
       |       1 | 0A030000000D0000000000000017000000000000002100000000000000 |
       |       1 | 010D000000                                                 |
       |       2 | 0117000000                                                 |
       +---------+------------------------------------------------------------+
       3 rows in set (0.01 sec)
    ```
    
