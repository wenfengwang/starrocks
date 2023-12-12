---
displayed_sidebar: "Japanese"
---

# schema_privileges

`schema_privileges`にはデータベースの権限に関する情報が提供されています。

`schema_privileges`には次のフィールドが提供されています：

| **Field**      | **Description**                                              |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されているユーザーの名前です。      |
| TABLE_CATALOG  | スキーマが属するカタログの名前です。この値は常に`def`です。 |
| TABLE_SCHEMA   | スキーマの名前です。                                      |
| PRIVILEGE_TYPE | 付与された権限です。各行は単一の権限をリストし、従って、GRANTEEが保持するスキーマ権限ごとに1行あります。 |
| IS_GRANTABLE   | ユーザーが`GRANT OPTION`権限を持っている場合は`YES`、そうでない場合は`NO`です。出力には、`PRIVILEGE_TYPE='GRANT OPTION'`の別々の行として`GRANT OPTION`がリストされません。 |