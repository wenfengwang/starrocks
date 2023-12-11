---
displayed_sidebar: "Japanese"
---

# ユーザーの作成

## 説明

StarRocksのユーザーを作成します。StarRocksでは、"user_identity"がユーザーを一意に識別します。

### 構文

```SQL
CREATE USER <user_identity> [auth_option] [DEFAULT ROLE <role_name>[, <role_name>, ...]]
```

## パラメーター

- `user_identity` は `username@'userhost'` の形式で、"user_name" と "host" の2つの部分で構成されます。 "host" 部分では、あいまいなマッチのために `%` を使用できます。 "host" が指定されていない場合、デフォルトで "%" が使用され、ユーザーは任意のホストからStarRocksに接続できます。

- `auth_option` は認証メソッドを指定します。現在、3つの認証メソッドがサポートされています。StarRocksネイティブパスワード、mysql_native_password、"authentication_ldap_simple" です。StarRocksネイティブパスワードは、ロジック上ではmysql_native_passwordと同じですが、構文が若干異なります。1つのユーザー識別子は1つの認証メソッドのみを使用できます。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **認証メソッド**       | **ユーザー作成のためのパスワード** | **ログインのためのパスワード** |
    | ----------------------- | -------------------------------- | ------------------------------ |
    | ネイティブパスワード    | 平文または暗号文                  | 平文                          |
    | `mysql_native_password BY`   | 平文                             | 平文                          |
    | `mysql_native_password WITH` | 暗号文                           | 平文                          |
    | `authentication_ldap_simple` | 平文                             | 平文                          |

> 注意：StarRocksはユーザーのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE <role_name>[, <role_name>, ...]`：このパラメータが指定されていると、役割がユーザーに自動的に割り当てられ、ユーザーがログインしたときにデフォルトでアクティブになります。指定されていない場合、このユーザーには特権がありません。指定されたすべての役割がすでに存在することを確認してください。

## 例

例1：ホストが指定されていない平文のパスワードを使用してユーザーを作成し、これは `jack@'%'` に相当します。

```SQL
CREATE USER 'jack' IDENTIFIED BY '123456';
```

例2：平文のパスワードを使用し、ユーザーが `'172.10.1.10'` からログインできるようにするユーザーを作成します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例3：暗号文のパスワードを使用し、ユーザーが `'172.10.1.10'` からログインできるようにします。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 注：暗号化されたパスワードは password() 関数を使用して取得できます。

例4：ドメイン名 'example_domain' からログインできるユーザーを作成します。

```SQL
CREATE USER 'jack'@['example_domain'] IDENTIFIED BY '123456';
```

例5：LDAP認証を使用するユーザーを作成します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例6：LDAP認証を使用し、LDAPのユーザーの識別名（DN）を指定します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例7：'192.168' サブネットからログインできるユーザーを作成し、そのユーザーのデフォルト役割として `db_admin` および `user_admin` を設定します。

```SQL
CREATE USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```