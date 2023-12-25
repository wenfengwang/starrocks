---
displayed_sidebar: Chinese
---

# views

`views` は、すべてのユーザー定義ビューに関する情報を提供します。

`views` は以下のフィールドを提供します：

| フィールド           | 説明                                                         |
| -------------------- | ------------------------------------------------------------ |
| TABLE_CATALOG        | ビューが属するカタログ名。この値は常に def です。            |
| TABLE_SCHEMA         | ビューが属するデータベース名。                               |
| TABLE_NAME           | ビューの名前。                                               |
| VIEW_DEFINITION      | ビュー定義のSELECTステートメントを提供します。               |
| CHECK_OPTION         | CHECK_OPTION 属性の値。この値は NONE、CASCADE、または LOCAL のいずれかです。 |
| IS_UPDATABLE         | ビューが更新可能かどうか。ビューに対する UPDATE および DELETE（および類似の操作）が合法であれば、フラグは YES（true）に設定されます。そうでなければ、フラグは NO（false）に設定されます。ビューが更新不可能な場合、UPDATE、DELETE、INSERT などのステートメントは非合法であり、拒否されます。 |
| DEFINER              | ビューを作成したユーザー。                                   |
| SECURITY_TYPE        | ビューの SQL SECURITY 特性。この値は DEFINER または INVOKER のいずれかです。 |
| CHARACTER_SET_CLIENT |                                                              |
| COLLATION_CONNECTION |                                                              |
