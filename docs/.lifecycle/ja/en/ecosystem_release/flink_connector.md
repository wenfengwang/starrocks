---
displayed_sidebar: English
---

# Flinkコネクタ

## 通知

**ユーザーガイド:**

- [Flinkコネクタを使用してStarRocksにデータをロードする](../loading/Flink-connector-starrocks.md)
- [Flinkコネクタを使用してStarRocksからデータを読み取る](../unloading/Flink_connector.md)

**ソースコード:** [starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JARファイルの命名フォーマット:**

- Flink 1.15以降: `flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- Flink 1.15より前: `flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**JARファイルの取得方法:**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks)からFlinkコネクタのJARファイルを直接ダウンロードする。
- Mavenプロジェクトの`pom.xml`ファイルにFlinkコネクタを依存関係として追加し、ダウンロードする。具体的な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。
- ソースコードをコンパイルしてFlinkコネクタのJARファイルを作成する。具体的な手順については、[ユーザーガイド](../loading/Flink-connector-starrocks.md#obtain-flink-connector)を参照してください。

**バージョン要件:**

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
| --------- | ------------------------ | ------------- | ---- | --------- |
| 1.2.9 | 1.15, 1.16, 1.17, 1.18 | 2.1以降 | 8 | 2.11, 2.12 |
| 1.2.8     | 1.13, 1.14, 1.15, 1.16, 1.17 | 2.1以降 | 8    | 2.11, 2.12 |
| 1.2.7     | 1.11, 1.12, 1.13, 1.14, 1.15 | 2.1以降 | 8    | 2.11, 2.12 |

> **注意**
>
> 一般的に、最新バージョンのFlinkコネクタはFlinkの最新の3バージョンのみとの互換性を維持します。

## リリースノート

### 1.2

#### 1.2.9

このリリースには、いくつかの機能とバグ修正が含まれています。特筆すべき変更は、Flinkコネクタが[Flink CDC 3.0](https://ververica.github.io/flink-cdc-connectors/master/content/overview/cdc-pipeline.html)と統合され、CDCソース（MySQLやKafkaなど）からStarRocksへのストリーミングELTパイプラインを簡単に構築できるようになったことです。詳細は[Flink CDC同期](../loading/Flink-connector-starrocks.md#flink-cdc-synchronization-schema-change-supported)をご覧ください。

**機能**

- Flink CDC 3.0をサポートするカタログを実装。[#295](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/295)
- [FLIP-191](https://cwiki.apache.org/confluence/display/FLINK/FLIP-191%3A+Extend+unified+Sink+interface+to+support+small+file+compaction)における新しいシンクAPIを実装してFlink CDC 3.0をサポート。[#301](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/301)
- Flink 1.18をサポート。[#305](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/305)

**バグ修正**

- 誤解を招くスレッド名とログを修正。[#290](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/290)
- 複数のテーブルへの書き込みに使用される誤ったstream-load-sdk設定を修正。[#298](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/298)

#### 1.2.8

このリリースには、いくつかの改善とバグ修正が含まれています。注目すべき変更点は以下の通りです：

- Flink 1.16および1.17をサポート。
- シンクが正確に一度のセマンティクスを保証するように設定されている場合、`sink.label-prefix`を設定することを推奨します。具体的な手順については、[正確に一度](../loading/Flink-connector-starrocks.md#exactly-once)を参照してください。

**改善点**

- 少なくとも一度の保証をするためにStream Loadトランザクションインターフェースを使用するかどうかを設定するサポート。[#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- シンクV1のリトライメトリクスを追加。[#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- EXISTING_JOB_STATUSがFINISHEDの場合、getLabelStateを取得する必要はありません。[#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- シンクV1の不要なスタックトレースログを削除。[#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [リファクタリング] StarRocksSinkManagerV2をstream-load-sdkに移動。[#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- ユーザーが明示的に指定した`sink.properties.columns`ではなく、Flinkテーブルのスキーマに基づいて部分更新を自動的に検出する。[#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [リファクタリング] probeTransactionStreamLoadをstream-load-sdkに移動。[#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- stream-load-sdkにgit-commit-id-pluginを追加。[#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- DefaultStreamLoader#closeの情報ログを使用。[#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- 依存関係なしでstream-load-sdk JARファイルを生成するサポート。[#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- stream-load-sdkでfastjsonをjacksonに置き換える。[#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- update_beforeレコードの処理をサポート。[#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- ファイルにApacheライセンスを追加。[#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- stream-load-sdkで例外を取得するサポート。[#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- デフォルトで`strip_outer_array`と`ignore_json_size`を有効にする。[#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- Flinkジョブが復元され、シンクのセマンティクスが正確に一度である場合、残留するトランザクションをクリーンアップする試み。[#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- リトライに失敗した後、最初の例外を返す。[#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**バグ修正**

- StarRocksStreamLoadVisitorの誤字を修正。[#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- fastjsonクラスローダーのリークを修正。[#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**テスト**

- KafkaからStarRocksへのロードのためのテストフレームワークを追加。[#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**ドキュメント**

- ドキュメントをリファクタリング。[#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- シンクのドキュメントを改善。[#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- シンク用のDataStream APIの例を追加。[#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)
