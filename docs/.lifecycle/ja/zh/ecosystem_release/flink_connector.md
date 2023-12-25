---
displayed_sidebar: Chinese
---

# Flink connector のリリースノート

## リリースノート

**使用文書：**

- [Flink connector を使用してデータを StarRocks にインポートする](../loading/Flink-connector-starrocks.md)
- [Flink connector を使用して StarRocks からデータを読み取る](../unloading/Flink_connector.md)

**ソースコードダウンロード先：**[starrocks-connector-for-apache-flink](https://github.com/StarRocks/starrocks-connector-for-apache-flink)

**JAR ファイル命名規則：**

- Flink 1.15 以降：`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar`
- Flink 1.15 以前：`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar`

**JAR ファイルの取得方法：**

以下の方法で Flink connector の JAR ファイルを取得できます：

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) からコンパイル済みの JAR ファイルを直接ダウンロードします。
- Maven プロジェクトの pom ファイルに Flink connector を依存関係として追加し、ダウンロードします。詳細は[使用文書](../loading/Flink-connector-starrocks.md)を参照してください。
- ソースコードを手動でコンパイルして JAR ファイルを作成します。詳細は[使用文書](../loading/Flink-connector-starrocks.md)を参照してください。

**バージョン要件：**

| コネクタ | Flink       | StarRocks  | Java | Scala      |
| --------- | ----------- | ---------- | ---- | ---------- |
| 1.2.9 | 1.15 ～ 1.18 | 2.1 以上 | 8 | 2.11、2.12 |
| 1.2.8     | 1.13 ～ 1.17 | 2.1 以上 | 8    | 2.11、2.12 |
| 1.2.7     | 1.11 ～ 1.15 | 2.1 以上 | 8    | 2.11、2.12 |

> **注意**
>
> 最新バージョンの Flink connector は、最新の3つの Flink バージョンのみをサポートしています。

## リリース履歴

### 1.2

#### 1.2.9

このリリースには、以下の新機能とバグ修正が含まれています。特筆すべき変更点として、Flink connector は Flink CDC 3.0 と統合され、CDC データソース（例：MySQL、Kafka）から StarRocks へのストリーミング ELT パイプラインを簡単に構築できるようになりました。詳細は [Flink CDC 同期（schema change 対応）](../loading/Flink-connector-starrocks.md#flink-cdc同步支持-schema-change) を参照してください。

**新機能**

- Flink CDC 3.0 をサポートするための catalog を実装しました。[#295](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/295)
- [FLP-191](https://cwiki.apache.org/confluence/display/FLINK/FLIP-191%3A+Extend+unified+Sink+interface+to+support+small+file+compaction) に記載されている新しい Sink API を実装し、[Flink CDC 3.0](https://github.com/ververica/flink-cdc-connectors/issues/2600) をサポートしました。[#301](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/301)
- Flink 1.18 をサポートしました。[#305](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/305)

**バグ修正**

- 誤解を招くスレッド名とログを修正しました。[#290](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/290)
- 複数のテーブルに書き込む際の誤った stream-load-sdk 設定を修正しました。[#298](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/298)

#### 1.2.8

このリリースには、以下の機能の最適化とバグ修正が含まれています。特に重要な最適化は以下の通りです：

- Flink 1.16 および 1.17 をサポートしました。
- Sink のセマンティクスを exactly-once に設定する場合は、`sink.label-prefix` の設定を推奨します。使用方法は [Exactly Once](../loading/Flink-connector-starrocks.md#exactly-once) を参照してください。

**機能最適化**

- at-least-once セマンティクスを実現するために Stream Load トランザクションインターフェースを使用するかどうかを設定できるようにしました。[#228](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/228)
- sink バージョン V1 にリトライ指標を追加しました。[#229](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/229)
- EXISTING_JOB_STATUS が FINISHED の場合は getLabelState を実行する必要がないようにしました。[#231](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/231)
- sink バージョン V1 から不要なスタックトレースログを削除しました。[#232](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/232)
- [リファクタリング] StarRocksSinkManagerV2 を stream-load-sdk に移動しました。[#233](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/233)
- Flink のテーブル構造に基づいて、データインポートが部分的な列の更新のみかどうかを自動的に判断し、ユーザーが `sink.properties.columns` パラメータを明示的に指定する必要がないようにしました。[#235](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/235)
- [リファクタリング] probeTransactionStreamLoad を stream-load-sdk に移動しました。 [#240](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/240)
- stream-load-sdk に git-commit-id-plugin を追加しました。[#242](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/242)
- info レベルのログに DefaultStreamLoader#close を記録しました。[#243](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/243)
- stream-load-sdk で依存関係を含まない jar を生成できるようにしました。[#245](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/245)
- stream-load-sdk で fastjson の代わりに jackson を使用するようにしました。[#247](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/247)
- update_before レコードの処理をサポートしました。[#250](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/250)
- ファイルに Apache ライセンスを追加しました。[#251](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/251)
- stream-load-sdk から返される例外情報を取得できるようにしました。[#252](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/252)
- `strip_outer_array` と `ignore_json_size` をデフォルトで有効にしました。[#259](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/259)
- sink のセマンティクスが exactly-once の場合、Flink ジョブが復旧した後、Flink connector はチェックポイントに含まれていない StarRocks の未完了トランザクションをクリーンアップしようとします。[#271](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/271)
- リトライに失敗した後、最初の例外情報を返します。[#279](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/279)

**バグ修正**

- StarRocksStreamLoadVisitor のスペルミスを修正しました。[#230](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/230)
- fastjson classloader のリーク問題を修正しました。[#260](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/260)

**テスト**

- Kafka から StarRocks へのインポートテストフレームワークを追加しました。[#249](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/249)

**ドキュメント**

- ドキュメントをリファクタリングしました。[#262](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/262)
- sink 機能のドキュメントを改善しました。[#268](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/268) [#275](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/275)
- DataStream API を呼び出して sink 機能を実装する方法の例を追加しました。[#253](https://github.com/StarRocks/starrocks-connector-for-apache-flink/pull/253)
