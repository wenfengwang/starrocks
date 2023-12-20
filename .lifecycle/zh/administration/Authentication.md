---
displayed_sidebar: English
---

# 认证方法

除了“用户名+密码”的认证方法之外，StarRocks还支持LDAP认证。

## LDAP认证

要启用LDAP认证，您首先需要将LDAP服务配置到FE节点中。

* authentication_ldap_simple_server_host：指定服务的IP地址。
* authentication_ldap_simple_server_port：指定服务的端口号，默认为389。

在创建用户时，可以通过设置IDENTIFIED WITH authentication_ldap_simple AS 'xxx'来指定使用LDAP认证方法。其中的xxx代表LDAP中该用户的DN（Distinguished Name，识别名称）。

示例1：

```sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple AS 'uid=tom,ou=company,dc=example,dc=com'
```

可以在不指定LDAP中用户DN的情况下创建用户。用户登录时，StarRocks会查询LDAP系统来获取用户信息，如果唯一匹配，则认证成功。

示例2：

```sql
CREATE USER tom IDENTIFIED WITH authentication_ldap_simple
```

在这种情况下，需要在FE中额外添加配置：

* authentication_ldap_simple_bind_base_dn：用户的基本DN，用于指定检索用户信息的范围。
* authentication_ldap_simple_user_search_attr：在LDAP对象中用于识别用户的属性名称，默认为uid。
* authentication_ldap_simple_bind_root_dn：用于检索用户信息的管理员账号的DN。
* authentication_ldap_simple_bind_root_pwd：用于检索用户信息时管理员账号的密码。

LDAP认证要求客户端向StarRocks传送明文密码。传送明文密码有三种方法：

* **MySQL命令行**

执行时，添加参数 --default-auth mysql_clear_password --enable-cleartext-plugin：

```sql
mysql -utom -P8030 -h127.0.0.1 -p --default-auth mysql_clear_password --enable-cleartext-plugin
```

* **JDBC**

由于JDBC的默认MysqlClearPasswordPlugin需要SSL加密传输，需要使用自定义插件。

```java
public class MysqlClearPasswordPluginWithoutSSL extends MysqlClearPasswordPlugin {
    @Override  
    public boolean requiresConfidentiality() {
        return false;
    }
}
```

连接成功后，将自定义插件配置到属性中。

```java
...
Properties properties = new Properties();// replace xxx.xxx.xxx to your pacakage name
properties.put("authenticationPlugins", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("defaultAuthenticationPlugin", "xxx.xxx.xxx.MysqlClearPasswordPluginWithoutSSL");
properties.put("disabledAuthenticationPlugins", "com.mysql.jdbc.authentication.MysqlNativePasswordPlugin");DriverManager.getConnection(url, properties);
```

* **ODBC**

在ODBC的DSN设置中添加default_auth=mysql_clear_password和ENABLE_CLEARTEXT_PLUGIN=1，同时包括用户名和密码。
