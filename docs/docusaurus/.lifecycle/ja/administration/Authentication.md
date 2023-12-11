---
displayed_sidebar: "English"
---

# 認証方法

"username+password"認証方法に加えて、StarRocksはLDAPもサポートしています。

## LDAP 認証

LDAP認証を使用するには、まずLDAPサービスをFEノードの構成に追加する必要があります。

* `authentication_ldap_simple_server_host`: サービスIPを指定します。
* `authentication_ldap_simple_server_port`: サービスポートを指定します。デフォルト値は389です。

ユーザを作成する際は、`IDENTIFIED WITH authentication_ldap_simple AS 'xxx'`によって認証方法をLDAP認証として指定します。xxxはLDAP中のユーザのDN（Distinguished Name）です。

例1:

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple AS 'uid=tom,ou=company,dc=example,dc=com'
~~~

ユーザのDNを特定せずにユーザを作成することも可能です。ユーザがログインすると、StarRocksはLDAPシステムにユーザ情報を取得しに行きます。一致する情報が一つだけあれば、認証は成功します。

例2:

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple
~~~

この場合、FEに追加の構成が必要になります。

* `authentication_ldap_simple_bind_base_dn`: ユーザのベースDNであり、ユーザの取得範囲を指定します。
* `authentication_ldap_simple_user_search_attr`: ユーザを識別するLDAPオブジェクト内の属性名。デフォルト値はuidです。
* `authentication_ldap_simple_bind_root_dn`: ユーザ情報を取得する際に使用される管理者アカウントのDN。
* `authentication_ldap_simple_bind_root_pwd`: ユーザ情報を取得する際に使用される管理者アカウントのパスワード。

LDAP認証では、クライアントが明示的なパスワードをStarRocksに渡す必要があります。クライアアントが明示的なパスワードを渡す方法は3つあります。

* **MySQLコマンドライン**

実行時に `--default-auth mysql_clear_password --enable-cleartext-plugin`を追加します。

~~~sql
mysql -utom -P8030 -h127.0.0.1 -p --default-auth mysql_clear_password --enable-cleartext-plugin
~~~

* **JDBC**

JDBCのデフォルトのMysqlClearPasswordPluginはSSLトランスポートを必要とするため、カスタムプラグインが必要です。

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
Properties properties = new Properties();// xxx.xxx.xxxをパッケージ名に置き換えてください
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
~~~

* **ODBC**

ODBCのDSNに `default\_auth=mysql_clear_password` と `ENABLE_CLEARTEXT\_PLUGIN=1`を追加し、ユーザ名とパスワードと共に設定します。