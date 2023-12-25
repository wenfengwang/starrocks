---
displayed_sidebar: English
---

# table_privileges

`table_privileges` はテーブルの権限に関する情報を提供します。

`table_privileges` には以下のフィールドが含まれています：

| **フィールド**      | **説明**                                              |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 特権が付与されるユーザーの名前です。      |
| TABLE_CATALOG  | テーブルが属するカタログの名前です。この値は常に `def` となります。 |
| TABLE_SCHEMA   | テーブルが属するデータベースの名前です。         |
| TABLE_NAME     | テーブルの名前です。                                       |
| PRIVILEGE_TYPE | 付与される特権の種類です。値はテーブルレベルで付与可能な任意の特権になります。 |
| IS_GRANTABLE   | `YES` の場合、ユーザーは `GRANT OPTION` 特権を持っています。`NO` の場合は持っていません。出力には `GRANT OPTION` が `PRIVILEGE_TYPE='GRANT OPTION'` として別行でリストされることはありません。 |
