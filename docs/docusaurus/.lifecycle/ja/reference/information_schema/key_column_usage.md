---
displayed_sidebar: "Japanese"
---

# key_column_usage

`key_column_usage`は、一意のプライマリキーまたは外部キー制約で制限されるすべての列を識別します。

`key_column_usage`には次のフィールドが提供されています：

| **フィールド**                   | **説明**                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG            | 制約が所属するカタログの名前。この値は常に `def` です。                 |
| CONSTRAINT_SCHEMA             | 制約が所属するデータベースの名前。                                |
| CONSTRAINT_NAME               | 制約の名前。                                                  |
| TABLE_CATALOG                 | テーブルが所属するカタログの名前。この値は常に `def` です。             |
| TABLE_SCHEMA                  | テーブルが所属するデータベースの名前。                             |
| TABLE_NAME                    | 制約を持つテーブルの名前。                                       |
| COLUMN_NAME                   | 制約を持つ列の名前。制約が外部キーの場合、これは外部キーの列であり、外部キーが参照する列ではありません。 |
| ORDINAL_POSITION              | 制約内の列の位置ですが、テーブル内の列の位置ではありません。列の位置は1から番号付けされます。|
| POSITION_IN_UNIQUE_CONSTRAINT | 一意制約およびプライマリキー制約では `NULL`。外部キー制約では、この列は参照されているテーブルのキーの序数位置です。|
| REFERENCED_TABLE_SCHEMA       | 制約によって参照されているスキーマの名前。                   |
| REFERENCED_TABLE_NAME         | 制約によって参照されているテーブルの名前。                  |
| REFERENCED_COLUMN_NAME        | 制約によって参照されている列の名前。                     |