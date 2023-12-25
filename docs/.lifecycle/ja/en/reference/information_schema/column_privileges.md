---
displayed_sidebar: English
---

# column_privileges

`column_privileges` は、現在有効なロールによって、または現在有効なロールに付与された列のすべての権限を識別します。

`column_privileges` には、以下のフィールドが提供されています。

| **フィールド** | **説明**                                                      |
| -------------- | ------------------------------------------------------------- |
| GRANTEE        | 権限が付与されたユーザーの名前。                              |
| TABLE_CATALOG  | 列を含むテーブルが属するカタログの名前。この値は常に `def` です。 |
| TABLE_SCHEMA   | 列を含むテーブルが属するデータベースの名前。                  |
| TABLE_NAME     | 列を含むテーブルの名前。                                      |
| COLUMN_NAME    | 列の名前。                                                    |
| PRIVILEGE_TYPE | 付与された権限。値は、列レベルで付与可能な任意の権限です。各行は単一の権限をリストするため、権限を受けたユーザーが保持する列権限ごとに1行があります。 |
| IS_GRANTABLE   | `YES` はユーザーが `GRANT OPTION` 権限を持っている場合、 `NO` はそうでない場合。出力には `GRANT OPTION` を `PRIVILEGE_TYPE='GRANT OPTION'` として別の行としてリストすることはありません。 |
