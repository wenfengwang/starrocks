---
displayed_sidebar: "Japanese"
---

# Kafkaコネクタ

## 通知

**ユーザーガイド:** [Kafkaコネクタを使用してデータをロードする](../loading/Kafka-connector-starrocks.md)

**圧縮ファイルの命名形式:** `starrocks-kafka-connector-${connector_version}.tar.gz`

**圧縮ファイルのダウンロードリンク:** [starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)

**バージョン要件:**

| Kafkaコネクタ | StarRocks | Java |
| --------------- | --------- | ---- |
| 1.0.0           | 2.1以降 | 8    |

## リリースノート

### 1.0

**1.0.0**

リリース日: 2023年6月25日

**機能**

- CSV、JSON、Avro、Protobufデータの読み込みをサポート
- セルフマネージドのApache KafkaクラスタやConfluent Cloudからのデータ読み込みをサポート