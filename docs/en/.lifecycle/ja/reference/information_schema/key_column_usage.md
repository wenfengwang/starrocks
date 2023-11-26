---
displayed_sidebar: "Japanese"
---

# key_column_usage

`key_column_usage`は、一意の制約、主キー制約、または外部キー制約によって制約されているすべての列を識別します。

`key_column_usage`には、次のフィールドが提供されます：

| **フィールド**                  | **説明**                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG            | 制約が所属するカタログの名前です。この値は常に `def` です。         |
| CONSTRAINT_SCHEMA             | 制約が所属するデータベースの名前です。                           |
| CONSTRAINT_NAME               | 制約の名前です。                                               |
| TABLE_CATALOG                 | テーブルが所属するカタログの名前です。この値は常に `def` です。         |
| TABLE_SCHEMA                  | テーブルが所属するデータベースの名前です。                         |
| TABLE_NAME                    | 制約を持つテーブルの名前です。                                     |
| COLUMN_NAME                   | 制約を持つ列の名前です。外部キーの場合、これは外部キーの列であり、外部キーが参照する列ではありません。 |
| ORDINAL_POSITION              | 制約内の列の位置です。テーブル内の列の位置ではありません。列の位置は1から始まります。 |
| POSITION_IN_UNIQUE_CONSTRAINT | 一意制約および主キー制約の場合は `NULL` です。外部キー制約の場合、この列は参照されるテーブルのキー内の序数位置です。 |
| REFERENCED_TABLE_SCHEMA       | 制約が参照するスキーマの名前です。                                 |
| REFERENCED_TABLE_NAME         | 制約が参照するテーブルの名前です。                                 |
| REFERENCED_COLUMN_NAME        | 制約が参照する列の名前です。                                     |
