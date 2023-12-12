---
displayed_sidebar: "Japanese"
---

# referential_constraints

`referential_constraints`にはすべての参照（外部キー）制約が含まれています。

`referential_constraints`には、次のフィールドが提供されています。

| **フィールド**             | **説明**                                                    |
| ------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG        | 制約が属するカタログの名前。この値は常に `def` です。              |
| CONSTRAINT_SCHEMA         | 制約が属するデータベースの名前。                                      |
| CONSTRAINT_NAME           | 制約の名前。                                                    |
| UNIQUE_CONSTRAINT_CATALOG | 制約が参照する一意制約を含むカタログの名前。この値は常に `def` です。  |
| UNIQUE_CONSTRAINT_SCHEMA  | 制約が参照する一意制約を含むスキーマの名前。                           |
| UNIQUE_CONSTRAINT_NAME    | 制約が参照する一意制約の名前。                                       |
| MATCH_OPTION              | 制約の `MATCH` 属性の値。現時点で唯一の有効な値は `NONE` です。           |
| UPDATE_RULE               | 制約の `ON UPDATE` 属性の値。有効な値: `CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT`、`NO ACTION`。 |
| DELETE_RULE               | 制約の `ON DELETE` 属性の値。有効な値: `CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT`、`NO ACTION`。 |
| TABLE_NAME                | テーブルの名前。この値は `TABLE_CONSTRAINTS` テーブルと同じです。       |
| REFERENCED_TABLE_NAME     | 制約が参照するテーブルの名前。                                       |