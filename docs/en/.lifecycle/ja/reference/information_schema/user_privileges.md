---
displayed_sidebar: "Japanese"
---

# ユーザー権限

`user_privileges`はユーザーの権限に関する情報を提供します。

`user_privileges`には以下のフィールドが提供されます：

| **フィールド**    | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されるユーザーの名前。                                      |
| TABLE_CATALOG  | カタログの名前。この値は常に`def`です。                                |
| PRIVILEGE_TYPE | 付与された権限。値はグローバルレベルで付与できる任意の権限です。                          |
| IS_GRANTABLE   | ユーザーが`GRANT OPTION`権限を持っている場合は`YES`、そうでない場合は`NO`です。出力には`PRIVILEGE_TYPE='GRANT OPTION'`の別の行として`GRANT OPTION`がリストされません。                           |
