---
displayed_sidebar: "Japanese"
---

# 認証方法

"ユーザ名+パスワード"の認証方法に加えて、StarRocksはLDAPもサポートしています。

## LDAP 認証

LDAP認証を使用するには、まずLDAPサービスをFEノードの構成に追加する必要があります。

* `authentication_ldap_simple_server_host`: サービスのIPを指定します。
* `authentication_ldap_simple_server_port`: デフォルト値は389で、サービスのポートを指定します。

ユーザを作成する際に、`IDENTIFIED WITH authentication_ldap_simple AS 'xxx'`として認証方法をLDAP認証に指定します。xxxはLDAP内のユーザのDN（識別名）です。

例 1:

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple AS 'uid=tom,ou=company,dc=example,dc=com'
~~~

LDAP内でユーザのDNを指定せずにユーザを作成することも可能です。ユーザがログインすると、StarRocksはLDAPシステムにユーザ情報を取得しに行きます。一致する情報が一つだけあれば、認証は成功します。

例 2:

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple
~~~

この場合、FEに追加の構成が必要です

* `authentication_ldap_simple_bind_base_dn`: ユーザのベースDNで、ユーザの取得範囲を指定します。
* `authentication_ldap_simple_user_search_attr`: ユーザを識別するLDAPオブジェクト内の属性の名前で、デフォルトはuidです。
* `authentication_ldap_simple_bind_root_dn`: ユーザ情報を取得する際に使用する管理者アカウントのDNです。
* `authentication_ldap_simple_bind_root_pwd`: ユーザ情報を取得する際に使用する管理者アカウントのパスワードです。

LDAP認証では、クライアントがStarRocksにクリアテキストのパスワードを渡す必要があります。クリアテキストのパスワードを渡す方法は3つあります。

* **MySQLコマンドライン**

実行時に `--default-auth mysql_clear_password --enable-cleartext-plugin` を追加します:

~~~sql
mysql -utom -P8030 -h127.0.0.1 -p --default-auth mysql_clear_password --enable-cleartext-plugin
~~~

* **JDBC**

JDBCのデフォルトMysqlClearPasswordPluginはSSL転送を必要とするため、カスタムプラグインが必要です。

~~~java
public class MysqlClearPasswordPluginWithoutSSL extends MysqlClearPasswordPlugin {
    @Override  
    public boolean requiresConfidentiality() {
        return false;
    }
}
~~~

接続後、カスタムプラグインをプロパティに設定します。

~~~java
...
Properties properties = new Properties();// xxx.xxx.xxxはパッケージ名に置き換えてください
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
~~~

* **ODBC**

ODBCのDSNに `default\_auth=mysql_clear_password` と `ENABLE_CLEARTEXT\_PLUGIN=1` を追加し、ユーザ名とパスワードと一緒に入力します。