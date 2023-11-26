---
displayed_sidebar: "Japanese"
---

# Flinkコネクタ

## **通知**

**ユーザーガイド：**

- [Flinkコネクタを使用してStarRocksにデータをロードする方法](../loading/Flink-connector-starrocks.md)
- [Flinkコネクタを使用してStarRocksからデータを読み取る方法](../unloading/Flink_connector.md)

**ソースコード：** [starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JARファイルの命名形式：**

- Flink 1.15以降：`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- Flink 1.15より前：`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**JARファイルの取得方法：**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)から直接FlinkコネクタのJARファイルをダウンロードします。
- Mavenプロジェクトの`pom.xml`ファイルにFlinkコネクタを依存関係として追加し、ダウンロードします。詳細な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。
- ソースコードをコンパイルしてFlinkコネクタのJARファイルを作成します。詳細な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。

**バージョン要件：**

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
| --------- | ------------------------ | ------------- | ---- | --------- |
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1以降       | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1以降       | 8    | 2.11,2.12 |

> **注意**
>
> 一般的に、Flinkコネクタの最新バージョンは、最新の3つのFlinkバージョンとの互換性を維持します。

## **リリースノート**

### 1.2

**1.2.8**

このリリースには、いくつかの改善とバグ修正が含まれています。主な変更点は以下の通りです：

- Flink 1.16と1.17のサポート。
- シンクが正確に一度だけのセマンティクスを保証するように設定されている場合、`sink.label-prefix`を設定することを推奨します。詳細な手順については、[Exactly Once](../loading/Flink-connector-starrocks.md#exactly-once)を参照してください。

**改善点**

- 最低一度の保証のためにStream Loadトランザクションインターフェースの使用を設定できるようにサポートしました。[#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- シンクV1のリトライメトリクスを追加しました。[#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- EXISTING_JOB_STATUSがFINISHEDの場合、getLabelStateは不要です。[#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- シンクV1の不要なスタックトレースログを削除しました。[#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [リファクタリング] StarRocksSinkManagerV2をstream-load-sdkに移動しました。[#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- ユーザーが明示的に指定した`sink.properties.columns`パラメータではなく、Flinkテーブルのスキーマに基づいて部分更新を自動的に検出します。[#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [リファクタリング] probeTransactionStreamLoadをstream-load-sdkに移動しました。[#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- stream-load-sdkにgit-commit-id-pluginを追加しました。[#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- DefaultStreamLoader#closeにinfoログを使用します。[#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 依存関係のないstream-load-sdk JARファイルを生成するサポートを追加しました。[#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- stream-load-sdkでfastjsonをjacksonに置き換えました。[#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- update_beforeレコードの処理をサポートしました。[#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- ファイルにApacheライセンスを追加しました。[#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- stream-load-sdkで例外を取得するサポートを追加しました。[#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- `strip_outer_array`と`ignore_json_size`をデフォルトで有効にしました。[#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- Flinkジョブが復元され、シンクセマンティクスが正確に一度だけの場合、残っているトランザクションをクリーンアップしようとします。[#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- リトライが失敗した後に最初の例外を返します。[#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**バグ修正**

- StarRocksStreamLoadVisitorのタイポを修正しました。[#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- fastjsonのクラスローダーリークを修正しました。[#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**テスト**

- KafkaからStarRocksへのロードのためのテストフレームワークを追加しました。[#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**ドキュメント**

- ドキュメントをリファクタリングしました。[#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- シンクのドキュメントを改善しました。[#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- シンクのためのDataStream APIの例を追加しました。[#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)
