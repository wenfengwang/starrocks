---
displayed_sidebar: English
---

# schema_privileges

`schema_privileges` はデータベース権限に関する情報を提供します。

`schema_privileges` には以下のフィールドが提供されています：

| **フィールド** | **説明**                                                    |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 特権が付与されるユーザーの名前です。                         |
| TABLE_CATALOG  | スキーマが属するカタログの名前です。この値は常に `def` となります。 |
| TABLE_SCHEMA   | スキーマの名前です。                                        |
| PRIVILEGE_TYPE | 付与された特権です。各行は単一の特権をリストするため、権限を持つユーザーごとにスキーマ特権が1行ずつあります。 |
| IS_GRANTABLE   | ユーザーが `GRANT OPTION` 権限を持っている場合は `YES`、そうでない場合は `NO` です。出力には `GRANT OPTION` を `PRIVILEGE_TYPE='GRANT OPTION'` として別の行としてリストすることはありません。 |
