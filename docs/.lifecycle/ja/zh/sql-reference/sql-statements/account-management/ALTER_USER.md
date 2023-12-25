---
displayed_sidebar: Chinese
---

# ALTER USER

## 機能

StarRocks のユーザー情報を変更します。例えば、ユーザーのパスワード、認証方法、またはデフォルトのロールです。

> **注意**
>
> すべてのユーザーは自分の情報を変更することができます。`user_admin` のみが他のユーザーの情報を変更できます。

## 文法

```SQL
ALTER USER user_identity [auth_option] [default_role]
```

## パラメータ説明

- `user_identity`：ユーザー識別子。ログインIP（userhost）とユーザー名（username）で構成され、`username@'userhost'` の形式で書きます。`userhost` は `%` を使用して曖昧なマッチングを行うことができます。`userhost` を指定しない場合、デフォルトは `%` となり、任意のホストから `username` を使用して StarRocks に接続するユーザーを意味します。

- `auth_option`：ユーザーの認証方法。現在、StarRocks はネイティブパスワード、mysql_native_password、LDAP の3種類の認証方法をサポートしています。ネイティブパスワードと mysql_native_password の認証方法は内部ロジックが同じで、具体的な設定構文にわずかな違いがあります。同じ user_identity は一つの認証方法のみを使用できます。ALTER 文を使用してユーザーの認証方法とパスワードを変更できます。

    ```SQL
      auth_option: {
          IDENTIFIED BY 'auth_string'
          IDENTIFIED WITH mysql_native_password BY 'auth_string'
          IDENTIFIED WITH mysql_native_password AS 'auth_string'
          IDENTIFIED WITH authentication_ldap_simple AS 'auth_string'
          
      }
      ```

      | **認証方法**                   | **ユーザー作成時のパスワード** | **ユーザーログイン時のパスワード** |
      | ---------------------------- | ---------------------------- | ---------------------------- |
      | ネイティブパスワード           | 平文または暗号化されたテキスト   | 平文                           |
      | `mysql_native_password BY`   | 平文                         | 平文                           |
      | `mysql_native_password AS`   | 暗号化されたテキスト           | 平文                           |
      | `authentication_ldap_simple` | 平文                         | 平文                           |

    > 注：すべての認証方法で、StarRocks はユーザーのパスワードを暗号化して保存します。

- `DEFAULT ROLE`

    ```SQL
    -- 挙げられたロールをユーザーのデフォルトのアクティブロールとして設定します。
    DEFAULT ROLE <role_name>[, <role_name>, ...]
    -- ユーザーが持っているすべてのロール（将来ユーザーに与えられるロールを含む）をデフォルトのアクティブロールとして設定します。
    DEFAULT ROLE ALL
    -- ユーザーのデフォルトロールをクリアします。注意：public ロールは自動的にアクティブになります。
    DEFAULT ROLE NONE
    ```

    ALTER コマンドでユーザーのデフォルトロールを変更する前に、対応するロールがユーザーに与えられていることを確認してください。設定後、ユーザーが再度ログインすると、対応するロールがデフォルトでアクティブになります。

## 例

例1：平文を使用してユーザーパスワードを変更します。

```SQL
ALTER USER 'jack' IDENTIFIED BY '123456';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password BY '123456';
```

例2：暗号化されたテキストを使用してユーザーパスワードを変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 暗号化されたパスワードは PASSWORD() 関数を使用して取得できます。

例3：ユーザーを LDAP 認証に変更します。

```SQL
ALTER USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例4：ユーザーを LDAP 認証に変更し、LDAP 内のユーザーの DN（Distinguished Name）を指定します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例5：ユーザーのデフォルトアクティブロールを `db_admin` と `user_admin` に変更します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

> 注意：ユーザーは `db_admin` と `user_admin` のロールを既に持っている必要があります。

例6：ユーザーのデフォルトアクティブロールをすべてのロールに変更します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE ALL;
```

> 注意：将来ユーザーに与えられるロールもデフォルトでアクティブになります。

例7：ユーザーのデフォルトアクティブロールを空に変更します。

```SQL
ALTER USER 'jack'@'192.168.%' DEFAULT ROLE NONE;
```

> 注意：ユーザーは `public` ロールをデフォルトでアクティブにします。

## 関連文書

- [CREATE USER](CREATE_USER.md)
- [SHOW USERS](SHOW_USERS.md)
- [DROP USER](DROP_USER.md)
