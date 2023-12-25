---
displayed_sidebar: Chinese
---

# referential_constraints

`referential_constraints` には、すべての参照（外部キー）制約が含まれています。

`referential_constraints` は以下のフィールドを提供します：

| フィールド                  | 説明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG        | 制約が属するカタログの名前。この値は常に def です。          |
| CONSTRAINT_SCHEMA         | 制約が属するデータベースの名前。                             |
| CONSTRAINT_NAME           | 制約の名前。                                                 |
| UNIQUE_CONSTRAINT_CATALOG | 制約が参照するユニーク制約が含まれるカタログの名前。この値は常に def です。 |
| UNIQUE_CONSTRAINT_SCHEMA  | 制約が参照するユニーク制約が含まれるスキーマの名前。         |
| UNIQUE_CONSTRAINT_NAME    | 制約が参照するユニーク制約の名前。                           |
| MATCH_OPTION              | 制約の MATCH 属性の値。現在有効な値は NONE のみです。        |
| UPDATE_RULE               | 制約の ON UPDATE 属性の値。有効な値は CASCADE、SET NULL、SET DEFAULT、RESTRICT、NO ACTION です。 |
| DELETE_RULE               | 制約の ON DELETE 属性の値。有効な値は CASCADE、SET NULL、SET DEFAULT、RESTRICT、NO ACTION です。 |
| TABLE_NAME                | テーブルの名前。この値は TABLE_CONSTRAINTS テーブル内の値と同じです。 |
| REFERENCED_TABLE_NAME     | 制約が参照するテーブルの名前。                               |
