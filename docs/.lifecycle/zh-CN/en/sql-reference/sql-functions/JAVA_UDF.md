---
displayed_sidebar: "Chinese"
---

# Java 用户自定义函数（UDFs）

从v2.2.0版本开始，您可以使用Java编程语言编译用户定义函数（UDFs）以满足特定的业务需求。

从v3.0版本开始，StarRocks支持全局UDFs，您只需要在相关的SQL语句（CREATE/SHOW/DROP）中包含`GLOBAL`关键字即可。

本主题介绍如何开发和使用各种UDFs。

目前，StarRocks支持标量UDFs、用户定义聚合函数（UDAFs）、用户定义窗口函数（UDWFs）和用户定义表函数（UDTFs）。

## 先决条件

- 您已经安装了[Apache Maven](https://maven.apache.org/download.cgi)，这样您就可以创建并编译Java项目。

- 在您的服务器上安装了JDK 1.8。

- 已启用Java UDF功能。您可以在FE配置文件**fe/conf/fe.conf**中将FE配置项`enable_udf`设置为`true`以启用此功能，然后重新启动FE节点使设置生效。有关更多信息，请参见[参数配置](../../administration/Configuration.md)。

## 开发和使用UDFs

您需要使用Java编程语言创建一个Maven项目，并编译您需要的UDF。

### 步骤1：创建一个Maven项目

创建一个Maven项目，其基本目录结构如下：

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

### 步骤2：添加依赖项

向**pom.xml**文件添加以下依赖项：

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

### 步骤3：编译UDF

使用Java编程语言编译UDF。

#### 编译标量UDF

标量UDF在单个数据行上操作，并返回一个值。当您在查询中使用标量UDF时，每行对应结果集中的单个值。典型的标量函数包括`UPPER`、`LOWER`、`ROUND`和`ABS`。

假设您的JSON数据字段的值是JSON字符串而不是JSON对象。当您使用SQL语句提取JSON字符串时，您需要运行`GET_JSON_STRING`两次，例如`GET_JSON_STRING(GET_JSON_STRING('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key"), "$.k0")`。

为了简化SQL语句，您可以编译一个标量UDF，可以直接提取JSON字符串，例如`MY_UDF_JSON_GET('{"key":"{\\"k0\\":\\"v0\\"}"}', "$.key.k0")`。

```Java
package com.starrocks.udf.sample;

import com.alibaba.fastjson.JSONPath;

public class UDFJsonGet {
    public final String evaluate(String obj, String key) {
        if (obj == null || key == null) return null;
        try {
            // 即使字段的值是JSON字符串，JSONPath库也可以完全展开。
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
> 方法中的请求参数和返回参数的数据类型必须与在[步骤6](#step-6-create-the-udf-in-starrocks){3cb8d7}中要执行的`CREATE FUNCTION`语句中声明的类型相同，并符合本主题的"[SQL数据类型与Java数据类型之间的映射](#mapping-between-sql-data-types-and-java-data-types)"中提供的映射。

| 方法                     | 描述                           |
| ------------------------ | ------------------------------ |
| TYPE1 evaluate(TYPE2, ...) | 运行UDF。`evaluate()`方法需要具有`public`访问级别。 |

#### 编译UDAF

UDAF在多个数据行上操作，并返回一个值。典型的聚合函数包括`SUM`、`COUNT`、`MAX`和`MIN`，它们在每个GROUP BY子句中指定的多个数据行上进行聚合，并返回单个值。

假设您想要编译一个名为`MY_SUM_INT`的UDAF。与内置的聚合函数`SUM`不同，它返回BIGINT类型的值，`MY_SUM_INT`函数仅支持INT数据类型的请求参数和返回参数。

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
            state.counter+= val;
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
> 方法中的请求参数和返回参数的数据类型必须与在[步骤6](#step-6-create-the-udf-in-starrocks){3cb8d7}中要执行的`CREATE FUNCTION`语句中声明的类型相同，并符合本主题的"[SQL数据类型与Java数据类型之间的映射](#mapping-between-sql-data-types-and-java-data-types)"中提供的映射。

| 方法                     | 描述                   |
| ------------------------ | ---------------------- |
| State create()           | 创建状态。               |
| void destroy(State)      | 销毁状态。               |
| void update(State, ...)  | 更新状态。除了第一个参数`State`外，您还可以在UDF声明中指定一个或多个请求参数。 |
| void serialize(State, ByteBuffer) | 将状态序列化为字节缓冲区。     |
| void merge(State, ByteBuffer)     | 从字节缓冲器中对状态进行反序列化，并将字节缓冲器合并到状态作为第一个参数。 |
| TYPE finalize(State)      | 从状态中获取UDF的最终结果。 |

在编译过程中，您还必须使用缓冲器类`java.nio.ByteBuffer`和本地变量`serializeLength`，这些在下表中进行了描述。

| 类和本地变量           | 描述                                    |
| ---------------------- | --------------------------------------- |
| java.nio.ByteBuffer() | 缓冲器类，用于存储中间结果。传输节点之间对执行的中间结果进行序列化或反序列化时，您还必须使用`serializeLength`变量指定允许的用于中间结果反序列化的长度。 |
| serializeLength()        | 允许用于中间结果反序列化的长度。单位：字节。将此本地变量设置为INT类型的值。例如，`State { int counter = 0; public int serializeLength() { return 4; }}` 指定中间结果为INT数据类型，反序列化长度为4字节。您可以根据业务需求调整这些设置。例如，如果要将中间结果的数据类型指定为LONG，并且将反序列化长度指定为8字节，则传递 `State { long counter = 0; public int serializeLength() { return 8; }}`。 |

请注意以下关于存储在`java.nio.ByteBuffer`类中的中间结果的反序列化的几点：

- 依赖于`ByteBuffer`类的remaining()方法无法调用以对状态进行反序列化。
- 不能在`ByteBuffer`类上调用clear()方法。
- `serializeLength`的值必须与写入数据的长度相同，否则在序列化和反序列化过程中会生成不正确的结果。

#### 编译UDWF

与常规聚合函数不同，UDWF作用于一组多行，集体称为窗口，并为每一行返回一个值。典型的窗口函数包括一个`OVER`子句，将行分成多个集合。它对每个行集执行计算，并为每一行返回一个值。

假设您要编译一个名为`MY_WINDOW_SUM_INT`的UDWF。与内置的聚合函数`SUM`不同，`MY_WINDOW_SUM_INT`函数仅支持请求参数，并返回INT数据类型的参数。

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
            state.counter+=val;
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

用户定义的类必须实现UDAF需要的方法（因为UDWF是一个特殊的聚合函数），以及下表中描述的windowUpdate()方法。

> **提示**
>
> 方法中请求参数和返回参数的数据类型必须与将在[第6步](#step-6-create-the-udf-in-starrocks)中执行的`CREATE FUNCTION`语句中声明的类型相同，并符合本主题中提供的"[SQL数据类型与Java数据类型的映射](#映射-between-sql-data-types-and-java-data-types)"。

| 方法                                                   | 描述                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| void windowUpdate(State state, int, int, int , int, ...) | 更新窗口数据。有关UDWF的更多信息，请参见[W窗口函数](../sql-functions/Window_function.md)。每次输入一行作为输入时，此方法获取窗口信息并相应地更新中间结果。<ul><li>`peer_group_start`：当前分区的起始位置。`PARTITION BY`用于OVER子句，以指定分区列。具有分区列中相同值的行被视为在同一分区中。</li><li>`peer_group_end`：当前分区的结束位置。</li><li>`frame_start`：当前窗口帧的起始位置。窗口帧子句指定一个计算范围，包括当前行以及距离当前行指定距离内的行。例如，`ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING`指定一个计算范围，涵盖当前行，当前行之前的上一行，以及当前行之后的以下行。</li><li>`frame_end`：当前窗口帧的结束位置。</li><li>`inputs`：输入到窗口的数据。该数据是仅支持特定数据类型的数组包。在此示例中，INT值作为输入，数组包是Integer[]。</li></ul> |

#### 编译UDTF

UDTF读取一行数据并返回多个可视为表的值。表值函数通常用于将行转换为列。

> **提示**
>
> StarRocks允许UDTF返回由多行和一列组成的表。

假设您要编译一个名为`MY_UDF_SPLIT`的UDTF。`MY_UDF_SPLIT`函数允许您使用空格作为分隔符，并且支持STRING数据类型的请求参数和返回参数。

```Java
package com.starrocks.udf.sample;

public class UDFSplit{
    public String[] process(String in) {
        if (in == null) return null;
        return in.split(" ");
    }
}
```

用户定义的类所定义的方法必须满足以下要求：

> **提示**
>
> 方法中请求参数和返回参数的数据类型必须与将在[第6步](#step-6-create-the-udf-in-starrocks)中执行的`CREATE FUNCTION`语句中声明的类型相同，并符合本主题中提供的"[SQL数据类型与Java数据类型的映射](#mapping-between-sql-data-types-and-java-data-types)"。

| 方法           | 描述         |
| ---------------- | ----------------------------------- |
| TYPE[] process() | 运行UDTF并返回数组。 |

### 第4步：打包Java项目

运行以下命令以打包Java项目：

```Bash
mvn package
```

在**target**文件夹中生成以下JAR文件：**udf-1.0-SNAPSHOT.jar** 和 **udf-1.0-SNAPSHOT-jar-with-dependencies.jar**。

### 第5步：上传Java项目

将JAR文件**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**上传到一个持续运行并且对StarRocks集群中的所有FEs和BEs都可访问的HTTP服务器。然后，运行以下命令以部署该文件：

```Bash
mvn deploy 
```

您可以使用Python设置一个简单的HTTP服务器，并将JAR文件上传到该HTTP服务器。

> **提示**
>
> 在[第6步](#step-6-create-the-udf-in-starrocks)中，FE将检查包含UDF代码的JAR文件并计算校验和，而BE将下载并执行JAR文件。

### 第6步：在StarRocks中创建UDF

StarRocks允许您在两种类型的命名空间中创建UDF：数据库命名空间和全局命名空间。

- 如果您对UDF没有可见性或隔离要求，可以将其创建为全局UDF。然后，可以通过使用不包括目录和数据库名称作为前缀的函数名来引用全局UDF。
- 如果您对UDF有可见性或隔离要求，或者需要在不同数据库中创建相同的UDF，可以在每个独立的数据库中创建它。因此，如果您的会话连接到目标数据库，您可以使用函数名引用UDF。如果您的会话连接到与目标数据库不同的其他目录或数据库，则需要在函数名的前缀中包括目录和数据库名称。例如，`catalog.database.function`。

> **提示**
>
> 在创建和使用全局UDF之前，您必须联系系统管理员以授予所需的权限。有关更多信息，请参见[GRANT](../sql-statements/account-management/GRANT.md)。

上传JAR包之后，您可以在StarRocks中创建UDF。对于全局UDF，您必须在创建语句中包括`GLOBAL`关键字。

#### 语法

```sql
CREATE [GLOBAL][AGGREGATE | TABLE] FUNCTION function_name
(arg_type [, ...])
RETURNS return_type
PROPERTIES ("key" = "value" [, ...])
```

#### 参数

| **参数**      | **必需** | **描述**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| GLOBAL        | 否       | 是否创建全局UDF，从v3.0支持。 |
| AGGREGATE     | No       | 是否创建UDAF或UDWF。       |
| TABLE         | No       | 是否创建UDTF。如果未同时指定`AGGREGATE`和`TABLE`，则创建标量函数。               |
| function_name | Yes       | 要创建的函数名称。您可以在此参数中包含数据库名称，例如，`db1.my_func`。如果`function_name`包含数据库名称，则UDF将在该数据库中创建。否则，UDF将在当前数据库中创建。新函数及其参数的名称不能与目标数据库中的现有名称相同。否则，无法创建该函数。如果函数名称相同但参数不同，则创建成功。 |
| arg_type      | Yes       | 函数的参数类型。添加的参数可以用`,…`表示。有关支持的数据类型，请参见[SQL数据类型和Java数据类型之间的映射](#mapping-between-sql-data-types-and-java-data-types)。|
| return_type      | Yes       | 函数的返回类型。有关支持的数据类型，请参见[Java UDF](#mapping-between-sql-data-types-and-java-data-types)。 |
| PROPERTIES    | Yes       | 函数的属性，这取决于要创建的UDF的类型。 |

#### 创建标量UDF

运行以下命令来创建您在上面示例中编译的标量UDF：

```SQL
CREATE [GLOBAL] FUNCTION MY_UDF_JSON_GET(string, string) 
RETURNS string
PROPERTIES (
    "symbol" = "com.starrocks.udf.sample.UDFJsonGet", 
    "type" = "StarrocksJar",
    "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
);
```

| 参数     | 描述                                                  |
| --------- | ------------------------------------------------------------ |
| symbol    | Maven项目所属的UDF的类名。此参数的值采用`<package_name>.<class_name>`格式。 |
| type      | UDF的类型。将值设置为`StarrocksJar`，指定UDF是基于Java的函数。 |
| file      | 可以下载包含UDF代码的JAR文件的HTTP URL。此参数的值采用`http://<http_server_ip>:<http_server_port>/<jar_package_name>`格式。 |

#### 创建UDAF

运行以下命令来创建您在上面示例中编译的UDAF：

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

PROPERTIES中参数的描述与[创建标量UDF](#create-a-scalar-udf)中的描述相同。

#### 创建UDWF

运行以下命令来创建您在上面示例中编译的UDWF：

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

`analytic`: UDF是否为窗口函数。将值设置为`true`。其他属性的描述与[创建标量UDF](#create-a-scalar-udf)中的描述相同。

#### 创建UDTF

运行以下命令来创建您在上面示例中编译的UDTF：

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

PROPERTIES中参数的描述与[创建标量UDF](#create-a-scalar-udf)中的描述相同。

### 步骤 7：使用UDF

创建UDF后，您可以根据业务需求对其进行测试和使用。

#### 使用标量UDF

运行以下命令来使用您在上面示例中创建的标量UDF：

```SQL
SELECT MY_UDF_JSON_GET('{"key":"{\\"in\\":2}"}', '$.key.in');
```

#### 使用UDAF

运行以下命令来使用您在上面示例中创建的UDAF：

```SQL
SELECT MY_SUM_INT(col1);
```

#### 使用UDWF

运行以下命令来使用您在上面示例中创建的UDWF：

```SQL
SELECT MY_WINDOW_SUM_INT(intcol) 
            OVER (PARTITION BY intcol2
                  ORDER BY intcol3
                  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
FROM test_basic;
```

#### 使用UDTF

运行以下命令来使用您在上面示例中创建的UDTF：

```Plain
-- 假设您有名为t1的表，其列a、b和c1的信息如下：
SELECT t1.a,t1.b,t1.c1 FROM t1;
> output:
1,2.1,"hello world"
2,2.2,"hello UDTF."

-- 运行MY_UDF_SPLIT()函数。
SELECT t1.a,t1.b, MY_UDF_SPLIT FROM t1, MY_UDF_SPLIT(t1.c1); 
> output:
1,2.1,"hello"
1,2.1,"world"
2,2.2,"hello"
2,2.2,"UDTF."
```

> **注意**
>
> - 上述代码片段中的第一个`MY_UDF_SPLIT`是由第二个`MY_UDF_SPLIT`(函数)返回的列的别名。
> - 您不能使用`AS t2(f1)`指定要返回的表及其列的别名。

## 查看UDF

运行以下命令来查询UDF：

```SQL
SHOW [GLOBAL] FUNCTIONS;
```

有关更多信息，请参见[SHOW FUNCTIONS](../sql-statements/data-definition/SHOW_FUNCTIONS.md)。

## 删除UDF

运行以下命令来删除UDF：

```SQL
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

有关更多信息，请参见[DROP FUNCTION](../sql-statements/data-definition/DROP_FUNCTION.md)。

## SQL数据类型和Java数据类型之间的映射

| SQL类型       | Java类型         |
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

在StarRocks集群中每个Java虚拟机（JVM）的**be/conf/hadoop_env.sh**文件中配置以下环境变量以控制内存使用。您也可以在文件中配置其他参数。

```Bash
export LIBHDFS_OPTS="-Xloggc:$STARROCKS_HOME/log/be.gc.log -server"
```

## 常见问题解答

我在创建UDF时可以使用静态变量吗？不同UDF的静态变量是否相互影响？

是的，您可以在编译UDF时使用静态变量。不同UDF的静态变量是相互隔离的，即使UDF具有相同名称的类，它们也不会相互影响。