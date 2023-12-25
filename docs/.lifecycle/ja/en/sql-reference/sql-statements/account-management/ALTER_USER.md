---
displayed_sidebar: English
---

# ALTER USER

## 説明

パスワード、認証方法、デフォルトロールなどのユーザー情報を変更します。

> 個々のユーザーは、このコマンドを使用して自己の情報を変更することができます。`user_admin`はこのコマンドを使用して他のユーザーの情報を変更することができます。

## 構文

```SQL
ALTER USER user_identity [auth_option] [default_role]
```

## パラメーター

- `user_identity`は"user_name"と"host"の2部分から構成され、`username@'userhost'`の形式を取ります。"host"部分では、`%`を使用してあいまいな一致を行うことができます。"host"が指定されていない場合は、デフォルトで`%`が使用され、ユーザーは任意のホストからStarRocksに接続できることを意味します。

- `auth_option`は認証方法を指定します。現在、StarRocksネイティブパスワード、`mysql_native_password`、`authentication_ldap_simple`の3つの認証方法がサポートされています。StarRocksネイティブパスワードは、ロジックは`mysql_native_password`と同じですが、構文が若干異なります。ユーザーIDごとに1つの認証方法のみ使用できます。`ALTER USER`を使用して、ユーザーのパスワードと認証方法を変更できます。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
    }
    ```

    | **認証方法**                     | **ユーザー作成時のパスワード** | **ログイン時のパスワード** |
    | -------------------------------- | ------------------------------ | -------------------------- |
    | ネイティブパスワード             | 平文または暗号文              | 平文                        |
    | `mysql_native_password BY`       | 平文                          | 平文                        |
    | `mysql_native_password AS`       | 暗号文                        | 平文                        |
    | `authentication_ldap_simple`     | 平文                          | 平文                        |

> 注: StarRocksは、ユーザーのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE`

   ```SQL
    -- 指定されたロールをデフォルトロールとして設定します。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- ユーザーに割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトロールとして設定します。
    DEFAULT ROLE ALL
    -- デフォルトロールは設定されませんが、ユーザーログイン後もpublicロールは有効です。
    DEFAULT ROLE NONE
    ```

  `ALTER USER`を実行してデフォルトロールを設定する前に、すべてのロールがユーザーに割り当てられていることを確認してください。ロールは、ユーザーが再ログインした後に自動的に有効になります。

## 例

例 1: ユーザーのパスワードを平文のパスワードに変更します。

```SQL
ALTER USER 'jack' IDENTIFIED BY '123456';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例 2: ユーザーのパスワードを暗号文のパスワードに変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 暗号化されたパスワードは`password()`関数を使用して取得できます。

例 3: 認証方法をLDAPに変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例 4: 認証方法をLDAPに変更し、LDAP内のユーザーの識別名(DN)を指定します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例 5: ユーザーのデフォルトロールを`db_admin`と`user_admin`に変更します。ユーザーにこれらのロールが割り当てられている必要があります。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

例 6: ユーザーに割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトロールとして設定します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

例 7: ユーザーのすべてのデフォルトロールをクリアします。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注: デフォルトでは、`public`ロールはユーザーに対して有効です。

## 参照

- [CREATE USER](CREATE_USER.md)
- [GRANT](GRANT.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)
