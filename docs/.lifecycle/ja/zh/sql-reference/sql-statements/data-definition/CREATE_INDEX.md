---
displayed_sidebar: Chinese
---

# CREATE INDEX

## 機能

ビットマップインデックスの作成をサポートしています。ビットマップインデックスの使用説明と適用シナリオについては、[Bitmap インデックス](../../../using_starrocks/Bitmap_index.md)を参照してください。

:::tip

この操作には対応するテーブルのALTER権限が必要です。[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

:::

## 文法

```SQL
CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT '']
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| index_name     | はい     | インデックス名。命名規則は以下の通りです：<ul><li>アルファベット(a-zまたはA-Z)、数字(0-9)、アンダースコア(_)で構成され、アルファベットで始まる必要があります。</li><li>全長は64文字を超えてはいけません。</li></ul>同一テーブル内で同名のインデックスを作成することはできません。 |
| table_name     | はい     | テーブル名。                                                 |
| column_name    | はい     | インデックスを作成する列名。このステートメントを実行すると、特定の列に対してのみインデックスを作成でき、同一列に複数のインデックスを作成することはできません。 |
| COMMENT        | いいえ   | インデックスのコメント。                                      |

## 注意事項

- 主キーモデルと明細モデルのすべての列でビットマップインデックスを作成できます。集約モデルと更新モデルでは、次元列（Key列）のみがビットマップインデックスの作成をサポートしています。
- FLOAT、DOUBLE、BOOLEAN、DECIMAL型の列にビットマップインデックスを作成することはサポートされていません。

## 例

たとえば、`sales_records`というテーブルがあり、その作成文は以下の通りです：

```SQL
CREATE TABLE sales_records
(
    record_id int,
    seller_id int,
    item_id int
)
DISTRIBUTED BY hash(record_id)
PROPERTIES (
    "replication_num" = "3"
);
```

テーブル `sales_records` の `item_id` 列にビットマップインデックスを作成し、インデックス名を`index3`とします。

```SQL
CREATE INDEX index3 ON sales_records (item_id) USING BITMAP COMMENT '';
```

または

```SQL
CREATE INDEX index3 ON sales_records (item_id);
```

## 関連操作

- インデックスを表示するには、[SHOW INDEX](../Administration/SHOW_INDEX.md)を参照してください。
- インデックスを削除するには、[DROP INDEX](../data-definition/DROP_INDEX.md)を参照してください。
