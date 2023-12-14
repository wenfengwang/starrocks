---
displayed_sidebar: "English"
---

# 身份验证方法

除了“用户名+密码”身份验证方法外，StarRocks还支持LDAP。

## LDAP认证

要使用LDAP身份验证，您需要首先将LDAP服务添加到FE节点配置中。

* `authentication_ldap_simple_server_host`: 指定服务IP。
* `authentication_ldap_simple_server_port`: 指定服务端口，默认值为389。

在创建用户时，通过 `IDENTIFIED WITH authentication_ldap_simple AS 'xxx'` 指定身份验证方法为LDAP认证。xxx是LDAP中用户的DN（Distinguished Name）。

示例1：

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple AS 'uid=tom,ou=company,dc=example,dc=com'
~~~

也可以创建用户而不指定LDAP中用户的DN。当用户登录时，StarRocks将访问LDAP系统以检索用户信息。如果有且仅有一个匹配项，则认证成功。

示例2：

~~~sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple
~~~

在这种情况下，还需要在FE中添加额外的配置

* `authentication_ldap_simple_bind_base_dn`: 用户的基本DN，指定用户的检索范围。
* `authentication_ldap_simple_user_search_attr`: LDAP对象中标识用户的属性名称，默认为uid。
* `authentication_ldap_simple_bind_root_dn`: 用于检索用户信息的管理员帐户的DN。
* `authentication_ldap_simple_bind_root_pwd`: 用于检索用户信息时使用的管理员帐户的密码。

LDAP认证要求客户端将明文密码传递给StarRocks。有三种方式可以传递明文密码：

* **MySQL命令行**

在执行时添加 `--default-auth mysql_clear_password --enable-cleartext-plugin`：

~~~sql
mysql -utom -P8030 -h127.0.0.1 -p --default-auth mysql_clear_password --enable-cleartext-plugin
~~~

* **JDBC**

由于JDBC的默认MysqlClearPasswordPlugin需要SSL传输，因此需要自定义插件。

~~~java
public class MysqlClearPasswordPluginWithoutSSL extends MysqlClearPasswordPlugin {
    @Override  
    public boolean requiresConfidentiality() {
        return false;
    }
}
~~~

连接后，将自定义插件配置到属性中。

~~~java
...
Properties properties = new Properties();// 将xxx.xxx.xxx替换为您的包名
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
~~~

* **ODBC**

在ODBC的DSN中添加 `default\_auth=mysql_clear_password` 和 `ENABLE_CLEARTEXT\_PLUGIN=1`，以及用户名和密码。