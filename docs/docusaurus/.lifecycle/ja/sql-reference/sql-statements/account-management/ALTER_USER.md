---
displayed_sidebar: "Japanese"
---

# ユーザの変更

## 説明

ユーザの情報（パスワード、認証方法、デフォルトのロールなど）を変更します。

> 個々のユーザは、このコマンドを使用して自分自身の情報を変更できます。 `user_admin` は他のユーザの情報を変更するためにこのコマンドを使用できます。

## 構文

```SQL
ALTER USER ユーザ識別子 [auth_option] [default_role]
```

## パラメータ

- `user_identity` は `username@'userhost'` の形式で、"user_name" と "host" の2つの部分で構成されます。"host" 部分では、不明瞭な一致のために `%` を使用できます。 "host" が指定されていない場合、デフォルトで "%" が使用され、ユーザはどのホストからでもStarRocksに接続できることを意味します。 

- `auth_option` は認証方法を指定します。現在、3つの認証方法がサポートされています。StarRocksネイティブパスワード、mysql_native_password、および "authentication_ldap_simple" です。StarRocksネイティブパスワードは、論理的には mysql_native_password と同じですが、構文が若干異なります。1つのユーザ識別子は1つの認証方法しか使用できません。ALTER USER を使用して、ユーザのパスワードと認証方法を変更できます。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **認証方法**      | **ユーザ作成のためのパスワード** | **ログインのためのパスワード** |
    | ------------------ | --------------------------------- | ------------------------------ |
    | ネイティブパスワード | 平文または暗号文                    | 平文                        |
    | `mysql_native_password BY` | 平文                          | 平文                        |
    | `mysql_native_password WITH` | 暗号文                        | 平文                        |
    | `authentication_ldap_simple` | 平文                          | 平文                        |

> 注意：StarRocksは、ユーザのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE`

  ```SQL
    -- 指定されたロールをデフォルトのロールとして設定します。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- このユーザに割り当てられるすべてのロールを、デフォルトのロールとして設定します。
    DEFAULT ROLE ALL
    -- デフォルトのロールを設定せず、ユーザログイン後もパブリックロールが有効のままになります。
    DEFAULT ROLE NONE
  ```

  デフォルトのロールを設定する前に、すべてのロールがユーザに割り当てられていることを確認してください。ユーザが再ログインすると、ロールは自動的に有効になります。

## 例

例1：ユーザのパスワードを平文パスワードに変更します。

```SQL
ALTER USER 'jack' IDENTIFIED BY '123456';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例2：ユーザのパスワードを暗号化パスワードに変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 暗号化されたパスワードは、password() 関数を使用して取得できます。

例3：認証方法をLDAPに変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例4：認証方法をLDAPに変更し、LDAP内のユーザの識別名（DN）を指定します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例5：ユーザのデフォルトのロールを `db_admin` と `user_admin` に変更します。ユーザにはこれらの2つのロールが割り当てられている必要があります。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

例6：このユーザに割り当てられるすべてのロールをデフォルトのロールとして設定します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

例7：ユーザのデフォルトのロールをすべてクリアします。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注意：デフォルトでは、`public` ロールはユーザに対して有効のままです。

## 参照

- [CREATE USER](CREATE_USER.md)
- [GRANT](GRANT.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)