---
displayed_sidebar: "Japanese"
---

# user_privileges

`user_privileges`はユーザーの特権に関する情報を提供します。

`user_privileges`には、以下のフィールドが提供されています。

| **フィールド**  | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 特権が付与されているユーザーの名前。                             |
| TABLE_CATALOG  | カタログの名前。この値は常に `def` です。                         |
| PRIVILEGE_TYPE | 付与された特権。この値はグローバルレベルで付与できる特権である場合があります。             |
| IS_GRANTABLE   | ユーザーが `GRANT OPTION` 特権を持っている場合は `YES`、そうでない場合は `NO` です。 出力には、`PRIVILEGE_TYPE='GRANT OPTION'` として独立した行に `GRANT OPTION` をリストしません。 |