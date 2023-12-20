---
displayed_sidebar: English
---

# Java UDFs

从 v2.2.0 版本开始，您可以使用 Java 编程语言编译用户定义函数（UDFs），以满足您的特定业务需求。

从 v3.0 版本开始，StarRocks 支持全局 UDFs，您只需在相关 SQL 语句（CREATE/SHOW/DROP）中包含 `GLOBAL` 关键字即可。

本主题介绍如何开发和使用各种 UDFs。

目前，StarRocks 支持标量 UDFs、用户定义聚合函数（UDAFs）、用户定义窗口函数（UDWFs）和用户定义表函数（UDTFs）。

## 先决条件

- 您已经安装了 [Apache Maven](https://maven.apache.org/download.cgi)，因此您可以创建并编译 Java 项目。

- 您已在服务器上安装了 JDK 1.8。

- Java UDF 功能已启用。您可以在 FE 配置文件 **fe/conf/fe.conf** 中将 FE 配置项 `enable_udf` 设置为 `true` 来启用此功能，并重启 FE 节点以使设置生效。更多信息请参见[参数配置](../../administration/FE_configuration.md)。

## 开发和使用 UDFs

您需要创建一个 Maven 项目，并使用 Java 编程语言编译您需要的 UDF。

### 第一步：创建 Maven 项目

创建一个 Maven 项目，其基本目录结构如下：

```Plain
project
|--pom.xml
|--src
|  |--main
|  |  |--java
|  |  |--resources
|  |--test
|--target
```

### 第 2 步：添加依赖项

在 **pom.xml** 文件中添加以下依赖项：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>udf</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.76</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.3.0</version>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 第 3 步：编译 UDF

使用 Java 编程语言编译 UDF。

#### 编译标量 UDF

标量 UDF 对单行数据进行操作并返回单个值。当您在查询中使用标量 UDF 时，每行对应于结果集中的一个值。典型的标量函数包括 `UPPER`、`LOWER`、`ROUND` 和 `ABS`。

假设您的 JSON 数据中的字段值是 JSON 字符串而不是 JSON 对象。当您使用 SQL 语句提取 JSON 字符串时，您需要运行 `GET_JSON_STRING` 两次，例如：`GET_JSON_STRING(GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key"), "$.k0")`。

为了简化 SQL 语句，您可以编译一个可以直接提取 JSON 字符串的标量 UDF，例如 `MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // JSONPath 库即使字段值是 JSON 字符串也可以完全展开。
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

用户定义的类必须实现下表中描述的方法。

> **注意**
> 方法中请求参数和返回参数的数据类型必须与[步骤 6](#step-6-create-the-udf-in-starrocks)要执行的 `CREATE FUNCTION` 语句中声明的数据类型一致，并符合本主题的“[SQL 数据类型与 Java 数据类型的映射](#mapping-between-sql-data-types-and-java-data-types)”部分中提供的映射。

|方法|描述|
|---|---|
|TYPE1 evaluate(TYPE2, ...)|运行 UDF。evaluate() 方法需要公共成员访问级别。|

#### 编译 UDAF

UDAF 对多行数据进行操作并返回单个值。典型的聚合函数包括 `SUM`、`COUNT`、`MAX` 和 `MIN`，它们聚合每个 `GROUP BY` 子句中指定的多行数据并返回单个值。

假设您要编译一个名为 `MY_SUM_INT` 的 UDAF。与内置聚合函数 `SUM` 返回 `BIGINT` 类型值不同，`MY_SUM_INT` 函数仅支持请求参数和返回 `INT` 数据类型的参数。

```Java
package com.starrocks.udf.sample;

public class SumInt {
    public static class State {
        int counter = 0;
        public int serializeLength() { return 4; }
    }

    public State create() {
        return new State();
    }

    public void destroy(State state) {
    }

    public final void update(State state, Integer val) {
        if (val != null) {
            state.counter += val;
        }
    }

    public void serialize(State state, java.nio.ByteBuffer buff) {
        buff.putInt(state.counter);
    }

    public void merge(State state, java.nio.ByteBuffer buffer) {
        int val = buffer.getInt();
        state.counter += val;
    }

    public Integer finalize(State state) {
        return state.counter;
    }
}
```

用户定义的类必须实现下表中描述的方法。

> **注意**
> 方法中的请求参数和返回参数的数据类型必须与[步骤 6](#step-6-create-the-udf-in-starrocks)要执行的 `CREATE FUNCTION` 语句中声明的数据类型相同，并符合本主题的“[SQL 数据类型与 Java 数据类型的映射](#mapping-between-sql-data-types-and-java-data-types)”部分中提供的映射。

|方法|描述|
|---|---|
|State create()|创建一个状态。|
|void destroy(State)|销毁一个状态。|
|void update(State, ...)|更新状态。除了第一个参数 `State` 外，您还可以在 UDF 声明中指定一个或多个请求参数。|
|void serialize(State, ByteBuffer)|将状态序列化到字节缓冲区中。|
|void merge(State, ByteBuffer)|从字节缓冲区反序列化状态，并将字节缓冲区合并到状态中作为第一个参数。|
|TYPE finalize(State)|从某个状态获取 UDF 的最终结果。|

在编译过程中，还必须使用缓冲类 `java.nio.ByteBuffer` 和局部变量 `serializeLength`，如下表所示。

|类和局部变量|描述|
|---|---|
|java.nio.ByteBuffer()|缓冲区类，存储中间结果。中间结果在节点之间传输以供执行时可能会被序列化或反序列化。因此，您还必须使用 `serializeLength` 变量来指定中间结果反序列化所允许的长度。|
|serializeLength()|中间结果反序列化所允许的长度。单位：字节。将此局部变量设置为 `INT` 类型值。例如，`State { int counter = 0; public int serializeLength() { return 4; }}` 指定中间结果为 `INT` 数据类型，反序列化长度为 4 字节。您可以根据您的业务需求调整这些设置。例如，如果要指定中间结果的数据类型为 `LONG`，反序列化的长度为 8 字节，则传递 `State { long counter = 0; public int serializeLength() { return 8; }}`。|

`java.nio.ByteBuffer` 类中存储的中间结果的反序列化需要注意以下几点：

- 无法调用依赖于 `ByteBuffer` 类的 `remaining()` 方法来反序列化状态。
- 无法在 `ByteBuffer` 类上调用 `clear()` 方法。
- `serializeLength` 的值必须与写入数据的长度相同。否则，在序列化和反序列化过程中会产生错误的结果。

#### 编译 UDWF

与常规聚合函数不同，UDWF 对一组多行（统称为窗口）进行操作，并为每行返回一个值。典型的窗口函数包含一个 `OVER` 子句，该子句将行分为多个集合。它对每组行执行计算并返回每行的值。

假设您要编译名为 `MY_WINDOW_SUM_INT` 的 UDWF。与内置聚合函数 `SUM` 返回 `BIGINT` 类型值不同，`MY_WINDOW_SUM_INT` 函数仅支持请求参数和返回 `INT` 数据类型的参数。

```Java
package com.starrocks.udf.sample;

public class WindowSumInt {    
    public static class State {
        int counter = 0;
        public int serializeLength() { return 4; }
        @Override
        public String toString() {
            return "State{" +
                    "counter=" + counter +
                    '}';
        }
    }

    public State create() {
        return new State();
    }

    public void destroy(State state) {

    }

    public void update(State state, Integer val) {
        if (val != null) {
            state.counter += val;
        }
    }

    public void serialize(State state, java.nio.ByteBuffer buff) {
        buff.putInt(state.counter);
    }

    public void merge(State state, java.nio.ByteBuffer buffer) {
        int val = buffer.getInt();
        state.counter += val;
    }

    public Integer finalize(State state) {
        return state.counter;
    }

    public void reset(State state) {
        state.counter = 0;
    }

    public void windowUpdate(State state,
                            int peer_group_start, int peer_group_end,
                            int frame_start, int frame_end,
                            Integer[] inputs) {
        for (int i = frame_start; i < frame_end; ++i) {
            state.counter += inputs[i];
        }
    }
}
```

用户定义的类必须实现 UDAFs 所需的方法（因为 UDWF 是一种特殊的聚合函数）和下表中描述的 `windowUpdate()` 方法。

> **注意**
> 方法中的请求参数和返回参数的数据类型必须与[步骤 6](#step-6-create-the-udf-in-starrocks)要执行的 `CREATE FUNCTION` 语句中声明的数据类型相同，并符合本主题的“[SQL 数据类型与 Java 数据类型的映射](#mapping-between-sql-data-types-and-java-data-types)”部分中提供的映射。

|方法|描述|
|---|---|
```
|方法|说明|
|---|---|
|TYPE[] process()|运行 UDTF 并返回一个数组。|

### 第四步：打包 Java 项目

执行以下命令打包 Java 项目：

```Bash
mvn package
```

在 **target** 文件夹中生成以下 JAR 文件：**udf-1.0-SNAPSHOT.jar** 和 **udf-1.0-SNAPSHOT-jar-with-dependencies.jar**。

### 第五步：上传 Java 项目

将 JAR 文件 **udf-1.0-SNAPSHOT-jar-with-dependencies.jar** 上传到一个始终运行且对 StarRocks 集群中的所有 FE 和 BE 可访问的 HTTP 服务器。然后，运行以下命令部署该文件：

```Bash
mvn deploy 
```

您可以使用 Python 设置一个简单的 HTTP 服务器并将 JAR 文件上传到该服务器。

> **注意**
> 在[第六步](#step-6-create-the-udf-in-starrocks)中，FE 将检查包含 UDF 代码的 JAR 文件并计算校验和，BE 将下载并执行 JAR 文件。

### 第六步：在 StarRocks 中创建 UDF

StarRocks 允许您在两种类型的命名空间中创建 UDF：数据库命名空间和全局命名空间。

- 如果您对 UDF 没有可见性或隔离性要求，可以将其创建为全局 UDF。然后，您可以使用函数名称引用全局 UDF，而无需将目录和数据库名称作为前缀。
- 如果您对 UDF 有可见性或隔离性要求，或者需要在不同数据库中创建相同的 UDF，可以在每个单独的数据库中创建它。这样，如果您的会话连接到目标数据库，您可以使用函数名称引用 UDF。如果您的会话连接到目标数据库以外的其他目录或数据库，则需要通过包括目录和数据库名称作为前缀来引用 UDF，例如 `catalog.database.function`。

> **注意**
> 在创建和使用全局 UDF 之前，您必须联系系统管理员授予您所需的权限。有关详细信息，请参阅 [GRANT](../sql-statements/account-management/GRANT.md)。

上传 JAR 包后，您可以在 StarRocks 中创建 UDF。对于全局 UDF，您必须在创建语句中包含 `GLOBAL` 关键字。

#### 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

#### 参数

|参数|必填|说明|
|---|---|---|
|GLOBAL|否|是否创建全局 UDF，从 v3.0 开始支持。|
|AGGREGATE|否|是否创建 UDAF 或 UDWF。|
|TABLE|否|是否创建 UDTF。如果未指定 `AGGREGATE` 和 `TABLE`，则创建标量函数。|
|function_name|是|您要创建的函数的名称。您可以在此参数中包含数据库的名称，例如 `db1.my_func`。如果 `function_name` 包含数据库名称，则在该数据库中创建 UDF。否则，将在当前数据库中创建 UDF。新函数的名称及其参数不能与目标数据库中的现有名称相同。否则，无法创建该函数。如果函数名相同但参数不同，则创建成功。|
|arg_type|是|函数的参数类型。添加的参数可以用 `, ...` 表示。支持的数据类型请参见 [SQL 数据类型与 Java 数据类型的映射](#mapping-between-sql-data-types-and-java-data-types)。|
|return_type|是|函数的返回类型。有关支持的数据类型，请参阅 [Java UDF](#mapping-between-sql-data-types-and-java-data-types)。|
|PROPERTIES|是|函数的属性，根据要创建的 UDF 的类型而变化。|

#### 创建标量 UDF

运行以下命令创建您在前面示例中编译的标量 UDF：

```SQL
CREATE [GLOBAL] FUNCTION MY_UDF_JSON_GET(string, string) 
RETURNS string
PROPERTIES (
    "symbol" = "com.starrocks.udf.sample.UDFJsonGet", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

|参数|说明|
|---|---|
|symbol|UDF 所属 Maven 项目的类名。此参数的值采用 `<package_name>.<class_name>` 格式。|
|type|UDF 的类型。将值设置为 `StarrocksJar`，指定 UDF 是基于 Java 的函数。|
|file|可以从中下载包含 UDF 代码的 JAR 文件的 HTTP URL。此参数的值采用 `http://<http_server_ip>:<http_server_port>/<jar_package_name>` 格式。|

#### 创建 UDAF

运行以下命令创建您在前面示例中编译的 UDAF：

```SQL
CREATE [GLOBAL] AGGREGATE FUNCTION MY_SUM_INT(INT) 
RETURNS INT
PROPERTIES 
( 
    "symbol" = "com.starrocks.udf.sample.SumInt", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

PROPERTIES 中的参数说明与[创建标量 UDF](#创建标量-udf)中的说明相同。

#### 创建 UDWF

运行以下命令创建您在前面示例中编译的 UDWF：

```SQL
CREATE [GLOBAL] AGGREGATE FUNCTION MY_WINDOW_SUM_INT(INT)
RETURNS INT
PROPERTIES 
(
    "analytic" = "true",
    "symbol" = "com.starrocks.udf.sample.WindowSumInt", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"    
);
```

`analytic`：UDF 是否为窗口函数。将值设置为 `true`。其他属性的描述与[创建标量 UDF](#创建标量-udf)中的相同。

#### 创建 UDTF

运行以下命令创建您在前面示例中编译的 UDTF：

```SQL
CREATE [GLOBAL] TABLE FUNCTION MY_UDF_SPLIT(string)
RETURNS string
PROPERTIES 
(
    "symbol" = "com.starrocks.udf.sample.UDFSplit", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

PROPERTIES 中的参数说明与[创建标量 UDF](#创建标量-udf)中的说明相同。

### 第七步：使用 UDF

创建 UDF 后，您可以根据业务需求进行测试和使用。

#### 使用标量 UDF

运行以下命令使用您在前面示例中创建的标量 UDF：

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### 使用 UDAF

运行以下命令使用您在前面示例中创建的 UDAF：

```SQL
SELECT MY_SUM_INT(col1);
```

#### 使用 UDWF

运行以下命令使用您在前面示例中创建的 UDWF：

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### 使用 UDTF

运行以下命令使用您在前面示例中创建的 UDTF：

```Plain
-- 假设您有一个名为 t1 的表，其列 a、b 和 c1 的信息如下：
SELECT t1.a,t1.b,t1.c1 FROM t1;
> 输出：
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- 运行 MY_UDF_SPLIT() 函数。
SELECT t1.a,t1.b, MY_UDF_SPLIT FROM t1, MY_UDF_SPLIT(t1.c1); 
> 输出：
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **注意**
- 上述代码片段中的第一个 `MY_UDF_SPLIT` 是第二个 `MY_UDF_SPLIT` 返回的列的别名，后者是一个函数。
- 您不能使用 `AS t2(f1)` 来指定要返回的表及其列的别名。

## 查看 UDF

执行以下命令查询 UDF：

```SQL
SHOW [GLOBAL] FUNCTIONS;
```

有关详细信息，请参阅 [SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)。

## 删除 UDF

运行以下命令删除 UDF：

```SQL
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

有关更多信息，请参阅 [DROP FUNCTION](../sql-statements/data-definition/DROP_FUNCTION.md)。

## SQL 数据类型与 Java 数据类型之间的映射

|SQL 类型|Java 类型|
|---|---|
|BOOLEAN|java.lang.Boolean|
|TINYINT|java.lang.Byte|
|SMALLINT|java.lang.Short|
|INT|java.lang.Integer|
|BIGINT|java.lang.Long|
|FLOAT|java.lang.Float|
|DOUBLE|java.lang.Double|
|STRING/VARCHAR|java.lang.String|

## 参数设置

在 StarRocks 集群中每个 Java 虚拟机 (JVM) 的 **be/conf/hadoop_env.sh** 文件中配置以下环境变量以控制内存使用。您还可以在该文件中配置其他参数。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## 常见问题解答

我在创建 UDF 时可以使用静态变量吗？不同 UDF 的静态变量是否会相互影响？

是的，您在编译 UDF 时可以使用静态变量。即使 UDF 有相同名称的类，不同 UDF 的静态变量也是相互隔离的，不会相互影响。
```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## 常见问题

我可以在创建 UDF 时使用静态变量吗？不同 UDF 的静态变量会相互影响吗？

是的，您在编译 UDF 时可以使用静态变量。不同 UDF 的静态变量是隔离的，即使 UDF 中有同名的类，它们也不会相互影响。