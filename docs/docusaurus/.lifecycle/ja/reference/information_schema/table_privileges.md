---
displayed_sidebar: "Japanese"
---

# table_privileges

`table_privileges`は、テーブル権限に関する情報を提供します。

`table_privileges`には、以下のフィールドが提供されます：

| **フィールド**   | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されるユーザーの名前。                                |
| TABLE_CATALOG  | テーブルが所属するカタログの名前。値は常に `def` です。                   |
| TABLE_SCHEMA   | テーブルが所属するデータベースの名前。                            |
| TABLE_NAME     | テーブルの名前。                                                |
| PRIVILEGE_TYPE | 付与された権限。値はテーブルレベルで付与できる権限のどれかです。         |
| IS_GRANTABLE   | ユーザーが `GRANT OPTION` 権限を持っている場合は `YES`、そうでない場合は `NO`。出力には `PRIVILEGE_TYPE='GRANT OPTION'` の別の行として `GRANT OPTION` はリストされません。   |