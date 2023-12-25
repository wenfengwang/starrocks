---
displayed_sidebar: English
---

# key_column_usage

`key_column_usage` は、ユニーク制約、プライマリキー、または外部キー制約によって制限されるすべての列を識別します。

`key_column_usage` で提供されるフィールドは以下の通りです：

| **フィールド**                     | **説明**                                              |
| ----------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG            | 制約が属するカタログの名前。この値は常に `def` です。 |
| CONSTRAINT_SCHEMA             | 制約が属するデータベースの名前。    |
| CONSTRAINT_NAME               | 制約の名前。                                  |
| TABLE_CATALOG                 | テーブルが属するカタログの名前。この値は常に `def` です。 |
| TABLE_SCHEMA                  | テーブルが属するデータベースの名前。         |
| TABLE_NAME                    | 制約を持つテーブルの名前。               |
| COLUMN_NAME                   | 制約を持つ列の名前。制約が外部キーである場合、これは外部キーの列であり、外部キーが参照する列ではありません。 |
| ORDINAL_POSITION              | テーブル内の列の位置ではなく、制約内の列の位置。列の位置は1から始まる番号で指定されます。 |
| POSITION_IN_UNIQUE_CONSTRAINT | ユニーク制約とプライマリキー制約には `NULL`。外部キー制約の場合、この列は参照されるテーブルのキー内での序数位置を示します。 |
| REFERENCED_TABLE_SCHEMA       | 制約によって参照されるスキーマの名前。         |
| REFERENCED_TABLE_NAME         | 制約によって参照されるテーブルの名前。          |
| REFERENCED_COLUMN_NAME        | 制約によって参照される列の名前。         |
