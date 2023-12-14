---
displayed_sidebar: "Chinese"
---

# Spark连接器

## **通知**

**用户指南:**

- [使用Spark连接器加载数据到StarRocks](../loading/Spark-connector-starrocks.md)
- [使用Spark连接器从StarRocks读取数据](../unloading/Spark_connector.md)

**源代码**: [starrocks-connector-for-apache-spark](https://github.com/StarRocks/starrocks-connector-for-apache-spark)

**JAR文件命名格式**: `starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

**获取JAR文件的方法:**

- 直接从[Maven中央仓库](https://repo1.maven.org/maven2/com/starrocks)下载Spark连接器的JAR文件。
- 将Spark连接器作为Maven项目的依赖项添加到`pom.xml`文件中并下载。具体操作，请参阅[用户指南](../loading/Spark-connector-starrocks.md#obtain-spark-connector)。
- 将源代码编译成Spark连接器的JAR文件。具体操作，请参阅[用户指南](../loading/Spark-connector-starrocks.md#obtain-spark-connector)。

**版本要求:**

| Spark连接器 | Spark            | StarRocks     | Java | Scala |
| -------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3或3.4   | 2.5及更高版本 | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3或3.4   | 2.5及更高版本 | 8    | 2.12  |

## **发布说明**

### 1.1

**1.1.1**

此版本主要包括用于将数据加载到StarRocks的一些功能和改进。

> **注意**
>
> 当将Spark连接器升级到此版本时，请注意一些更改。详情请参阅[升级Spark连接器](../loading/Spark-connector-starrocks.md#upgrade-from-version-110-to-111)。

**功能**

- Sink支持重试。[#61](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/61)
- 支持加载数据到BITMAP和HLL列。[#67](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/67)
- 支持加载ARRAY类型数据。[#74](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/74)
- 支持根据缓冲行数刷新。[#78](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/78)

**改进**

- 移除无用的依赖项，使Spark连接器的JAR文件变得轻量级。[#55](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/55) [#57](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/57)
- 用jackson替换fastjson。[#58](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/58)
- 添加缺失的Apache许可证头。[#60](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/60)
- 不要在Spark连接器的JAR文件中打包MySQL JDBC驱动程序。[#63](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/63)
- 支持配置时区参数，与Spark Java8 API日期时间兼容。[#64](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/64)
- 优化行字符串转换器，减少CPU成本。[#68](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/68)
- `starrocks.fe.http.url`参数支持添加http scheme。[#71](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/71)
- 实现接口BatchWrite#useCommitCoordinator在DataBricks 13.1上运行。[#79](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/79)
- 在错误日志中添加检查特权和参数的提示。[#81](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/81)

**错误修复**

- 解析CSV相关参数`column_seperator`和`row_delimiter`中的转义字符。[#85](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/85)

**文档**

- 重构文档。[#66](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/66)
- 添加加载数据到BITMAP和HLL列的示例。[#70](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/70)
- 添加用Python编写的Spark应用程序示例。[#72](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/72)
- 添加加载ARRAY类型数据的示例。[#75](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/75)
- 添加在主键表上执行部分更新和条件更新的示例。[#80](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/80)

**1.1.0**

**功能**

- 支持将数据加载到StarRocks。

### 1.0

**功能**

- 支持从StarRocks卸载数据。