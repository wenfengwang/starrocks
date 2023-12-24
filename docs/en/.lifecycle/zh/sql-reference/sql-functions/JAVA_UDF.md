---
displayed_sidebar: English
---

# Java UDF

从 v2.2.0 开始，您可以使用 Java 编程语言编译用户自定义函数（UDFs）以满足您的特定业务需求。

从 v3.0 开始，StarRocks 支持全局 UDFs，您只需要在相关的 SQL 语句（CREATE/SHOW/DROP）中包含 `GLOBAL` 关键字即可。

本主题介绍了如何开发和使用各种 UDFs。

目前，StarRocks 支持标量 UDFs、用户定义的聚合函数（UDAFs）、用户定义的窗口函数（UDWFs）和用户定义的表函数（UDTFs）。

## 先决条件

- 您已安装 [Apache Maven](https://maven.apache.org/download.cgi)，因此可以创建和编译 Java 项目。

- 您已在服务器上安装了 JDK 1.8。

- Java UDF 功能已启用。您可以在 FE 配置文件 **fe/conf/fe.conf** 中将 FE 配置项 `enable_udf` 设置为 `true` 以启用此功能，然后重新启动 FE 节点以使设置生效。有关更多信息，请参见[参数配置](../../administration/FE_configuration.md)。

## 开发和使用 UDFs

您需要创建一个 Maven 项目，并使用 Java 编程语言编译您需要的 UDF。

### 步骤 1：创建 Maven 项目

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

### 步骤 2：添加依赖项

将以下依赖项添加到 **pom.xml** 文件：

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

### 步骤 3：编译 UDF

使用 Java 编程语言编译 UDF。

#### 编译标量 UDF

标量 UDF 对单行数据进行操作并返回单个值。在查询中使用标量 UDF 时，每行对应于结果集中的单个值。典型的标量函数包括 `UPPER`、`LOWER`、`ROUND` 和 `ABS`。

假设 JSON 数据中字段的值是 JSON 字符串，而不是 JSON 对象。使用 SQL 语句提取 JSON 字符串时，需要运行 `GET_JSON_STRING` 两次，例如 `GET_JSON_STRING(GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key"), "$.k0")`。

为了简化 SQL 语句，您可以编译一个可以直接提取 JSON 字符串的标量 UDF，例如 `MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // JSONPath 库可以完全展开字段的值，即使字段的值是 JSON 字符串。
            return JSONPath.read(obj, key).toString();
        } catch (Exception e) {
            return null;
        }
    }
}
```

用户定义的类必须实现下表中描述的方法。

> **注意**
>
> 该方法中的请求参数和返回参数的数据类型必须与 `CREATE FUNCTION` 步骤6中要执行的语句中声明的数据类型相同，并且符合本主题中“SQL 数据类型与 Java 数据类型的映射关系”一节中提供的映射关系。

| 方法                     | 描述                                                  |
| -------------------------- | ------------------------------------------------------------ |
| TYPE1 evaluate(TYPE2, ...) | 运行 UDF。evaluate() 方法需要公共成员访问级别。 |

#### 编译 UDAF

UDAF 对多行数据进行操作，并返回单个值。典型的聚合函数包括 `SUM`、`COUNT`、`MAX` 和 `MIN`，它们聚合每个 GROUP BY 子句中指定的多行数据并返回单个值。

假设您要编译一个名为 `MY_SUM_INT` 的 UDAF。与返回 BIGINT 类型值的内置聚合函数 `SUM` 不同，`MY_SUM_INT` 函数仅支持 INT 数据类型的请求参数和返回参数。

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
>
> 方法中的请求参数和返回参数的数据类型必须与 `CREATE FUNCTION` 步骤6中要执行的语句中声明的数据类型一致，并且符合本主题中“SQL 数据类型与 Java 数据类型的映射关系”一节中提供的映射关系。

| 方法                            | 描述                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| State create()                    | 创建状态。                                             |
| void destroy(State)               | 销毁状态。                                            |
| void update(State, ...)           | 更新状态。除了第一个参数 `State`，您还可以在 UDF 声明中指定一个或多个请求参数。 |
| void serialize(State, ByteBuffer) | 将状态序列化到字节缓冲区中。                     |
| void merge(State, ByteBuffer)     | 反序列化字节缓冲区中的状态，并将字节缓冲区合并到状态中作为第一个参数。 |
| TYPE finalize(State)              | 从状态获取 UDF 的最终结果。            |

在编译过程中，还必须使用下表中描述的 buffer 类 `java.nio.ByteBuffer` 和局部变量 `serializeLength`。

| 类和局部变量 | 描述                                                  |
| ------------------------ | ------------------------------------------------------------ |
| java.nio.ByteBuffer()    | 缓冲区类，用于存储中间结果。中间结果在节点之间传输以执行时，可能会被序列化或反序列化。因此，还必须使用 `serializeLength` 变量来指定中间结果的反序列化所允许的长度。 |
| serializeLength()        | 中间结果反序列化所允许的长度。单位：字节。将此局部变量设置为 INT 类型值。例如，`State { int counter = 0; public int serializeLength() { return 4; }}` 指定中间结果为 INT 数据类型，反序列化的长度为 4 个字节。您可以根据业务需求调整这些设置。例如，如果要将中间结果的数据类型指定为 LONG，将反序列化的长度指定为 8 个字节，请传递 `State { long counter = 0; public int serializeLength() { return 8; }}`。 |

对于类中存储的中间结果的反序列化，请注意以下几点：

- 不能调用依赖于 `ByteBuffer` 类的 `remaining()` 方法来反序列化状态。
- 不能在 `ByteBuffer` 类上调用 `clear()` 方法。
- `serializeLength` 的值必须与写入数据的长度相同。否则，在序列化和反序列化过程中会生成不正确的结果。

#### 编译 UDWF

与常规聚合函数不同，UDWF 对一组多行（统称为一个窗口）进行操作，并为每一行返回一个值。典型的窗口函数包括一个 `OVER` 子句，将行划分为多个集合。它对每组行执行计算，并为每行返回一个值。

假设您要编译一个名为 `MY_WINDOW_SUM_INT` 的 UDWF。与返回 BIGINT 类型值的内置聚合函数 `SUM` 不同，该函数仅支持 INT 数据类型的请求参数和返回参数。

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
        for (int i = (int)frame_start; i < (int)frame_end; ++i) {
            state.counter += inputs[i];
        }
    }
}
```

用户定义的类必须实现 UDAFs 所需的方法（因为 UDWF 是一种特殊的聚合函数），以及下表中描述的 `windowUpdate()` 方法。

> **注意**
>
> 该方法中的请求参数和返回参数的数据类型必须与 `CREATE FUNCTION` 步骤6中要执行的语句中声明的数据类型相同，并且符合本主题中“SQL 数据类型与 Java 数据类型的映射关系”一节中提供的映射关系。

| 方法                                                   | 描述                                                  |
| -------------------------------------------------------- | ------------------------------------------------------------ |

| void windowUpdate(State state, int, int, int, int, ...) | 更新窗口的数据。有关 UDWF 的详细信息，请参阅[窗口函数](../sql-functions/Window_function.md)。每次输入一行作为输入时，此方法都会获取窗口信息并相应地更新中间结果。<ul><li>`peer_group_start`：当前分区的起始位置。 `PARTITION BY` 在 OVER 子句中用于指定分区列。分区列中具有相同值的行被视为位于同一分区中。</li><li>`peer_group_end`：当前分区的结束位置。</li><li>`frame_start`：当前窗口框架的起始位置。window frame 子句指定一个计算范围，该范围涵盖当前行以及与当前行在指定距离内的行。例如，`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING`指定一个计算范围，该计算范围涵盖当前行、当前行之前的上一行以及当前行之后的下一行。</li><li>`frame_end`：当前窗口框架的结束位置。</li><li>`inputs`：作为窗口输入的数据。数据是仅支持特定数据类型的数组包。在此示例中，输入 INT 值作为输入，数组包为 Integer[]。</li></ul> |

#### 编译 UDTF

UDTF 读取一行数据并返回多个值，这些值可以被视为一个表。表值函数通常用于将行转换为列。

> **注意**
>
> StarRocks 允许 UDTF 返回一个由多行一列组成的表。

假设您要编译一个名为 `MY_UDF_SPLIT` 的 UDTF。`MY_UDF_SPLIT` 函数允许您使用空格作为分隔符，并支持 STRING 数据类型的请求参数和返回参数。

```Java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

用户定义类定义的方法必须满足以下要求：

> **注意**
>
> 方法中的请求参数和返回参数的数据类型必须与`CREATE FUNCTION`步骤6中[要执行的语句中](#step-6-create-the-udf-in-starrocks)声明的数据类型相同，并且符合本文“SQL数据类型与Java数据类型[的映射关系”一节](#mapping-between-sql-data-types-and-java-data-types)中提供的映射关系。

| 方法           | 描述                         |
| ---------------- | ----------------------------------- |
| TYPE[] process() | 运行 UDTF 并返回一个数组。 |

### 步骤 4：打包 Java 项目

执行以下命令，对 Java 项目进行打包：

```Bash
mvn package
```

在目标文件夹中生成以下 JAR 文件：`udf-1.0-SNAPSHOT.jar` 和 `udf-1.0-SNAPSHOT-jar-with-dependencies.jar`。

### 步骤 5：上传 Java 项目

将 JAR 文件 `udf-1.0-SNAPSHOT-jar-with-dependencies.jar` 上传到 StarRocks 集群中所有 FE 和 BE 都可以访问的 HTTP 服务器。然后，运行以下命令以部署该文件：

```Bash
mvn deploy 
```

您可以使用 Python 设置一个简单的 HTTP 服务器，并将 JAR 文件上传到该 HTTP 服务器。

> **注意**
>
> [在步骤6](#step-6-create-the-udf-in-starrocks)中，FE 会检查包含 UDF 代码的 JAR 文件并计算校验和，BE 会下载并执行 JAR 文件。

### 第 6 步：在 StarRocks 中创建 UDF

StarRocks 支持创建两种类型的 UDF：数据库命名空间和全局命名空间。

- 如果对 UDF 没有可见性或隔离要求，则可以将其创建为全局 UDF。然后，您可以使用函数名称引用全局 UDF，而无需将目录和数据库名称作为函数名称的前缀。
- 如果对 UDF 有可见性或隔离要求，或者需要在不同的数据库中创建相同的 UDF，则可以在每个单独的数据库中创建它。因此，如果您的会话连接到目标数据库，则可以使用函数名称引用 UDF。如果会话连接到目标数据库以外的其他目录或数据库，则需要通过将目录和数据库名称作为函数名称的前缀（例如 `catalog.database.function`）。

> **通知**
>
> 在创建和使用全局 UDF 之前，您需要与系统管理员联系，以授予您所需的权限。有关详细信息，请参阅 [GRANT](../sql-statements/account-management/GRANT.md)。

上传 JAR 包后，您可以在 StarRocks 中创建 UDF。对于全局 UDF，必须在 `GLOBAL` 创建语句中包含关键字。

#### 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

#### 参数

| **参数**      | **必填** | **描述**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | 否       | 是否创建全局 UDF，从 v3.0 开始支持。 |
| AGGREGATE     | 否       | 是创建 UDAF 还是 UDWF。       |
| TABLE         | 否       | 是否创建 UDTF。如果两者都 `AGGREGATE` 和 `TABLE` 未指定，则创建标量函数。               |
| function_name | 是       | 要创建的函数的名称。您可以在此参数中包含数据库的名称，例如，`db1.my_func`。如果 `function_name` 包含数据库名称，则在该数据库中创建 UDF。否则，将在当前数据库中创建 UDF。新函数的名称及其参数不能与目标数据库中的现有名称相同。否则，无法创建函数。如果函数名称相同但参数不同，则创建成功。 |
| arg_type      | 是       | 函数的参数类型。添加的参数可以用 `, ...` 表示。有关支持的数据类型，请参阅 [SQL 数据类型和 Java 数据类型之间的映射](#mapping-between-sql-data-types-and-java-data-types)。|
| return_type      | 是       | 函数的返回类型。有关支持的数据类型，请参阅 [Java UDF](#mapping-between-sql-data-types-and-java-data-types)。 |
| PROPERTIES    | 是       | 函数的属性，具体取决于要创建的 UDF 的类型。 |

#### 创建标量 UDF

执行以下命令，创建在上述示例中编译的标量 UDF。

```SQL
CREATE [GLOBAL] FUNCTION MY_UDF_JSON_GET(string, string) 
RETURNS string
PROPERTIES (
    "symbol" = "com.starrocks.udf.sample.UDFJsonGet", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

| 参数 | 描述                                                  |
| --------- | ------------------------------------------------------------ |
| symbol    | UDF 所属的 Maven 项目的类名称。此参数的值采用以下 `<package_name>.<class_name>` 格式。 |
| type      | UDF 的类型。将值设置为 `StarrocksJar`，这将指定 UDF 是基于 Java 的函数。 |
| file      | 可从中下载包含 UDF 代码的 JAR 文件的 HTTP URL。此参数的值采用以下 `http://<http_server_ip>:<http_server_port>/<jar_package_name>` 格式。 |

#### 创建 UDAF

执行以下命令，创建在上述示例中编译的 UDAF。

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

PROPERTIES 中的参数描述与创建标量 UDF 中的参数描述相同。

#### 创建 UDWF

执行以下命令，创建在上述示例中编译的 UDWF。

```SQL
CREATE [GLOBAL] AGGREGATE FUNCTION MY_WINDOW_SUM_INT(Int)
RETURNS Int
properties 
(
    "analytic" = "true",
    "symbol" = "com.starrocks.udf.sample.WindowSumInt", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"    
);
```

`analytic`：UDF 是否为窗口函数。将值设置为 `true`。其他属性的描述与创建标量 UDF 中的描述相同。

#### 创建 UDTF

执行以下命令，创建在上述示例中编译的 UDTF。

```SQL
CREATE [GLOBAL] TABLE FUNCTION MY_UDF_SPLIT(string)
RETURNS string
properties 
(
    "symbol" = "com.starrocks.udf.sample.UDFSplit", 
    "type" = "StarrocksJar", 
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

PROPERTIES 中的参数描述与创建标量 UDF 中的参数描述相同。

### 步骤 7：使用 UDF

UDF 创建完成后，您可以根据业务需求进行测试和使用。

#### 使用标量 UDF

执行以下命令，使用在上述示例中创建的标量 UDF。

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### 使用 UDAF

执行以下命令，使用在上述示例中创建的 UDAF。

```SQL
SELECT MY_SUM_INT(col1);
```

#### 使用 UDWF

执行以下命令，使用在上述示例中创建的 UDWF。

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### 使用 UDTF

执行以下命令，使用在上述示例中创建的 UDTF。

```Plain
-- 假设您有一个名为 t1 的表，其列 a、b 和 c1 的信息如下：
SELECT t1.a,t1.b,t1.c1 FROM t1;
> 输出:
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- 运行 MY_UDF_SPLIT() 函数。
SELECT t1.a,t1.b, MY_UDF_SPLIT FROM t1, MY_UDF_SPLIT(t1.c1); 
> 输出:
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **注意**
>
> -  前面代码段中的第一个 `MY_UDF_SPLIT` 是第二个返回的列的别名 `MY_UDF_SPLIT`，它是一个函数。
> - 不能使用 `AS t2(f1)` 指定要返回的表及其列的别名。

## 查看 UDF

执行以下命令，查询 UDF。

```SQL
SHOW [GLOBAL] FUNCTIONS;
```

有关详细信息，请参阅 [SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)。

## 删除 UDF

执行以下命令，删除 UDF。

```SQL
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

有关详细信息，请参阅 [DROP FUNCTION](../sql-statements/data-definition/DROP_FUNCTION.md)。

## SQL 数据类型和 Java 数据类型之间的映射

| SQL 类型       | Java 类型         |
| -------------- | ----------------- |
| BOOLEAN        | java.lang.Boolean |
| TINYINT        | java.lang.Byte    |
| SMALLINT       | java.lang.Short   |
| INT            | java.lang.Integer |
| BIGINT         | java.lang.Long    |
| FLOAT          | java.lang.Float   |
| DOUBLE         | java.lang.Double  |
| STRING/VARCHAR | java.lang.String  |

## 参数设置
在 StarRocks 集群中，需要在每个 Java 虚拟机（JVM）的 **be/conf/hadoop_env.sh** 文件中配置以下环境变量，以控制内存使用。您也可以在该文件中配置其他参数。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## 常见问题

在创建 UDF 时，我可以使用静态变量吗？不同 UDF 的静态变量是否会相互影响？

是的，您可以在编译 UDF 时使用静态变量。不同 UDF 的静态变量是相互隔离的，即使 UDF 具有相同名称的类也不会相互影响。