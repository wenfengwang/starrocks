---
displayed_sidebar: "Chinese"
---

# Flink 连接器

## **通知**

**用户指南：**

- [使用 Flink 连接器将数据加载到 StarRocks](../loading/Flink-connector-starrocks.md)
- [使用 Flink 连接器从 StarRocks 读取数据](../unloading/Flink_connector.md)

**源代码：** [apache-flink-for-starrocks-连接器](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JAR 文件命名格式：**

- Flink 1.15 及更高版本：`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- 早于 Flink 1.15 版本：`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**获取 JAR 文件的方法：**

- 直接从 [Maven 中央仓库](https://repo1.maven.org/maven2/com/starrocks) 下载 Flink 连接器的 JAR 文件。
- 将 Flink 连接器作为 Maven 项目的依赖项添加到 `pom.xml` 文件中并下载。有关具体说明，请参阅[用户指南](../loading/Flink-connector-starrocks.md#obtain-flink-connector)。
- 将源代码编译成 Flink 连接器的 JAR 文件。有关具体说明，请参阅[用户指南](../loading/Flink-connector-starrocks.md#obtain-flink-connector)。

**版本要求：**

| 连接器 | Flink                    | StarRocks     | Java | Scala     |
| ------ | ------------------------ | ------------- | ---- | --------- |
| 1.2.8  | 1.13,1.14,1.15,1.16,1.17 | 2.1 及更高版本 | 8    | 2.11,2.12 |
| 1.2.7  | 1.11,1.12,1.13,1.14,1.15 | 2.1 及更高版本 | 8    | 2.11,2.12 |

> **注意**
>
> 通常情况下，Flink 连接器的最新版本仅与 Flink 的最近三个版本保持兼容性。

## **发布说明**

### 1.2

**1.2.8**

此版本包含一些改进和 bug 修复。值得注意的变化如下：

- 支持 Flink 1.16 和 1.17。
- 建议在配置 sink 以保证精准一次语义时设置 `sink.label-prefix`。有关具体说明，请参阅[精准一次语义](../loading/Flink-connector-starrocks.md#exactly-once)。

**改进**

- 支持配置是否使用流加载事务接口以保证至少一次语义。[#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- 为 sink V1 添加重试指标。[#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- 当 EXISTING_JOB_STATUS 完成时，无需获取 getLabelState。[#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- 移除 sink V1 中无用的堆栈跟踪日志。[#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [重构] 将 StarRocksSinkManagerV2 移动到流加载 SDK。[#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- 根据 Flink 表的模式自动检测部分更新，而不是用户明确指定的 `sink.properties.columns` 参数。[#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [重构] 将 probeTransactionStreamLoad 移动到流加载 SDK。[#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- 为流加载 SDK 添加 git-commit-id-plugin。[#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- 为 DefaultStreamLoader#close 使用 info 日志。[#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 支持生成不带依赖项的流加载 SDK JAR 文件。[#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- 将 fastjson 替换为流加载 SDK 中的 jackson。[#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- 支持处理 update_before 记录。[#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- 将 Apache 许可证添加到文件中。[#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- 支持在流加载 SDK 中获取异常。[#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- 默认启用 `strip_outer_array` 和 `ignore_json_size`。[#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- 尝试清理残留事务，当 Flink 作业恢复且 sink 语义为精准一次时。[#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- 重试失败后返回第一个异常。[#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**Bug 修复**

- 修复 StarRocksStreamLoadVisitor 中的拼写错误。[#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- 修复 fastjson 类加载泄漏。[#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**测试**

- 为从 Kafka 加载到 StarRocks 添加测试框架。[#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**文档**

- 重构文档。[#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- 改进 sink 的文档。[#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- 为 sink 添加 DataStream API 示例。[#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)