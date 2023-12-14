---
displayed_sidebar: "Chinese"
---

# Kafka 连接器

## 通知

**用户指南:** [使用 Kafka 连接器加载数据](../loading/Kafka-connector-starrocks.md)

**压缩文件的命名格式:** `starrocks-kafka-connector-${connector_version}.tar.gz`

**压缩文件的下载链接:** [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)

**版本要求:**

| Kafka 连接器 | StarRocks | Java |
| --------------- | --------- | ---- |
| 1.0.0           | 2.1 及更高版本 | 8    |

## 发布说明

### 1.0

**1.0.0**

发布日期: 2023年6月25日

**功能**

- 支持加载 CSV、JSON、Avro 和 Protobuf 数据。
- 支持从自管 Apache Kafka 集群或 Confluent 云加载数据。