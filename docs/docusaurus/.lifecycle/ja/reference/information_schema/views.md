---
displayed_sidebar: "Japanese"
---

# ビュー

`views`は、すべてのユーザー定義ビューに関する情報を提供します。

`views`には、次のフィールドが提供されています：

| **フィールド**        | **説明**                                                 |
| ------------------- | --------------------------------------------------------- |
| TABLE_CATALOG       | ビューが属するカタログの名前。この値は常に `def` です。   |
| TABLE_SCHEMA        | ビューが属するデータベースの名前。                         |
| TABLE_NAME          | ビューの名前。                                           |
| VIEW_DEFINITION     | ビューの定義を提供する `SELECT` 文。                       |
| CHECK_OPTION        | `CHECK_OPTION` 属性の値。値は `NONE`、 `CASCADE`、または `LOCAL` のいずれかです。 |
| IS_UPDATABLE        | ビューが更新可能かどうか。ビューが更新可能であれば、フラグは `YES`（TRUE）に設定されます。それ以外の場合、フラグは `NO`（FALSE）に設定されます。ビューが更新不可能な場合、`UPDATE`、 `DELETE`、 `INSERT` などのステートメントは不適切であり、拒否されます。 |
| DEFINER             | ビューを作成したユーザーのユーザー。                       |
| SECURITY_TYPE       | ビューの `SQL SECURITY` 特性。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CHARACTER_SET_CLIENT|                                                             |
| COLLATION_CONNECTION|                                                             |