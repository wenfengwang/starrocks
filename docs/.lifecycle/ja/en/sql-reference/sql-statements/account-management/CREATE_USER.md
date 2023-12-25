---
displayed_sidebar: English
---

# CREATE USER

## 説明

StarRocksのユーザーを作成します。StarRocksでは、「user_identity」がユーザーを一意に識別します。

### 構文

```SQL
CREATE USER <user_identity> [auth_option] [DEFAULT ROLE <role_name>[, <role_name>, ...]]
```

## パラメーター

- `user_identity` は、"user_name" と "host" の2つの部分から構成され、`username@'userhost'`の形式です。"host"部分には、あいまい一致のために`%`を使用できます。"host"が指定されていない場合、デフォルトで`%`が使用され、ユーザーは任意のホストからStarRocksに接続できることを意味します。

- `auth_option` は認証方法を指定します。現在、StarRocksネイティブパスワード、`mysql_native_password`、および`authentication_ldap_simple`の3つの認証方法がサポートされています。StarRocksネイティブパスワードは、論理的には`mysql_native_password`と同じですが、構文が若干異なります。一つのユーザーIDで使用できる認証方法は一つだけです。

    ```SQL
    auth_option: {
        IDENTIFIED BY 'auth_string'
        IDENTIFIED WITH mysql_native_password BY 'auth_string'
        IDENTIFIED WITH mysql_native_password AS 'auth_string'
        IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
        
    }
    ```

    | **認証方法**                     | **ユーザー作成時のパスワード** | **ログイン時のパスワード** |
    | -------------------------------- | ------------------------------ | ---------------------- |
    | ネイティブパスワード              | 平文または暗号文               | 平文                    |
    | `mysql_native_password BY`       | 平文                           | 平文                    |
    | `mysql_native_password AS`       | 暗号文                         | 平文                    |
    | `authentication_ldap_simple`     | 平文                           | 平文                    |

> 注: StarRocksはユーザーのパスワードを保存する前に暗号化します。

- `DEFAULT ROLE <role_name>[, <role_name>, ...]`: このパラメータが指定された場合、ロールはユーザーに自動的に割り当てられ、ユーザーがログインするとデフォルトでアクティブになります。指定されていない場合、このユーザーはいかなる権限も持ちません。指定されたすべてのロールが既に存在していることを確認してください。

## 例

例 1: ホストを指定せずに平文パスワードを使用してユーザーを作成する例。これは`jack@'%'`と同等です。

```SQL
CREATE USER 'jack' IDENTIFIED BY '123456';
```

例 2: 平文パスワードを使用してユーザーを作成し、ユーザーが`'172.10.1.10'`からログインできるようにする例。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例 3: 暗号文パスワードを使用してユーザーを作成し、ユーザーが`'172.10.1.10'`からログインできるようにする例。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 注: 暗号化されたパスワードは`password()`関数を使用して取得できます。

例 4: ドメイン名`'example_domain'`からログインを許可するユーザーを作成する例。

```SQL
CREATE USER 'jack'@['example_domain'] IDENTIFIED BY '123456';
```

例 5: LDAP認証を使用するユーザーを作成する例。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例 6: LDAP認証を使用し、LDAP内のユーザーの識別名(DN)を指定してユーザーを作成する例。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例 7: `'192.168'`サブネットからのログインを許可し、`db_admin`と`user_admin`をデフォルトロールとして設定するユーザーを作成する例。

```SQL
CREATE USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```
