---
displayed_sidebar: "Japanese"
---

# Flinkコネクタ

## **通知**

**ユーザーガイド:**

- [Flinkコネクタを使用してStarRocksにデータをロードする方法](../loading/Flink-connector-starrocks.md)
- [Flinkコネクタを使用してStarRocksからデータを読み取る方法](../unloading/Flink_connector.md)

**ソースコード:** [starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JARファイルの命名形式:**

- Flink 1.15以降: `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- Flink 1.15より前: `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**JARファイルを取得する方法:**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)からFlinkコネクタJARファイルを直接ダウンロードします。
- Mavenプロジェクトの`pom.xml`ファイルにFlinkコネクタを依存関係として追加し、ダウンロードします。詳細な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。
- ソースコードをFlinkコネクタJARファイルにコンパイルします。詳細な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。

**バージョン要件:**

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
| --------- | ------------------------ | ------------- | ---- | --------- |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 以降      | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 以降      | 8    | 2.11,2.12 |

> **注意**
>
> 一般的に、Flinkコネクタの最新バージョンは、Flinkの直近の3つのバージョンとの互換性を維持します。

## **リリースノート**

### 1.2

**1.2.8**

このリリースには、いくつかの改善とバグ修正が含まれています。注目すべき変更点は次のとおりです：

- Flink 1.16および1.17をサポートします。
- シンクが正確に一度だけのセマンティクスを保証するように構成されている場合、`sink.label-prefix`を設定することを推奨します。詳細な手順については、[Exacty Once](../loading/Flink-connector-starrocks.md#exactly-once)を参照してください。

**改善**

- 少なくとも一度の保証のためにStream Loadトランザクションインターフェースを使用するかどうかを構成するサポート。[#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- sink V1のためのリトライメトリクスを追加します。[#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- EXISTING_JOB_STATUSがFINISHEDの場合は、getLabelStateは不要です。[#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- sink V1のための無駄なスタックトレースログを削除します。[#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- StarRocksSinkManagerV2をstream-load-sdkに移動します。[#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- ユーザーによって明示的に指定された`sink.properties.columns`パラメータの代わりに、Flinkテーブルのスキーマに基づいて部分更新を自動検出します。[#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- probeTransactionStreamLoadをstream-load-sdkに移動します。[#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- stream-load-sdkのgit-commit-id-pluginを追加します。[#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- DefaultStreamLoader#closeにinfoログを使用します。[#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 依存関係なしでstream-load-sdk JARファイルを生成するサポートを追加します。[#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- stream-load-sdkでfastjsonをjacksonに置き換えます。[#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- update_beforeレコードを処理するサポートを追加します。[#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- ファイルにApacheライセンスを追加します。[#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- stream-load-sdkで例外を取得するサポートを追加します。[#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- `strip_outer_array`および`ignore_json_size`をデフォルトで有効にします。[#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- Flinkジョブが復元され、シンクセマンティクスが正確に一度だけである場合に、残留トランザクションをクリーンアップしようとします。[#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- リトライが失敗した後に最初の例外を返します。[#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**バグ修正**

- StarRocksStreamLoadVisitorの誤記を修正します。[#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- fastjsonのクラスローダーリークを修正します。[#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**テスト**

- KafkaからStarRocksに読み込むためのテストフレームワークを追加します。[#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**ドキュメント**

- ドキュメントをリファクタリングします。[#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- シンクのためのドキュメントを改善します。[#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- シンクのためのDataStream APIの例を追加します。[#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)