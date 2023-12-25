---
displayed_sidebar: Chinese
---

# DROP USER

## 機能

ユーザーを削除します。

## 文法

```sql
 -- コマンド
 DROP USER 'user_identity'

 --パラメータ説明
user_identity:user@'host'
```

 指定された `user identity` を削除します。`user identity` は `user_name` と `host` の二つの部分で構成されており、`user_name` はユーザー名です。`host` はユーザーが接続するホストアドレスを識別します。

## 例

ユーザー jack@'192.%' を削除します。

```sql
DROP USER 'jack'@'192.%';
```
