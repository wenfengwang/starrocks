---
displayed_sidebar: English
---

# JDBC 目录

StarRocks 从 v3.0 版本开始支持 JDBC 目录。

JDBC 目录是一种外部目录，允许您通过 JDBC 访问数据源并进行查询，无需数据摄入。

此外，您还可以通过基于[JDBC 目录](../../sql-reference/sql-statements/data-manipulation/INSERT.md)的 INSERT INTO 语句直接转换并加载来自 JDBC 数据源的数据。

目前，JDBC 目录支持 MySQL 和 PostgreSQL 数据库。

## 先决条件

- 您的 StarRocks 集群中的 FE 和 BE 能够通过 driver_url 参数指定的下载 URL 获取 JDBC 驱动程序。
- 在每个 BE 节点的 **$BE_HOME/bin/start_be.sh** 文件中，`JAVA_HOME` 需要被正确设置为 JDK 环境的路径，而不是 JRE 环境的路径。例如，您可以设置 `export JAVA_HOME = \\u003cJDK_absolute_path\\u003e`。您必须在脚本开始处添加此配置，并重启 BE 以使配置生效。

## 创建 JDBC 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数

#### catalog_name

JDBC 目录的名称。命名规则如下：

- 名称可以包含字母、数字 (0-9) 和下划线 (_)，且必须以字母开头。
- 名称区分大小写，长度不得超过 1023 个字符。

#### comment

JDBC 目录的描述，此参数是可选的。

#### PROPERTIES

JDBC 目录的属性。PROPERTIES 必须包含以下参数：

|参数|说明|
|---|---|
|类型|资源的类型。将值设置为 jdbc。|
|user|用于连接目标数据库的用户名。|
|密码|用于连接到目标数据库的密码。|
|jdbc_uri|JDBC 驱动程序用于连接到目标数据库的 URI。对于 MySQL，URI 的格式为“jdbc:mysql://ip:port”。对于 PostgreSQL，URI 的格式为“jdbc:postgresql://ip:port/db_name”。欲了解更多信息：PostgreSQL。|
|driver_url|JDBC驱动jar包的下载地址。支持 HTTP URL 或文件 URL，例如 https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar 和 file:///home/disk1 /postgresql-42.3.3.jar.NOTE您还可以将 JDBC 驱动程序放置到 FE 和 BE 节点上的任何相同路径，并将 driver_url 设置为该路径，该路径必须位于文件:///<path>/to/the /驱动程序格式。|
|driver_class|JDBC 驱动程序的类名。常见数据库引擎的 JDBC 驱动程序类名如下： MySQL：com.mysql.jdbc.Driver（MySQL v5.x 及更早版本）和 com.mysql.cj.jdbc.Driver（MySQL v6.x 及更高版本）PostgreSQL： org.postgresql.Driver|

> **注意**
> FE 在创建JDBC目录时会下载JDBC驱动的JAR包，而BE在首次查询时会下载。下载时间会根据网络条件而有所不同。

### 示例

以下示例创建了两个 JDBC 目录：jdbc0 和 jdbc1。

```SQL
CREATE EXTERNAL CATALOG jdbc0
PROPERTIES
(
    "type"="jdbc",
    "user"="postgres",
    "password"="changeme",
    "jdbc_uri"="jdbc:postgresql://127.0.0.1:5432/jdbc_test",
    "driver_url"="https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar",
    "driver_class"="org.postgresql.Driver"
);

CREATE EXTERNAL CATALOG jdbc1
PROPERTIES
(
    "type"="jdbc",
    "user"="root",
    "password"="changeme",
    "jdbc_uri"="jdbc:mysql://127.0.0.1:3306",
    "driver_url"="https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.28/mysql-connector-java-8.0.28.jar",
    "driver_class"="com.mysql.cj.jdbc.Driver"
);
```

## 查看 JDBC 目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 命令查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 命令查询外部目录的创建语句。以下示例查询名为 `jdbc0` 的 JDBC 目录的创建语句：

```SQL
SHOW CREATE CATALOG jdbc0;
```

## 删除 JDBC 目录

您可以使用[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)命令删除JDBC目录。

以下示例删除名为 jdbc0 的 JDBC 目录：

```SQL
DROP Catalog jdbc0;
```

## 查询 JDBC 目录中的表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 命令查看数据库在您的 JDBC 兼容集群中：

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 命令在当前会话中切换到目标目录：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   然后，使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 命令指定当前会话中的活跃数据库：

   ```SQL
   USE <db_name>;
   ```

   或者，您可以使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)命令直接指定目标目录中的活跃数据库：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 命令查询指定数据库中的目标表：

   ```SQL
   SELECT * FROM <table_name>;
   ```

## 常见问题解答

如果出现“数据库 URL 格式错误，无法解析主 URL 部分”的错误，我该怎么办？

如果您遇到这类错误，说明您在 `jdbc_uri` 中传递的 URI 无效。请检查您传递的 URI，并确保其有效性。更多详细信息，请参见本主题"[PROPERTIES](#properties)"部分中的参数描述。
