---
displayed_sidebar: English
---

# 身份验证方法

除了“用户名+密码”的认证方式外，StarRocks 还支持 LDAP。

## LDAP 身份验证

要使用 LDAP 认证，您需要先将 LDAP 服务添加到 FE 节点配置中。

* `authentication_ldap_simple_server_host`：指定服务IP。
* `authentication_ldap_simple_server_port`：指定服务端口，默认值为 389。

创建用户时，请通过 `IDENTIFIED WITH authentication_ldap_simple AS 'xxx'` 将认证方法指定为 LDAP 认证。xxx 是 LDAP 中用户的 DN（可分辨名称）。

示例 1：

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple AS 'uid=tom,ou=company,dc=example,dc=com'
~~~

可以在不指定用户 LDAP 中的 DN 的情况下创建用户。当用户登录时，StarRocks 会进入 LDAP 系统获取用户信息。如果存在且只有一个匹配项，则身份验证成功。

示例 2：

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple
~~~

在这种情况下，需要向 FE 添加额外的配置

* `authentication_ldap_simple_bind_base_dn`：用户的基本DN，指定用户的检索范围。
* `authentication_ldap_simple_user_search_attr`：LDAP 对象中标识用户的属性的名称，默认为 uid。
* `authentication_ldap_simple_bind_root_dn`：用于检索用户信息的管理员帐户的 DN。
* `authentication_ldap_simple_bind_root_pwd`：检索用户信息时使用的管理员帐户的密码。

LDAP 认证需要客户端向 StarRocks 传递明文密码。有三种方法可以传递明文密码：

* **MySQL命令行**

执行时添加 `--default-auth=mysql_clear_password --enable-cleartext-plugin`：

~~~sql
mysql -utom -P8030 -h127.0.0.1 -p --default-auth=mysql_clear_password --enable-cleartext-plugin
~~~

* **JDBC**

由于 JDBC 的默认 MysqlClearPasswordPlugin 需要 SSL 传输，因此需要自定义插件。

~~~java
public class MysqlClearPasswordPluginWithoutSSL extends MysqlClearPasswordPlugin {
    @Override  
    public boolean requiresConfidentiality() {
        return false;
    }
}
~~~

连接后，在属性中配置自定义插件。

~~~java
...
Properties properties = new Properties();// 将 xxx.xxx.xxx 替换为您的包名称
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
~~~

* **ODBC**

在 ODBC 的 DSN 中添加 `default_auth=mysql_clear_password` 和 `ENABLE_CLEARTEXT_PLUGIN=1`，以及用户名和密码。
