---
displayed_sidebar: English
---

# SET PASSWORD

## 説明

### 構文

```SQL
SET PASSWORD [FOR user_identity] =
[PASSWORD('plain password')]|['hashed password']
```

SET PASSWORDコマンドは、ユーザーのログインパスワードを変更するために使用されます。フィールド [FOR user_identity] が存在しない場合、現在のユーザーのパスワードが変更されます。

```plain text
user_identityはCREATE USERを使用してユーザーを作成する際に指定したuser_identityと完全に一致している必要があります。そうでない場合、ユーザーは存在しないと報告されます。user_identityが指定されていない場合、現在のユーザーは 'username'@'ip' となり、これは任意のuser_identityと一致しない可能性があります。現在のユーザーはSHOW GRANTSを通じて確認できます。
```

PASSWORD()はプレーンテキストパスワードを入力するために使用されますが、文字列を直接使用する場合は、暗号化されたパスワードの送信が必要です。

他のユーザーのパスワードを変更するには、管理者権限が必要です。

## 例

1. 現在のユーザーのパスワードを変更する

    ```SQL
    SET PASSWORD = PASSWORD('123456')
    SET PASSWORD = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```

2. 指定されたユーザーのパスワードを変更する

    ```SQL
    SET PASSWORD FOR 'jack'@'192.%' = PASSWORD('123456')
    SET PASSWORD FOR 'jack'@'domain' = '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'
    ```
