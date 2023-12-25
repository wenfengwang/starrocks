---
displayed_sidebar: Chinese
---

# key_column_usage

`key_column_usage` は、一意の制約、主キー、または外部キーの制約によって制限されるすべての列を識別するために使用されます。

`key_column_usage` は以下のフィールドを提供します：

| フィールド                      | 説明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG            | 制約が属するカタログの名前。この値は常に def です。             |
| CONSTRAINT_SCHEMA             | 制約が属するデータベースの名前。                               |
| CONSTRAINT_NAME               | 制約の名前。                                                   |
| TABLE_CATALOG                 | テーブルが属するカタログの名前。この値は常に def です。           |
| TABLE_SCHEMA                  | テーブルが属するデータベースの名前。                             |
| TABLE_NAME                    | 制約を持つテーブルの名前。                                     |
| COLUMN_NAME                   | 制約を持つ列の名前。制約が外部キーの場合、これは外部キーの列であり、参照される列ではありません。 |
| ORDINAL_POSITION              | 列が制約内での位置、テーブル内での位置ではありません。列の位置は 1 から始まります。 |
| POSITION_IN_UNIQUE_CONSTRAINT | 一意の制約と主キー制約の場合は NULL。外部キー制約の場合、この列は参照されるテーブルのキーの序数位置です。 |
| REFERENCED_TABLE_SCHEMA       | 制約が参照するスキーマの名前。                                 |
| REFERENCED_TABLE_NAME         | 制約が参照するテーブルの名前。                                 |
| REFERENCED_COLUMN_NAME        | 制約が参照する列の名前。                                       |
