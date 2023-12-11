---
displayed_sidebar: "Japanese"
---

# ユーザーの削除

## 説明

指定されたユーザーを削除します。

## 構文

```sql
 DROP USER '<user_identity>'

`user_identity`:

 user@'host'
user@['domain']
```

## 例

ユーザーjack@'192.%'を削除します。

```sql
DROP USER 'jack'@'192.%'
```