---
displayed_sidebar: Chinese
---

# DROP INDEX（インデックスの削除）

## 機能

指定されたテーブルのbitmapインデックスを削除します。bitmapインデックスを作成すると追加のストレージスペースが必要になるため、不要なインデックスは削除することをお勧めします。インデックスを削除すると、ストレージスペースは直ちに解放されます。

:::tip

この操作には対象テーブルのALTER権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```SQL
DROP INDEX index_name ON [db_name.]table_name
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**               |
| -------------- | -------- | ---------------------- |
| index_name     | はい     | 削除するインデックス名。 |
| table_name     | はい     | インデックスが作成されたテーブル。 |
| db_name        | いいえ   | テーブルが属するデータベース。   |

## 例

たとえば、`sales_records`テーブルの`item_id`列にbitmapインデックスを作成し、そのインデックス名を`index3`とします。

```SQL
CREATE INDEX index3 ON sales_records (item_id);
```

`sales_records`テーブルのインデックス`index3`を削除します。

```SQL
DROP INDEX index3 ON sales_records;
```
