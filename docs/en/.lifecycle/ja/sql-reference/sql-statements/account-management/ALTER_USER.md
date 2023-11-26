---
displayed_sidebar: "Japanese"
---

# ALTER USER（ユーザーの変更）

## 説明

パスワード、認証方法、デフォルトのロールなど、ユーザー情報を変更します。

> 個々のユーザーは、自分自身の情報を変更するためにこのコマンドを使用できます。`user_admin`は他のユーザーの情報を変更するためにこのコマンドを使用できます。

## 構文

```SQL
ALTER USER user_identity [auth_option] [default_role]
```

## パラメータ

- `user_identity`は、`username@'userhost'`の形式で構成される2つの部分、「user_name」と「host」からなります。 「host」の部分では、曖昧な一致のために`%`を使用できます。 「host」が指定されていない場合、デフォルトで`%`が使用され、ユーザーは任意のホストからStarRocksに接続できます。

- `auth_option`は認証方法を指定します。現在、3つの認証方法がサポートされています：StarRocksネイティブパスワード、mysql_native_password、および「authentication_ldap_simple」。StarRocksネイティブパスワードは、論理的にはmysql_native_passwordと同じですが、構文が若干異なります。1つのユーザーIDは1つの認証方法のみを使用できます。ALTER USERを使用してユーザーのパスワードと認証方法を変更できます。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **認証方法**                  | **ユーザー作成時のパスワード** | **ログイン時のパスワード** |
    | ---------------------------- | ------------------------------ | ---------------------- |
    | ネイティブパスワード           | 平文または暗号文                | 平文                   |
    | `mysql_native_password BY`   | 平文                           | 平文                   |
    | `mysql_native_password WITH` | 暗号文                         | 平文                   |
    | `authentication_ldap_simple` | 平文                           | 平文                   |

> 注意：StarRocksは、ユーザーのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE`

   ```SQL
    -- 指定したロールをデフォルトのロールとして設定します。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- このユーザーに割り当てられるロールを含む、ユーザーのすべてのロールをデフォルトのロールとして設定します。
    DEFAULT ROLE ALL
    -- デフォルトのロールは設定されませんが、ユーザーログイン後にはパブリックロールが有効になります。
    DEFAULT ROLE NONE
    ```

  デフォルトのロールを設定する前に、すべてのロールがユーザーに割り当てられていることを確認してください。ユーザーが再ログインすると、ロールが自動的にアクティブになります。

## 例

例1：ユーザーのパスワードを平文のパスワードに変更します。

```SQL
ALTER USER 'jack' IDENTIFIED BY '123456';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例2：ユーザーのパスワードを暗号文のパスワードに変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 暗号化されたパスワードは、password()関数を使用して取得できます。

例3：認証方法をLDAPに変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例4：認証方法をLDAPに変更し、LDAPのユーザーの識別名（DN）を指定します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例5：ユーザーのデフォルトのロールを`db_admin`と`user_admin`に変更します。ユーザーにこれらの2つのロールが割り当てられている必要があります。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

例6：ユーザーのすべてのロール、このユーザーに割り当てられるロールをデフォルトのロールとして設定します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

例7：ユーザーのすべてのデフォルトのロールをクリアします。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注意：デフォルトでは、`public`ロールはユーザーに対して有効になっています。

## 参照

- [CREATE USER](CREATE_USER.md)
- [GRANT](GRANT.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)
