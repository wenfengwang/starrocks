---
displayed_sidebar: Chinese
---

# ユーザー認証の設定

この文書では、StarRocksでユーザー認証（authentication）を設定する方法について説明します。

## LDAP認証の設定

従来のユーザー名とパスワードによる認証方法に加えて、StarRocksはLightweight Directory Access Protocol（LDAP）認証もサポートしています。

### LDAP認証を有効にする

FEノードの設定ファイル **fe.conf** に以下の設定項目を追加します。

```conf
# LDAPサーバーのIPアドレスを追加します。
authentication_ldap_simple_server_host =
# LDAPサーバーのポートを追加します。
authentication_ldap_simple_server_port =
```

StarRocksを使用してLDAPシステム内のユーザーを直接検索してログインユーザーを認証する場合は、**さらに以下の設定項目を追加**する必要があります。

```conf
# ユーザーのBase DNを追加し、ユーザーの検索範囲を指定します。
authentication_ldap_simple_bind_base_dn =
# LDAPオブジェクト内でユーザーを識別する属性名を追加します。デフォルトはuidです。
authentication_ldap_simple_user_search_attr =
# ユーザーを検索する際に使用する管理者アカウントのDNを追加します。
authentication_ldap_simple_bind_root_dn =
# ユーザーを検索する際に使用する管理者アカウントのパスワードを追加します。
authentication_ldap_simple_bind_root_pwd =
```

### ユーザーの作成

上記の設定を完了した後、StarRocksに対応するユーザーを作成し、その認証方法と認証情報を指定する必要があります。

```sql
CREATE USER user_identity IDENTIFIED WITH authentication_ldap_simple [AS 'ldap_distinguished_name'];
```

以下の例では、LDAP Distinguished Name（DN）が "uid=zhansan,ou=company,dc=example,dc=com" であるLDAP認証ユーザーzhangsanを作成します。

```sql
CREATE USER zhangsan IDENTIFIED WITH authentication_ldap_simple AS 'uid=zhansan,ou=company,dc=example,dc=com'
```

StarRocksを使用してLDAPシステム内のユーザーを直接検索してログインユーザーを認証する場合は、上記の**追加設定を完了した後**、ユーザーを作成する際にLDAP内のDNを指定する必要はありません。そのユーザーがログインする際に、StarRocksはLDAPシステム内でそのユーザーを検索し、一致する結果が一つだけあれば、認証は成功します。

### ユーザーの認証

LDAP認証を使用する際は、クライアントからStarRocksに平文パスワードを渡す必要があります。

典型的なクライアント設定で平文パスワードを渡す方法は以下の三つです。

* **MySQLクライアント**

```shell
mysql -u<user_identity> -P<query_port> -h<fe_host> -p --default-auth=mysql_clear_password --enable-cleartext-plugin
```

例：

```shell
mysql -uzhangsan -P9030 -h127.0.0.1 -p --default-auth=mysql_clear_password --enable-cleartext-plugin
```

* **JDBC**

JDBCのデフォルトのMysqlClearPasswordPluginはSSLを使用してデータを転送する必要があるため、カスタムプラグインを作成する必要があります。

```java
public class MysqlClearPasswordPluginWithoutSSL extends MysqlClearPasswordPlugin {
    @Override  
    public boolean requiresConfidentiality() {
        return false;
    }
}
```

接続を取得する際に、カスタムプラグインをプロパティに設定する必要があります。

```java
...
Properties properties = new Properties();// パッケージ名をxxx.xxx.xxxに置き換えてください
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
```

* **ODBC**

ODBCのDSNに以下の設定を追加し、ユーザー名とパスワードを設定する必要があります。

```conf
default_auth = mysql_clear_password
ENABLE_CLEARTEXT_PLUGIN = 1
```
