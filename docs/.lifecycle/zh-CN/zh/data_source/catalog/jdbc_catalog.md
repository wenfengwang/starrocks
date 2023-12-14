---
displayed_sidebar: "Chinese"
---

# JDBC目录

StarRocks 从 3.0 版本开始支持 JDBC目录。

JDBC目录是一种外部目录。通过JDBC目录，您可以直接查询JDBC数据源中的数据，而无需执行数据导入。

此外，您还可以基于JDBC目录，结合[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)的功能，对JDBC数据源的数据进行转换和导入。

目前，JDBC目录支持MySQL和PostgreSQL。

## 前提条件

- 确保FE和BE可以通过`driver_url`指定的下载路径，下载所需的JDBC驱动程序。
- BE所在机器的启动脚本**$BE_HOME/bin/start_be.sh**中需要配置`JAVA_HOME`，要配置成JDK环境，不能配置成JRE环境，比如`export JAVA_HOME = <JDK的绝对路径>`。注意需要将该配置添加在BE启动脚本最开头，添加完成后需重启BE。

## 创建JDBC目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### 参数说明

#### `catalog_name`

JDBC目录的名称。命名要求如下：

- 必须由字母（a-z或A-Z）、数字（0-9）或下划线（_）组成，且只能以字母开头。
- 总长度不能超过1023个字符。
- Catalog名称大小写敏感。

#### `comment`

JDBC目录的描述。此参数为可选。

#### PROPERTIES

JDBC目录的属性，包含如下必填配置项：

| **参数**     | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| type         | 资源类型，固定取值为`jdbc`。                                |
| user         | 目标数据库登录用户名。                                       |
| password     | 目标数据库用户登录密码。                                     |
| jdbc_uri     | JDBC驱动程序连接目标数据库的URI。如果使用MySQL，格式为：`"jdbc:mysql://ip:port"`。如果使用PostgreSQL，格式为`"jdbc:postgresql://ip:port/db_name"`。 |
| driver_url   | 用于下载JDBC驱动程序JAR包的URL。支持使用HTTP协议或者file协议，例如`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`和`file:///home/disk1/postgresql-42.3.3.jar`。<br />**说明**<br />您也可以把JDBC驱动程序部署在FE或BE所在节点上任意相同路径下，然后把`driver_url`设置为该路径，格式为`file:///<path>/to/the/driver`。 |
| driver_class | JDBC驱动程序的类名称。以下是常见数据库引擎支持的JDBC驱动程序类名称：<ul><li>MySQL：`com.mysql.jdbc.Driver`（MySQL5.x及之前版本）、`com.mysql.cj.jdbc.Driver`（MySQL6.x及之后版本）</li><li>PostgreSQL: `org.postgresql.Driver`</li></ul> |

> **说明**
>
> FE会在创建JDBC目录时去获取JDBC驱动程序，BE会在第一次执行查询时去获取驱动程序。获取驱动程序的耗时跟网络条件相关。

### 创建示例

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

您可以通过[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)查询当前所在StarRocks集群里所有目录：

```SQL
SHOW CATALOGS;
```

您也可以通过[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)查询某个外部目录的创建语句。例如，通过如下命令查询JDBC目录`jdbc0`的创建语句：

```SQL
SHOW CREATE CATALOG jdbc0;
```

## 删除JDBC目录

您可以通过[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)删除一个JDBC目录。

例如，通过如下命令删除JDBC目录`jdbc0`：

```SQL
DROP Catalog jdbc0;
```

## 查询JDBC目录中的表数据

1. 通过[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)查看指定目录所属的集群中的数据库：

   ```SQL
   SHOW DATABASES from <catalog_name>;
   ```

2. 通过[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)切换当前会话生效的目录：

    ```SQL
    SET CATALOG <catalog_name>;
    ```

    再通过[USE](../../sql-reference/sql-statements/data-definition/USE.md)指定当前会话生效的数据库：

    ```SQL
    USE <db_name>;
    ```

    或者，也可以通过[USE](../../sql-reference/sql-statements/data-definition/USE.md)直接将会话切换到目标目录下的指定数据库：

    ```SQL
    USE <catalog_name>.<db_name>;
    ```

3. 通过[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)查询目标数据库中的目标表：

   ```SQL
   SELECT * FROM <table_name>;
   ```

## 常见问题

系统返回"Malformed database URL, failed to parse the main URL sections"报错应该如何处理？

该报错通常是由于`jdbc_uri`中传入的URI有误而引起的。请检查并确保传入的URI是否正确无误。参见本文“[PROPERTIES](#properties)”小节相关的参数说明。