---
displayed_sidebar: "英語"
---

# table_constraints

`table_constraints`は、制約が存在するテーブルについて記述します。

以下のフィールドが`table_constraints`に提供されています：

| **フィールド**       | **説明**                                                   |
| -------------------- | ----------------------------------------------------------- |
| CONSTRAINT_CATALOG   | 制約が属しているカタログの名前です。この値は常に`def`になります。   |
| CONSTRAINT_SCHEMA    | 制約が属しているデータベースの名前です。                      |
| CONSTRAINT_NAME      | 制約の名前です。                                           |
| TABLE_SCHEMA         | テーブルが属しているデータベースの名前です。                 |
| TABLE_NAME           | テーブルの名前です。                                       |
| CONSTRAINT_TYPE      | 制約のタイプです。                                         |