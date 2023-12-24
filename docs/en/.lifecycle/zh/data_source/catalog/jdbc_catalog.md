---
displayed_sidebar: English
---

# JDBC 目录

StarRocks 从 v3.0 开始支持 JDBC 目录。

JDBC 目录是一种外部目录，它使您能够通过 JDBC 访问的数据源查询数据，而无需进行数据摄入。

此外，您还可以使用基于 JDBC 目录的 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 直接从 JDBC 数据源转换和加载数据。

JDBC 目录目前支持 MySQL 和 PostgreSQL。

## 先决条件

- 您的 StarRocks 集群中的 FE 和 BE 可以从 `driver_url` 参数指定的下载 URL 下载 JDBC 驱动。
- 每个 BE 节点上 **$BE_HOME/bin/start_be.sh** 文件中的 `JAVA_HOME` 必须正确配置为 JDK 环境的路径，而不是 JRE 环境的路径。例如，您可以配置 `export JAVA_HOME = <JDK_absolute_path>`。您必须在脚本的开头添加此配置，并重新启动 BE 以使配置生效。

## 创建 JDBC 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数

#### `catalog_name`

JDBC 目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### `comment`

JDBC 目录的描述。此参数是可选的。

#### `PROPERTIES`

JDBC 目录的属性。`PROPERTIES` 必须包含以下参数：

| **参数**     | **描述**                                                     |
| ----------------- | ------------------------------------------------------------ |
| type              | 资源的类型。将值设置为 `jdbc`。           |
| user              | 用于连接到目标数据库的用户名。 |
| password          | 用于连接到目标数据库的密码。 |
| jdbc_uri          | JDBC 驱动程序用于连接到目标数据库的 URI。对于 MySQL，URI 的格式为 `"jdbc:mysql://ip:port"`。对于 PostgreSQL，URI 的格式为 `"jdbc:postgresql://ip:port/db_name"`。有关更多信息，请访问： [PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)。 |
| driver_url        | JDBC 驱动程序 JAR 包的下载 URL。例如，支持 HTTP URL 或文件 URL，例如 `https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar` 和 `file:///home/disk1/postgresql-42.3.3.jar`。<br />**注意**<br />您还可以将 JDBC 驱动程序放在 FE 和 BE 节点上的相同路径，并将 `driver_url` 设置为该路径，该路径必须采用 `file:///<path>/to/the/driver` 格式。 |
| driver_class      | JDBC 驱动程序的类名。常见数据库引擎的 JDBC 驱动类名如下：<ul><li>MySQL：`com.mysql.jdbc.Driver`（MySQL v5.x 及更早版本）和 `com.mysql.cj.jdbc.Driver`（MySQL v6.x 及更高版本）</li><li>PostgreSQL：`org.postgresql.Driver`</li></ul> |

> **注意**
>
> FE 在创建 JDBC 目录时会下载 JDBC 驱动 JAR 包，BE 在首次查询时会下载 JDBC 驱动 JAR 包。下载所需的时间因网络条件而异。

### 例子

以下示例创建两个 JDBC 目录：`jdbc0` 和 `jdbc1`。

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

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录的创建语句。以下示例查询名为 `jdbc0` 的 JDBC 目录的创建语句：

```SQL
SHOW CREATE CATALOG jdbc0;
```

## 删除 JDBC 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除 JDBC 目录。

以下示例删除名为 `jdbc0` 的 JDBC 目录：

```SQL
DROP Catalog jdbc0;
```

## 查询 JDBC 目录中的表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看您的 JDBC 兼容集群中的数据库：

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 切换到当前会话中的目标目录：

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    然后，使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定当前会话中的活动数据库：

    ```SQL
    USE <db_name>;
    ```

    或者，您可以使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 直接指定目标目录中的活动数据库：

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询指定数据库中的目标表：

   ```SQL
   SELECT * FROM <table_name>;
   ```

## 常见问题

如果出现“格式错误的数据库 URL，无法解析主 URL 部分”的错误提示，我该怎么办？

如果遇到此类错误，说明您传递给 `jdbc_uri` 的 URI 无效。请检查传递的 URI 并确保其有效。有关详细信息，请参阅本主题的“[属性](#properties)”部分中的参数说明。
