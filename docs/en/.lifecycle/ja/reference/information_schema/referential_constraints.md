---
displayed_sidebar: "Japanese"
---

# referential_constraints

`referential_constraints`には、すべての参照（外部キー）制約が含まれています。

`referential_constraints`には、次のフィールドが提供されています：

| **フィールド**                 | **説明**                                              |
| ------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG        | 制約が所属するカタログの名前です。この値は常に `def` です。 |
| CONSTRAINT_SCHEMA         | 制約が所属するデータベースの名前です。    |
| CONSTRAINT_NAME           | 制約の名前です。                                  |
| UNIQUE_CONSTRAINT_CATALOG | 制約が参照する一意の制約を含むカタログの名前です。この値は常に `def` です。 |
| UNIQUE_CONSTRAINT_SCHEMA  | 制約が参照する一意の制約を含むスキーマの名前です。 |
| UNIQUE_CONSTRAINT_NAME    | 制約が参照する一意の制約の名前です。 |
| MATCH_OPTION              | 制約の `MATCH` 属性の値です。現時点では、有効な値は `NONE` のみです。 |
| UPDATE_RULE               | 制約の `ON UPDATE` 属性の値です。有効な値: `CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT`、`NO ACTION`。 |
| DELETE_RULE               | 制約の `ON DELETE` 属性の値です。有効な値: `CASCADE`、`SET NULL`、`SET DEFAULT`、`RESTRICT`、`NO ACTION`。 |
| TABLE_NAME                | テーブルの名前です。この値は `TABLE_CONSTRAINTS` テーブルと同じです。 |
| REFERENCED_TABLE_NAME     | 制約が参照するテーブルの名前です。          |
