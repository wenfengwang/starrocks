---
displayed_sidebar: English
---

# Kafkaコネクタ

## 通知

**ユーザーガイド:** [Kafkaコネクタを使用してデータをロードする](../loading/Kafka-connector-starrocks.md)

**圧縮ファイルの命名形式:** `starrocks-kafka-connector-${connector_version}.tar.gz`

**圧縮ファイルのダウンロードリンク:** [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)

**バージョン要件:**

| Kafka Connector | StarRocks | Java |
| --------------- | --------- | ---- |
| 1.0.0           | 2.1 以降  | 8    |

## リリースノート

### 1.0

**1.0.0**

リリース日: 2023年6月25日

**機能**

- CSV、JSON、Avro、およびProtobufデータのロードをサポート。
- 自己管理型Apache KafkaクラスターやConfluent Cloudからのデータロードをサポート。
