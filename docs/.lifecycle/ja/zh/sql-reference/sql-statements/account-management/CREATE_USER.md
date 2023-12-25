---
displayed_sidebar: Chinese
---

# ユーザーの作成

## 機能

StarRocksのユーザーを作成します。

> **注意**
>
> `user_admin` ロールを持つユーザーのみがこの操作を実行できます。

## 文法

```SQL
CREATE USER <user_identity> [auth_option] [DEFAULT ROLE <role_name>[, <role_name>, ...]]
```

## パラメータ説明

- `user_identity`：ユーザー識別子。ログインIP（userhost）とユーザー名（username）で構成され、`username@'userhost'` の形式で書きます。`userhost` は `%` を使用して曖昧なマッチングを行うことができます。`userhost` を指定しない場合、デフォルトは `%` となり、任意のホストから `username` を使用して StarRocks に接続できるユーザーが作成されます。

- `auth_option`：ユーザーの認証方法。現在、StarRocksはネイティブパスワード、mysql_native_password、LDAPの3種類の認証方法をサポートしています。ネイティブパスワードと mysql_native_password の認証方法は内部ロジックが同じで、具体的な設定構文にわずかな違いがあります。同一の user identity には1種類の認証方法のみ使用できます。

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
      | ネイティブパスワード           | 平文または暗号化されたパスワード | 平文                         |
      | `mysql_native_password BY`   | 平文                         | 平文                         |
      | `mysql_native_password AS`   | 暗号化されたパスワード       | 平文                         |
      | `authentication_ldap_simple` | 平文                         | 平文                         |

    > 注：すべての認証方法において、StarRocksはユーザーのパスワードを暗号化して保存します。

- `DEFAULT ROLE <role_name>[, <role_name>, ...]`：このパラメータを指定すると、指定されたロールがユーザーに自動的に割り当てられ、ユーザーがログインした後にデフォルトでアクティブになります。指定しない場合、ユーザーにはデフォルトで権限がありません。指定するロールは既に存在している必要があります。

## 例

例1：平文パスワードを使用してユーザーを作成します（ホストを指定しない場合は jack@'%' と同じです）。

```SQL
CREATE USER jack IDENTIFIED BY '123456';
CREATE USER jack IDENTIFIED WITH mysql_native_password BY '123456';
```

例2：暗号化されたパスワードを使用してユーザーを作成し、そのユーザーが '172.10.1.10' からログインできるようにします。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH mysql_native_password AS '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
```

> 暗号化されたパスワードはPASSWORD()関数を使用して取得できます。

例3：ドメイン 'example_domain' からログインできるユーザーを作成します。

```SQL
CREATE USER 'jack'@['example_domain'] IDENTIFIED BY '123456';
```

例4：LDAP認証を使用するユーザーを作成します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple;
```

例5：LDAP認証を使用するユーザーを作成し、LDAP内のDN（Distinguished Name）を指定します。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED WITH authentication_ldap_simple AS 'uid=jack,ou=company,dc=example,dc=com';
```

例6：'192.168' サブネットからログインできるユーザーを作成し、デフォルトロールとして `db_admin` と `user_admin` を指定します。

```SQL
CREATE USER 'jack'@'192.168.%' DEFAULT ROLE db_admin, user_admin;
```

## 関連文書

ユーザーを作成した後、ユーザーに権限やロールを付与したり、ユーザー情報を変更したり、ユーザーを削除したりすることができます。

- [ALTER USER](ALTER_USER.md)
- [GRANT](GRANT.md)
- [DROP USER](DROP_USER.md)
