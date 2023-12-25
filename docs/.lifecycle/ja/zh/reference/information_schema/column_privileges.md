---
displayed_sidebar: Chinese
---

# column_privileges

`column_privileges` は、現在有効なロールに付与された、または現在有効なロールが付与したすべての列権限を識別するために使用されます。

`column_privileges` は以下のフィールドを提供します：

| フィールド     | 説明                                                         |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されたユーザーの名前。                             |
| TABLE_CATALOG  | 列が含まれるテーブルが属するカタログの名前。この値は常に `def` です。 |
| TABLE_SCHEMA   | 列が含まれるテーブルが属するスキーマ（データベース）の名前。 |
| TABLE_NAME     | 列が含まれるテーブルの名前。                                 |
| COLUMN_NAME    | 列の名前。                                                   |
| PRIVILEGE_TYPE | 付与された権限。この値は列レベルで付与される任意の権限になり得ます。各行に1つの権限がリストされるため、付与された各列権限について1行が存在します。 |
| IS_GRANTABLE   | ユーザーが `GRANT OPTION` 権限を持っている場合は `YES`、そうでない場合は `NO`。出力は `GRANT OPTION` を `PRIVILEGE_TYPE` として `'GRANT OPTION'` の単独の行としてはリストしません。 |
