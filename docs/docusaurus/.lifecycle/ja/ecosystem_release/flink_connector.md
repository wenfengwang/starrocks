---
displayed_sidebar: "Japanese"
---

# Flinkコネクタ

## **通知**

**ユーザーガイド：**

- [Flinkコネクタを使用してデータをStarRocksにロードする方法](../loading/Flink-connector-starrocks.md)
- [Flinkコネクタを使用してStarRocksからデータを読み取る方法](../unloading/Flink_connector.md)

**ソースコード：** [starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JARファイルの命名形式:**

- Flink 1.15以降：`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- Flink 1.15より前：`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**JARファイルを取得する方法:**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)からFlinkコネクタのJARファイルを直接ダウンロードします。
- Mavenプロジェクトの`pom.xml`ファイルにFlinkコネクタを依存関係として追加し、ダウンロードします。詳細な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。
- ソースコードをコンパイルしてFlinkコネクタのJARファイルに変換します。詳細な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。

**バージョン要件:**

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
| --------- | ------------------------ | ------------- | ---- | --------- |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 以降      | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 以降      | 8    | 2.11,2.12 |

> **注意**
>
> 一般的に、Flinkコネクタの最新バージョンは、Flinkの直近3バージョンとの互換性を維持します。

## **リリースノート**

### 1.2

**1.2.8**

このリリースにはいくつかの改善とバグ修正が含まれています。注目すべき変更点は以下の通りです：

- Flink 1.16および1.17のサポート
- シンクが確実な一度限りのセマンティクスを確実にするために`sink.label-prefix`を設定することを推奨します。詳細な手順については、[Exactly Once](../loading/Flink-connector-starrocks.md#exactly-once)を参照してください。

**改善点**

- 少なくとも一度の実行を保証するために、Stream Loadトランザクションインターフェイスの使用を設定するサポート [#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- シンクV1のリトライメトリクスを追加 [#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- EXISTING_JOB_STATUSがFINISHEDの場合はgetLabelStateを行う必要がない [#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- シンクV1の不要なスタックトレースログを削除 [#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [リファクタ] StarRocksSinkManagerV2をstream-load-sdkに移動 [#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- ユーザーが明示的に指定した`sink.properties.columns`パラメータの代わりに、Flinkテーブルのスキーマに基づいて部分更新を自動的に検出するサポート [#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [リファクタ] probeTransactionStreamLoadをstream-load-sdkに移動 [#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- stream-load-sdkのgit-commit-id-pluginを追加 [#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- DefaultStreamLoader#closeに対してinfoログを使用するサポート [#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 依存関係のない状態でstream-load-sdkのJARファイルを生成するサポート [#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- stream-load-sdkでfastjsonをjacksonで置き換えるサポート [#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- update_beforeレコードを処理するサポート [#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- ファイルにApacheライセンスを追加 [#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- stream-load-sdkで例外を取得するサポート [#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- `strip_outer_array`および`ignore_json_size`をデフォルトで有効にするサポート [#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- Flinkジョブが復元され、シンクのセマンティクスが一度限りの場合に、残留するトランザクションをクリーンアップしようとします [#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- リトライが失敗した後に最初の例外を返します [#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**バグ修正**

- StarRocksStreamLoadVisitorのタイポを修正 [#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- fastjsonのクラスローダーリークを修正 [#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**テスト**

- KafkaからStarRocksへのロードのためのテストフレームワークを追加 [#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**ドキュメント**

- ドキュメントをリファクタリング [#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- シンクのためのドキュメントを改善 [#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- シンクのためのDataStream APIの例を追加 [#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)