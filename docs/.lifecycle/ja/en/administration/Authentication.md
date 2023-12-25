---
displayed_sidebar: English
---

# 認証方法

StarRocksは「ユーザー名+パスワード」の認証方法に加えて、LDAPもサポートしています。

## LDAP認証

LDAP認証を使用するには、まずFEノードの設定にLDAPサービスを追加する必要があります。

* `authentication_ldap_simple_server_host`: サービスのIPを指定します。
* `authentication_ldap_simple_server_port`: サービスポートを指定します。デフォルト値は389です。

ユーザーを作成する際には、`IDENTIFIED WITH authentication_ldap_simple AS 'xxx'`を使用してLDAP認証を指定します。xxxはLDAP内のユーザーのDN（識別名）です。

例1:

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple AS 'uid=tom,ou=company,dc=example,dc=com'
~~~

LDAP内でユーザーのDNを指定せずにユーザーを作成することも可能です。ユーザーがログインする際、StarRocksはLDAPシステムに問い合わせてユーザー情報を取得します。一致するものが1つだけの場合、認証は成功します。

例2:

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple
~~~

この場合、追加の設定をFEに追加する必要があります。

* `authentication_ldap_simple_bind_base_dn`: ユーザーの検索範囲を指定するベースDN。
* `authentication_ldap_simple_user_search_attr`: ユーザーを識別するLDAPオブジェクトの属性名。デフォルトはuidです。
* `authentication_ldap_simple_bind_root_dn`: ユーザー情報を取得するために使用される管理者アカウントのDN。
* `authentication_ldap_simple_bind_root_pwd`: ユーザー情報を取得する際に使用される管理者アカウントのパスワード。

LDAP認証では、クライアントは平文のパスワードをStarRocksに渡す必要があります。平文のパスワードを渡す方法は3つあります。

* **MySQLコマンドライン**

 実行時に`--default-auth=mysql_clear_password --enable-cleartext-plugin`を追加します。

~~~sql
mysql -utom -P8030 -h127.0.0.1 -p --default-auth=mysql_clear_password --enable-cleartext-plugin
~~~

* **JDBC**

JDBCのデフォルトのMysqlClearPasswordPluginはSSLトランスポートを必要としますが、カスタムプラグインを使用することもできます。

~~~java
public class MysqlClearPasswordPluginWithoutSSL extends MysqlClearPasswordPlugin {
    @Override  
    public boolean requiresConfidentiality() {
        return false;
    }
}
~~~

接続後、プロパティにカスタムプラグインを設定します。

~~~java
...
Properties properties = new Properties();// replace xxx.xxx.xxx to your package name
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
~~~

* **ODBC**

ODBCのDSNに`default_auth=mysql_clear_password`と`ENABLE_CLEARTEXT_PLUGIN=1`を追加します。ユーザー名とパスワードも必要です。
