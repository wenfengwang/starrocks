---
displayed_sidebar: "Japanese"
---

# ALTER USER（ユーザーの変更）

## 説明

ユーザーの情報（パスワード、認証方法、デフォルトのロールなど）を変更します。

> 個々のユーザーは自分自身の情報を変更するためにこのコマンドを使用できます。 `user_admin` は他のユーザーの情報を変更するためにこのコマンドを使用できます。

## 構文

```SQL
ALTER USER ユーザー識別子 [auth_option] [default_role]
```

## パラメーター

- `ユーザー識別子` は `ユーザー名@'ユーザーホスト'` の形式で構成されており、2つの部分で構成されています。 "host" 部分には、曖昧な一致のために `%` を使用できます。 "host" が指定されていない場合、デフォルトで `%` が使用され、ユーザーはどのホストからでも StarRocks に接続できます。

- `auth_option` は認証方法を指定します。現在、3つの認証方法がサポートされています: StarRocks ネイティブパスワード、mysql_native_password、および "authentication_ldap_simple"。 StarRocks ネイティブパスワードはロジックでは mysql_native_password と同じですが、構文に若干の違いがあります。 1つのユーザー識別子は1つの認証方法のみを使用できます。 ALTER USER を使用してユーザーのパスワードと認証方法を変更できます。

    ```SQL
    auth_option: {
        'auth_string' で識別
        mysql_native_password の 'auth_string' で識別
        mysql_native_password では 'auth_string' として識別
        authentication_ldap_simple では 'auth_string' として識別
        
    }
    ```

    | **認証方法**             | **ユーザー作成のパスワード** | **ログインのパスワード** |
    | --------------------------- | ------------------------------ | ---------------------- |
    | ネイティブパスワード           | 平文または暗号文                  | 平文                  |
    | `mysql_native_password` では    | 平文                              | 平文                  |
    | `mysql_native_password` では   | 暗号文                             | 平文                  |
    | `authentication_ldap_simple` では | 平文                              | 平文                  |

> 注: StarRocks はユーザーのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE`

   ```SQL
    -- 指定されたロールをデフォルトの役割として設定します。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- このユーザーに割り当てられるロールを含む、すべてのロールをデフォルトの役割として設定します。 
    DEFAULT ROLE ALL
    -- ロールをデフォルトに設定しないが、ユーザーログイン後には public ロールが有効のままです。
    DEFAULT ROLE NONE
    ```

  デフォルトのロールを設定する前に、すべてのロールがユーザーに割り当てられていることを確認してください。 ユーザーが再度ログインすると、ロールは自動的にアクティブ化されます。

## 例

例1: ユーザーのパスワードを平文パスワードに変更します。

```SQL
ALTER USER 'jack' '123456' で識別;
ALTER USER jack@'172.10.1.10' は mysql_native_password の '123456' で識別;
```

例2: ユーザーのパスワードを暗号文のパスワードに変更します。

```SQL
ALTER USER jack@'172.10.1.10' でパスワード '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' で識別;
ALTER USER jack@'172.10.1.10' は mysql_native_password として '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' で識別;
```

> 暗号化されたパスワードは password() 関数を使用して取得できます。

例3: 認証方法を LDAP に変更します。

```SQL
ALTER USER jack@'172.10.1.10' は authentication_ldap_simple として識別;
```

例4: 認証方法を LDAP に変更し、LDAP でのユーザーの識別名（DN）を指定します。

```SQL
CREATE USER jack@'172.10.1.10' は authentication_ldap_simple として 'uid=jack,ou=company,dc=example,dc=com' で識別;
```

例5: ユーザーのデフォルトの役割を `db_admin` および `user_admin` に変更します。 ユーザーはこれらの2つのロールが割り当てられている必要があります。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

例6: このユーザーに割り当てられるロールを含め、ユーザーのすべてのロールをデフォルトのロールとして設定します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

例7: ユーザーのすべてのデフォルトロールをクリアします。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注: デフォルトで、`public` ロールはユーザーに対して有効です。

## 参照

- [CREATE USER](CREATE_USER.md)
- [GRANT](GRANT.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)