---
displayed_sidebar: Chinese
---

# Kafka コネクタのリリースノート

## リリース説明

**使用ドキュメント:** [Kafka コネクタを使用してデータをインポートする](../loading/Kafka-connector-starrocks.md)

**圧縮ファイルの命名規則：**`starrocks-kafka-connector-${connector_version}.tar.gz`

**圧縮ファイルのダウンロード先:**[starrocks-kafka-connector-1.0.0.tar.gz](https://releases.starrocks.io/starrocks/starrocks-kafka-connector-1.0.0.tar.gz)

**バージョン要件：**

| Kafka Connector  | StarRocks | Java |
| ---------------  | --------- | ---- |
| 1.0.0            | 2.1 以上   | 8    |

## リリース記録

### 1.0

**1.0.0**

リリース日：2023年6月25日

**新機能**

- CSV、JSON、Avro、Protobuf 形式のデータのインポートをサポート。
- 自己管理の Apache Kafka クラスターや Confluent cloud からのデータインポートをサポート。
