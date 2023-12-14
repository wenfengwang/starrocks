---
displayed_sidebar: "Chinese"
---

# JDBC目录

StarRocks从v3.0开始支持JDBC目录。

JDBC目录是一种外部目录，它使您能够通过JDBC访问数据源来查询数据，而无需进行数据摄入。

此外，您可以直接使用基于JDBC目录的[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)来转换和加载JDBC数据源的数据。

JDBC目录目前支持MySQL和PostgreSQL。

## 先决条件

- StarRocks集群中的FE和BE可以从`driver_url`参数指定的下载URL下载JDBC驱动程序。
- 每个BE节点上的**$BE_HOME/bin/start_be.sh**文件中的`JAVA_HOME`应该被适当地配置为JDK环境中的路径，而不是JRE环境中的路径。例如，您可以配置`export JAVA_HOME = <JDK_absolute_path>`。必须在脚本的开头添加此配置，并重新启动BE以使配置生效。

## 创建JDBC目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数

#### `catalog_name`

JDBC目录的名称。命名约定如下：

- 名称可包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，且长度不能超过1023个字符。

#### `comment`

JDBC目录的描述。此参数可选。

#### `PROPERTIES`

JDBC目录的属性。`PROPERTIES`必须包括以下参数：

| **参数**       | **描述**                                |
| -------------- | ---------------------------------------- |
| type           | 资源的类型。将值设置为`jdbc`。          |
| user           | 连接到目标数据库使用的用户名。           |
| password       | 连接到目标数据库使用的密码。             |
| jdbc_uri       | JDBC驱动程序用于连接到目标数据库的URI。对于MySQL，URI采用`"jdbc:mysql://ip:port"`格式。对于PostgreSQL，URI采用`"jdbc:postgresql://ip:port/db_name"`格式。更多信息，请参见：[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)。 |
| driver_url     | JDBC驱动程序JAR包的下载URL。支持HTTP URL或文件URL，例如，`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`和`file:///home/disk1/postgresql-42.3.3.jar`。<br />**注意**<br />您也可以将JDBC驱动程序放置到FE和BE节点上的任何相同路径，并将`driver_url`设置为该路径，该路径必须采用`file:///<path>/to/the/driver`格式。 |
| driver_class   | JDBC驱动程序的类名。常见数据库引擎的JDBC驱动程序类名如下：<ul><li>MySQL：`com.mysql.jdbc.Driver`（MySQL v5.x及更早版本）和`com.mysql.cj.jdbc.Driver`（MySQL v6.x及以后版本）</li><li>PostgreSQL：`org.postgresql.Driver`</li></ul> |

> **注意**
>
> FE在创建JDBC目录时下载JDBC驱动程序JAR包，BE在执行第一个查询时下载JDBC驱动程序JAR包。下载所需时间因网络状况而异。

### 示例

以下示例创建了两个JDBC目录：`jdbc0`和`jdbc1`。

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

## 查看JDBC目录

您可以使用[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)来查询当前StarRocks集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)来查询外部目录的创建语句。以下示例查询名为`jdbc0`的JDBC目录的创建语句：

```SQL
SHOW CREATE CATALOG jdbc0;
```

## 删除JDBC目录

您可以使用[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)来删除JDBC目录。

以下示例删除名为`jdbc0`的JDBC目录：

```SQL
DROP Catalog jdbc0;
```

## 查询JDBC目录中的表

1. 使用[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)来查看JDBC兼容集群中的数据库：

   ```SQL
   SHOW DATABASES <catalog_name>;
   ```

2. 使用[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)在当前会话中切换到目标目录：

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    然后，使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)来指定当前会话中的活动数据库：

    ```SQL
    USE <db_name>;
    ```

    或者，您可以直接使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)来指定目标目录中的活动数据库：

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. 使用[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)来查询指定数据库中的目标表：

   ```SQL
   SELECT * FROM <table_name>;
   ```

## 常见问题

如果出现“数据库URL格式错误，无法解析主URL部分”的错误提示，应该怎么办？

如果遇到此类错误，表示您传递给`jdbc_uri`的URI无效。请检查您传递的URI，并确保它是有效的。有关更多信息，请参见本主题的"[PROPERTIES](#properties)"部分中的参数描述。