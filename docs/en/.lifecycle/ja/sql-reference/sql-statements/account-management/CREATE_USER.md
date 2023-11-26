---
displayed_sidebar: "Japanese"
---

# CREATE USER（ユーザーの作成）

## 説明

StarRocksのユーザーを作成します。StarRocksでは、「user_identity」がユーザーを一意に識別します。

### 構文

```SQL
CREATE USER <user_identity> [auth_option] [DEFAULT ROLE <role_name>[, <role_name>, ...]]
```

## パラメータ

- `user_identity`は、「user_name」と「host」の2つの部分で構成されており、`username@'userhost'`の形式で指定します。 "host"の部分では、曖昧なマッチングのために`%`を使用することができます。 "host"が指定されていない場合、デフォルトで`%`が使用され、ユーザーは任意のホストからStarRocksに接続することができます。

- `auth_option`は、認証方法を指定します。現在、3つの認証方法がサポートされています：StarRocksネイティブパスワード、mysql_native_password、および「authentication_ldap_simple」です。StarRocksネイティブパスワードは、論理的にはmysql_native_passwordと同じですが、構文が若干異なります。1つのユーザーIDは、1つの認証方法のみを使用できます。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **認証方法**                | **ユーザー作成用パスワード** | **ログイン用パスワード** |
    | ---------------------------- | ------------------------------ | ---------------------- |
    | ネイティブパスワード              | 平文または暗号文        | 平文              |
    | `mysql_native_password BY`   | 平文                      | 平文              |
    | `mysql_native_password WITH` | 暗号文                     | 平文              |
    | `authentication_ldap_simple` | 平文                      | 平文              |

> 注意：StarRocksは、ユーザーのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE <role_name>[, <role_name>, ...]`：このパラメータが指定されている場合、ユーザーにはロールが自動的に割り当てられ、ユーザーがログインしたときにデフォルトで有効になります。指定されていない場合、このユーザーには特権がありません。指定されたすべてのロールが既に存在することを確認してください。

## 例

例1：ホストが指定されていない平文パスワードを使用してユーザーを作成します。これは`jack@'%'`と同等です。

```SQL
CREATE USER 'jack' IDENTIFIED BY '123456';
```

例2：平文パスワードを使用してユーザーを作成し、ユーザーが`'172.10.1.10'`からログインできるようにします。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例3：暗号文パスワードを使用してユーザーを作成し、ユーザーが`'172.10.1.10'`からログインできるようにします。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 注意：パスワード()関数を使用して、暗号化されたパスワードを取得できます。

例4：ドメイン名「example_domain」からログインできるユーザーを作成します。

```SQL
CREATE USER 'jack'@['example_domain'] IDENTIFIED BY '123456';
```

例5：LDAP認証を使用するユーザーを作成します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例6：LDAP認証を使用し、LDAPのユーザーの識別名（DN）を指定するユーザーを作成します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例7：「192.168」サブネットからログインできるユーザーを作成し、デフォルトのロールとして`db_admin`と`user_admin`を設定します。

```SQL
CREATE USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```
