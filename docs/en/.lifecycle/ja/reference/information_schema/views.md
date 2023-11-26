---
displayed_sidebar: "Japanese"
---

# ビュー

`views`は、すべてのユーザー定義ビューに関する情報を提供します。

`views`には、以下のフィールドが提供されます：

| **フィールド**           | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG         | ビューが所属するカタログの名前です。この値は常に `def` です。               |
| TABLE_SCHEMA          | ビューが所属するデータベースの名前です。                                  |
| TABLE_NAME            | ビューの名前です。                                                |
| VIEW_DEFINITION       | ビューの定義を提供する `SELECT` ステートメントです。                           |
| CHECK_OPTION          | `CHECK_OPTION` 属性の値です。値は `NONE`、`CASCADE`、または `LOCAL` のいずれかです。 |
| IS_UPDATABLE          | ビューが更新可能かどうかを示します。このフラグは、ビューに対して `UPDATE` や `DELETE`（および類似の操作）が許可されている場合に `YES`（true）に設定されます。それ以外の場合、フラグは `NO`（false）に設定されます。ビューが更新不可能な場合、`UPDATE`、`DELETE`、`INSERT` などのステートメントは不正であり、拒否されます。 |
| DEFINER               | ビューを作成したユーザーのユーザーです。                                   |
| SECURITY_TYPE         | ビューの `SQL SECURITY` 特性です。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CHARACTER_SET_CLIENT  |                                                              |
| COLLATION_CONNECTION |                                                              |
