---
displayed_sidebar: English
---

# Flink 连接器

## 通知

**用户指南：**

- [使用 Flink 连接器将数据加载到 StarRocks 中](../loading/Flink-connector-starrocks.md)
- [使用 Flink 连接器从 StarRocks 读取数据](../unloading/Flink_connector.md)

**源代码：** [starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JAR文件的命名格式：**

- Flink 1.15 及更高版本： `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- 在 Flink 1.15 之前： `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**获取JAR文件的方法：**

- 直接从 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) 下载 Flink 连接器 JAR 文件。
- 将 Flink 连接器作为 Maven 项目的依赖项添加到 `pom.xml` 文件中并下载。有关具体说明，请参阅[用户指南](../loading/Flink-connector-starrocks.md#obtain-flink-connector)。
- 将源代码编译成 Flink 连接器 JAR 文件。有关具体说明，请参阅 [用户指南](../loading/Flink-connector-starrocks.md#obtain-flink-connector)。

**版本要求：**

| 连接器 | Flink                    | StarRocks（星石）     | Java | Scala     |
| --------- | ------------------------ | ------------- | ---- | --------- |
| 1.2.9 | 1.15,1.16,1.17,1.18 | 2.1 及更高版本| 8 | 2.11,2.12 |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 及更高版本 | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 及更高版本 | 8    | 2.11,2.12 |

> **注意**
>
> 通常情况下，最新版本的 Flink 连接器仅与 Flink 的最近三个版本保持兼容。

## 发行说明

### 1.2

#### 1.2.9

此版本包括一些功能和错误修复。值得注意的变化是，Flink 连接器集成了[Flink CDC 3.0](https://ververica.github.io/flink-cdc-connectors/master/content/overview/cdc-pipeline.html)，可以轻松构建从 CDC 源（如 MySQL 和 Kafka）到 StarRocks 的流式 ELT 流水线。详情请参见 [Flink CDC 同步](../loading/Flink-connector-starrocks.md#flink-cdc-synchronization-schema-change-supported)。

**功能**

- 实现目录以支持 Flink CDC 3.0。 [#295](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/295)
- 实现新的 sink API，以支持 Flink CDC 3.0。 [#301](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/301)
- 支持 Flink 1.18。 [#305](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/305)

**错误修复**

- 修复误导性的线程名称和日志。 [#290](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/290)
- 修复用于写入多个表的 stream-load-sdk 配置错误。 [#298](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/298)

#### 1.2.8

此版本包括一些改进和错误修复。值得注意的变化如下：

- 支持 Flink 1.16 和 1.17。
- 建议在配置接收器时设置 `sink.label-prefix` 以保证 exactly-once 语义。有关具体说明，请参阅 [正好一次](../loading/Flink-connector-starrocks.md#exactly-once)。

**改进**

- 支持配置是否使用 Stream Load 事务接口来保证至少一次。 [#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- 为接收器 V1 添加重试指标。 [#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- 当 FINISHED EXISTING_JOB_STATUS 时，无需 getLabelState。 [#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- 删除接收器 V1 的无用堆栈跟踪日志。 [#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [重构] 将 StarRocksSinkManagerV2 迁移至 stream-load-sdk。 [#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- 根据 Flink 表的 schema 自动检测部分更新，而不是 `sink.properties.columns` 用户明确指定的参数。 [#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [重构] 将 probeTransactionStreamLoad 移动到 stream-load-sdk。 [#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- 为 stream-load-sdk 添加 git-commit-id-plugin。 [#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- 使用 DefaultStreamLoader#close 的信息日志。 [#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 支持生成不带依赖的 stream-load-sdk JAR 文件。 [#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- 在 stream-load-sdk 中将 fastjson 替换为 jackson。 [#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- 支持处理update_before记录。 [#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- 将 Apache 许可证添加到文件中。 [#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- 支持在 stream-load-sdk 中获取异常。 [#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- 启用 `strip_outer_array` ，默认 `ignore_json_size` 。 [#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- 尝试在 Flink 作业恢复且接收器语义恰好为一次时清理延迟事务。 [#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- 重试失败后返回第一个异常。 [#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**错误修复**

- 修复 StarRocksStreamLoadVisitor 中的拼写错误。 [#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- 修复 fastjson 类加载器泄漏。 [#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**测试**

- 添加从 Kafka 加载到 StarRocks 的测试框架。 [#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**文档**

- 重构文档。 [#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- 改进接收器的文档。 [#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- 为接收器添加 DataStream API 示例。 [#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)
