---
displayed_sidebar: English
---

# user_privileges

`user_privileges` はユーザー権限に関する情報を提供します。

`user_privileges` には以下のフィールドが提供されています：

| **フィールド** | **説明**                                                      |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 特権が付与されるユーザーの名前です。                          |
| TABLE_CATALOG  | カタログの名前です。この値は常に `def` となります。          |
| PRIVILEGE_TYPE | 付与される特権の種類です。値はグローバルレベルで付与可能な任意の特権になり得ます。 |
| IS_GRANTABLE   | `YES` はユーザーが `GRANT OPTION` 特権を持っている場合、`NO` はそうでない場合です。出力では `GRANT OPTION` は `PRIVILEGE_TYPE='GRANT OPTION'` として別行でリストされません。 |
