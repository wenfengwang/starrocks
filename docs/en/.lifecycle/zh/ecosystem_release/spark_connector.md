---
displayed_sidebar: English
---

# Spark 连接器

## **通知**

**用户指南：**

- [使用 Spark 连接器将数据加载到 StarRocks](../loading/Spark-connector-starrocks.md)
- [使用 Spark 连接器从 StarRocks 读取数据](../unloading/Spark_connector.md)

**源代码**：[starrocks-connector-for-apache-spark](https://github.com/StarRocks/starrocks-connector-for-apache-spark)

**JAR 文件命名格式**：`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

**获取 JAR 文件的方法：**

- 直接从 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) 下载 Spark 连接器 JAR 文件。
- 将 Spark 连接器添加为 Maven 项目的 `pom.xml` 文件中的依赖项并下载。有关具体说明，请参阅[用户指南](../loading/Spark-connector-starrocks.md#obtain-spark-connector)。
- 编译源代码以生成 Spark 连接器 JAR 文件。有关具体说明，请参阅[用户指南](../loading/Spark-connector-starrocks.md#obtain-spark-connector)。

**版本要求：**

|Spark 连接器|Spark|StarRocks|Java|Scala|
|---|---|---|---|---|
|1.1.1|3.2、3.3 或 3.4|2.5 及更高版本|8|2.12|
|1.1.0|3.2、3.3 或 3.4|2.5 及更高版本|8|2.12|

## **发布说明**

### 1.1

**1.1.1**

此版本主要包括一些用于将数据加载到 StarRocks 的功能和改进。

> **注意**
> 当您升级到此版本的 Spark 连接器时，请注意一些变更。详情请参阅[升级 Spark 连接器](../loading/Spark-connector-starrocks.md#upgrade-from-version-110-to-111)。

**特性**

- Sink 支持重试机制。[#61](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/61)
- 支持将数据加载到 BITMAP 和 HLL 列。[#67](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/67)
- 支持加载 ARRAY 类型数据。[#74](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/74)
- 支持根据缓冲行数刷新数据。[#78](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/78)

**改进**

- 移除无用依赖，使 Spark 连接器 JAR 文件更加轻量。[#55](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/55) [#57](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/57)
- 用 Jackson 替换 fastjson。[#58](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/58)
- 添加缺失的 Apache 许可证头部。[#60](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/60)
- 不再将 MySQL JDBC 驱动打包在 Spark 连接器 JAR 文件中。[#63](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/63)
- 支持配置时区参数，与 Spark Java8 API 日期时间兼容。[#64](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/64)
- 优化行字符串转换器，降低 CPU 消耗。[#68](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/68)
- `starrocks.fe.http.url` 参数支持添加 http 协议。[#71](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/71)
- 实现了接口 BatchWrite#useCommitCoordinator，以便在 DataBricks 13.1 上运行。[#79](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/79)
- 在错误日志中增加检查权限和参数的提示。[#81](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/81)

**Bug 修复**

- 解析 CSV 相关参数 `column_seperator` 和 `row_delimiter` 中的转义字符。[#85](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/85)

**文档**

- 重构文档。[#66](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/66)
- 添加加载数据到 BITMAP 和 HLL 列的示例。[#70](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/70)
- 添加 Python 编写的 Spark 应用程序示例。[#72](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/72)
- 添加加载 ARRAY 类型数据的示例。[#75](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/75)
- 添加对主键表执行部分更新和条件更新的示例。[#80](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/80)

**1.1.0**

**特性**

- 支持将数据加载到 StarRocks 中。

### 1.0

**特性**

- 支持从 StarRocks 卸载数据。