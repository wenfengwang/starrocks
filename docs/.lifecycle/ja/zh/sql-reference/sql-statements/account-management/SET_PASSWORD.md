---
displayed_sidebar: Chinese
---

# パスワードの設定

## 機能

SET PASSWORD コマンドは、ユーザーのログインパスワードを変更するために使用されます。

また、[ALTER USER](ALTER_USER.md) を使用してユーザーパスワードを変更することもできます。

> **注意**
>
> - どのユーザーも自分のパスワードをリセットできます。
> - `user_admin` ロールを持つユーザーのみが他のユーザーのパスワードを変更できます。
> - root ユーザーのパスワードは、root ユーザー自身のみがリセットできます。詳細は [ユーザー権限の管理 - 失われた root パスワードのリセット](../../../administration/User_privilege.md#ユーザーの変更) を参照してください。

## 文法

```SQL
SET PASSWORD [FOR user_identity] =
[PASSWORD('plain password')]|['hashed password']
```

`[FOR user_identity]` が省略された場合、現在のユーザーのパスワードが変更されます。

ここでの `user_identity` の文法は [CREATE USER](../account-management/CREATE_USER.md) のセクションと同じです。そして、`CREATE USER` で作成された `user_identity` でなければなりません。そうでない場合は、ユーザーが存在しないというエラーが発生します。`user_identity` を指定しない場合、現在のユーザーは `'username'@'ip'` となりますが、この現在のユーザーは `user_identity` と一致しない可能性があります。`SHOW GRANTS;` を使用して現在のユーザーを確認できます。

**PASSWORD()** を使用すると、平文のパスワードが入力されます。一方、文字列を直接使用する場合は、ハッシュ化されたパスワードを渡す必要があります。

## 例

例1： 現在のユーザーのパスワードを変更します。

```SQL
SET PASSWORD = PASSWORD('123456');
SET PASSWORD = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

例2： 特定のユーザーのパスワードを変更します。

```SQL
SET PASSWORD FOR 'jack'@'192.%' = PASSWORD('123456');
SET PASSWORD FOR 'jack'@['domain'] = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```
