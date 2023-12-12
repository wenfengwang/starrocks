---
displayed_sidebar: "Japanese"
---

# パスワードの設定

## 説明

### 構文

```SQL
SET PASSWORD [FOR user_identity] =
[PASSWORD('plain password')]|['hashed password']
```

SET PASSWORDコマンドは、ユーザーのログインパスワードを変更するために使用できます。[FOR user_identity]フィールドが存在しない場合、現在のユーザーのパスワードが変更されます。

```plain text
user_identityは、CREATE USERを使用してユーザーを作成する際に指定したuser_identityと完全に一致する必要があります。そうでない場合、ユーザーは存在しないと報告されます。user_identityが指定されていない場合、現在のユーザーは'username'@'ip'であり、これは任意のuser_identityに一致しない可能性があります。現在のユーザーはSHOW GRANTSを使用して表示できます。 
```

PASSWORD()は平文のパスワードを入力し、文字列の直接使用は暗号化されたパスワードの送信を必要とします。

他のユーザーのパスワードを変更するには、管理者権限が必要です。

## 例

1. 現在のユーザーのパスワードを変更する

    ```SQL
    SET PASSWORD = PASSWORD('123456')
    SET PASSWORD = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```

2. 指定したユーザーのパスワードを変更する

    ```SQL
    SET PASSWORD FOR 'jack'@'192.%' = PASSWORD('123456')
    SET PASSWORD FOR 'jack'@['domain'] = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```