---
displayed_sidebar: "Japanese"
---

# column_privileges

`column_privileges`は、現在有効なロールによって付与された列へのすべての権限、または現在有効なロールによって付与された権限を識別します。

`column_privileges`には、次のフィールドが提供されています。

| **フィールド**   | **説明**                                                      |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されているユーザーの名前。                         |
| TABLE_CATALOG  | 列を含むテーブルが属するカタログの名前。この値は常に `def` です。|
| TABLE_SCHEMA   | 列を含むテーブルが属するデータベースの名前。                 |
| TABLE_NAME     | 列を含むテーブルの名前。                                       |
| COLUMN_NAME    | 列の名前。                                                    |
| PRIVILEGE_TYPE | 付与された権限。値は列レベルで付与できる任意の権限です。各行は単一の権限をリストするため、`PRIVILEGE_TYPE`ごとに1行があります。 |
| IS_GRANTABLE   | `GRANT OPTION` 権限を持っている場合は `YES`、それ以外の場合は `NO`。出力には、`PRIVILEGE_TYPE='GRANT OPTION'` の別々の行として `GRANT OPTION` がリストされません。 |