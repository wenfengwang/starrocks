---
displayed_sidebar: "Japanese"
---

# table_constraints

`table_constraints`は、どのテーブルに制約があるかを記述しています。

`table_constraints`には次のフィールドが提供されています。

| **Field**          | **Description**                                              |
| ------------------ | ------------------------------------------------------------ |
| CONSTRAINT_CATALOG | 制約が属するカタログの名前。この値は常に `def` です。        |
| CONSTRAINT_SCHEMA  | 制約が属するデータベースの名前。                             |
| CONSTRAINT_NAME    | 制約の名前。                                                 |
| TABLE_SCHEMA       | テーブルが属するデータベースの名前。                         |
| TABLE_NAME         | テーブルの名前。                                              |
| CONSTRAINT_TYPE    | 制約の種類。                                                 |