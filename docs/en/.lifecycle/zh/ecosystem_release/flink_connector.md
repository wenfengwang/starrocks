---
displayed_sidebar: English
---

# Flink 连接器

## 通知

**用户指南：**

- [使用 Flink 连接器将数据加载到 StarRocks](../loading/Flink-connector-starrocks.md)
- [使用 Flink 连接器从 StarRocks 读取数据](../unloading/Flink_connector.md)

**源代码：** [starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JAR 文件的命名格式：**

- Flink 1.15 及更高版本：`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- Flink 1.15 之前：`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**获取 JAR 文件的方法：**

- 直接从 [Maven 中央仓库](https://repo1.maven.org/maven2/com/starrocks)下载 Flink 连接器 JAR 文件。
- 将 Flink 连接器作为 Maven 项目的 `pom.xml` 文件中的依赖项添加并下载。具体说明，请参阅[用户指南](../loading/Flink-connector-starrocks.md#obtain-flink-connector)。
- 编译源代码生成 Flink 连接器 JAR 文件。具体说明，请参阅[用户指南](../loading/Flink-connector-starrocks.md#obtain-flink-connector)。

**版本要求：**

| 连接器 | Flink | StarRocks | Java | Scala |
| --- | --- | --- | --- | --- |
| 1.2.9 | 1.15, 1.16, 1.17, 1.18 | 2.1 及更高版本 | 8 | 2.11, 2.12 |
| 1.2.8 | 1.13, 1.14, 1.15, 1.16, 1.17 | 2.1 及更高版本 | 8 | 2.11, 2.12 |
| 1.2.7 | 1.11, 1.12, 1.13, 1.14, 1.15 | 2.1 及更高版本 | 8 | 2.11, 2.12 |

> **注意**
> 通常，最新版本的 Flink 连接器仅与 Flink 的最新三个版本保持兼容。

## 发行说明

### 1.2

#### 1.2.9

此版本包括一些新特性和错误修复。显著的变化是 Flink 连接器与 [Flink CDC 3.0](https://ververica.github.io/flink-cdc-connectors/master/content/overview/cdc-pipeline.html) 集成，以便轻松构建从 CDC 源（如 MySQL 和 Kafka）到 StarRocks 的流式 ELT 管道。详细信息请参见[Flink CDC 同步](../loading/Flink-connector-starrocks.md#flink-cdc-synchronization-schema-change-supported)。

**新特性**

- 实现目录以支持 Flink CDC 3.0。[#295](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/295)
- 在 [FLIP-191](https://cwiki.apache.org/confluence/display/FLINK/FLIP-191%3A+Extend+unified+Sink+interface+to+support+small+file+compaction) 中实现新的 Sink API 以支持 Flink CDC 3.0。[#301](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/301)
- 支持 Flink 1.18。[#305](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/305)

**Bug 修复**

- 修复误导性的线程名称和日志。[#290](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/290)
- 修复错误的 stream-load-sdk 配置，用于写入多个表。[#298](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/298)

#### 1.2.8

此版本包括一些改进和错误修复。显著的变化包括：

- 支持 Flink 1.16 和 1.17。
- 建议在配置 Sink 时设置 `sink.label-prefix` 以保证精确一次（exactly-once）语义。具体说明请参见[精确一次](../loading/Flink-connector-starrocks.md#exactly-once)。

**改进**

- 支持配置是否使用 Stream Load 事务接口以保证至少一次（at-least-once）语义。[#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- 为 Sink V1 添加重试指标。[#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- 当 **EXISTING_JOB_STATUS** 为 **FINISHED** 时，无需获取 LabelState。[#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- 移除 Sink V1 中无用的堆栈跟踪日志。[#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [重构] 将 StarRocksSinkManagerV2 移至 stream-load-sdk。[#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- 根据 Flink 表的 schema 自动检测部分更新，而不是用户显式指定的 `sink.properties.columns` 参数。[#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [重构] 将 probeTransactionStreamLoad 移至 stream-load-sdk。[#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- 为 stream-load-sdk 添加 git-commit-id-plugin。[#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- 使用 info 日志记录 DefaultStreamLoader#close。[#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 支持生成无依赖的 stream-load-sdk JAR 文件。[#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- 在 stream-load-sdk 中将 fastjson 替换为 jackson。[#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- 支持处理 update_before 记录。[#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- 将 Apache 许可证添加到文件中。[#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- 支持在 stream-load-sdk 中获取异常。[#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- 默认启用 `strip_outer_array` 和 `ignore_json_size`。[#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- 尝试在 Flink 作业恢复时清理悬挂的事务，且 Sink 语义为精确一次（exactly-once）。[#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- 重试失败后返回第一个异常。[#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**Bug 修复**

- 修复 StarRocksStreamLoadVisitor 中的拼写错误。[#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- 修复 fastjson 类加载器泄漏问题。[#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**测试**

- 添加从 Kafka 到 StarRocks 的加载测试框架。[#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**文档**

- 重构文档。[#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- 改进 Sink 相关文档。[#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- 添加 Sink 的 DataStream API 示例。[#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)