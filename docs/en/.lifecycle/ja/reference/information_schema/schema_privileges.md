---
displayed_sidebar: "Japanese"
---

# schema_privileges

`schema_privileges`はデータベースの権限に関する情報を提供します。

`schema_privileges`には以下のフィールドが提供されます：

| **フィールド**    | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されるユーザーの名前。                                      |
| TABLE_CATALOG  | スキーマが所属するカタログの名前。この値は常に`def`です。 |
| TABLE_SCHEMA   | スキーマの名前。                                      |
| PRIVILEGE_TYPE | 付与された権限。各行には1つの権限がリストされるため、権限を保持しているスキーマごとに1行あります。 |
| IS_GRANTABLE   | ユーザーが`GRANT OPTION`権限を持っている場合は`YES`、そうでない場合は`NO`です。出力では`PRIVILEGE_TYPE='GRANT OPTION'`の別の行として`GRANT OPTION`はリストされません。 |
