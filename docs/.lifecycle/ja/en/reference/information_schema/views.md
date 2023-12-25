---
displayed_sidebar: English
---

# views

`views` は、すべてのユーザー定義ビューに関する情報を提供します。

`views` には以下のフィールドが含まれています:

| **フィールド**            | **説明**                                              |
| -------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG        | ビューが属するカタログの名前。この値は常に `def` です。 |
| TABLE_SCHEMA         | ビューが属するデータベースの名前。          |
| TABLE_NAME           | ビューの名前。                                        |
| VIEW_DEFINITION      | ビューの定義を提供する `SELECT` ステートメント。 |
| CHECK_OPTION         | `CHECK_OPTION` 属性の値。値は `NONE`、`CASCADE`、または `LOCAL` のいずれかです。 |
| IS_UPDATABLE         | ビューが更新可能かどうか。フラグは、ビューに対して `UPDATE` や `DELETE`（および類似の操作）が許可されている場合は `YES`（true）、そうでない場合は `NO`（false）に設定されます。ビューが更新不可能な場合、`UPDATE`、`DELETE`、`INSERT` などのステートメントは不正とされ、拒否されます。 |
| DEFINER              | ビューを作成したユーザー。                   |
| SECURITY_TYPE        | ビューの `SQL SECURITY` 特性。値は `DEFINER` または `INVOKER` のいずれかです。 |
| CHARACTER_SET_CLIENT |                                                              |
| COLLATION_CONNECTION |                                                              |

