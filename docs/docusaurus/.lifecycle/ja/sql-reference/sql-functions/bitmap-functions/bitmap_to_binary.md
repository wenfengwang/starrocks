---
displayed_sidebar: "Japanese"
---

# bitmap_to_binary

## 説明

Bitmap値をバイナリ文字列に変換します。

bitmap_to_binaryは主にBitmapデータのエクスポートに使用されます。 これは[bitmap_to_base64](./bitmap_to_base64.md)よりも圧縮効果が高いです。

Parquetなどのバイナリファイルに直接データをエクスポートする予定がある場合は、この関数を推奨します。

この機能はv3.0からサポートされています。

## 構文

```Haskell
VARBINARY bitmap_to_binary(BITMAP bitmap)
```

## パラメーター

`bitmap`: 変換するBitmap、必須。 入力値が無効な場合、エラーが返されます。

## 戻り値

VARBINARYタイプの値を返します。

## 例

例1：他のbitmap関数と一緒にこの関数を使用します。

```Plain
mysql> select hex(bitmap_to_binary(bitmap_from_string("0, 1, 2, 3")));
+---------------------------------------------------------+
| hex(bitmap_to_binary(bitmap_from_string('0, 1, 2, 3'))) |
+---------------------------------------------------------+
| 023A3000000100000000000300100000000000010002000300      |
+---------------------------------------------------------+
1行が返されました（0.01秒）

mysql> select hex(bitmap_to_binary(to_bitmap(1)));
+-------------------------------------+
| hex(bitmap_to_binary(to_bitmap(1))) |
+-------------------------------------+
| 0101000000                          |
+-------------------------------------+
1行が返されました（0.01秒）

mysql> select hex(bitmap_to_binary(bitmap_empty()));
+---------------------------------------+
| hex(bitmap_to_binary(bitmap_empty())) |
+---------------------------------------+
| 00                                    |
+---------------------------------------+
1行が返されました（0.01秒）
```

例2：BITMAP列内の各値をバイナリ文字列に変換します。

1. `page_id`、`visit_date`の`AGGREGATE KEY`でAggregateテーブル`page_uv`を作成します。 このテーブルには集約されるべき`visit_users`という名前のBITMAP列が含まれています。

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

2. このテーブルにデータを挿入します。

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

3. `visit_users`列内の各値をバイナリエンコードされた文字列に変換します。

    ```Plain
       mysql> select page_id, hex(bitmap_to_binary(visit_users)) from page_uv;
       +---------+------------------------------------------------------------+
       | page_id | hex(bitmap_to_binary(visit_users))                         |
       +---------+------------------------------------------------------------+
       |       1 | 0A030000000D0000000000000017000000000000002100000000000000 |
       |       1 | 010D000000                                                 |
       |       2 | 0117000000                                                 |
       +---------+------------------------------------------------------------+
       3行が返されました（0.01秒）
    ```