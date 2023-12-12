---
displayed_sidebar: "Japanese"
---

# key_column_usage

`key_column_usage`は、一意のプライマリキーまたは外部キー制約によって制限されるすべての列を識別します。

`key_column_usage`には、次のフィールドが提供されています：

| **フィールド**                 | **説明**                                                    |
| ----------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG            | 制約が属するカタログの名前。この値は常に `def` です。       |
| CONSTRAINT_SCHEMA             | 制約が属するデータベースの名前。                            |
| CONSTRAINT_NAME               | 制約の名前。                                                |
| TABLE_CATALOG                 | テーブルが属するカタログの名前。この値は常に `def` です。    |
| TABLE_SCHEMA                  | テーブルが属するデータベースの名前。                       |
| TABLE_NAME                    | 制約を持つテーブルの名前。                                  |
| COLUMN_NAME                   | 制約を持つ列の名前。制約が外部キーの場合、これは外部キーの列であり、外部キーが参照する列ではありません。|
| ORDINAL_POSITION              | 制約内の列の位置。列の位置は1から始まります。                |
| POSITION_IN_UNIQUE_CONSTRAINT | ユニークキーおよびプライマリキー制約の場合は `NULL` です。外部キー制約の場合、この列は参照されているテーブルのキー内の序数です。|
| REFERENCED_TABLE_SCHEMA       | 制約で参照されるスキーマの名前。                            |
| REFERENCED_TABLE_NAME         | 制約で参照されるテーブルの名前。                            |
| REFERENCED_COLUMN_NAME        | 制約で参照される列の名前。                                  |