---
displayed_sidebar: Chinese
---
# StarRocks バージョン 2.2

## 2.2.13

リリース日： 2023年4月6日

### 機能最適化

- 特定のシナリオでの `bitmap_contains()` 関数が大量のメモリを消費し、パフォーマンスが低下する問題を最適化しました。([#20616](https://github.com/StarRocks/starrocks/issues/20616))
- Compaction フレームワークが CPU リソースを消費する量を削減しました。([#11746](https://github.com/StarRocks/starrocks/issues/11746))

## 問題修正

以下の問題を修正しました：

- Stream Load ジョブを送信する際に誤った URL リクエストアドレスを提供した場合、FE が内部で停止し、HTTP リクエストを受信できなくなる問題。([#18468](https://github.com/StarRocks/starrocks/issues/18468))
- FE が統計情報を収集することでメモリ消費が過剰になり、OOM が発生する可能性がある問題。([#16331](https://github.com/StarRocks/starrocks/issues/16331))
- 一部のクエリでメモリ解放処理に問題があり、BE がクラッシュする問題。([#11395](https://github.com/StarRocks/starrocks/issues/11395))
- `TRUNCATE TABLE` 実行後に null ポインター問題が発生し、FE が再起動できなくなる問題。([#16773](https://github.com/StarRocks/starrocks/issues/16773))

## 2.2.10

リリース日： 2022年12月2日

### 機能最適化

- Routine Load のエラーメッセージを最適化しました。([#12203](https://github.com/StarRocks/starrocks/pull/12203))
- 論理演算子 `&&` をサポートしました。([#11819](https://github.com/StarRocks/starrocks/issues/11819))
- BE がクラッシュした際にクエリを迅速にキャンセルし、長時間のクエリタイムアウトによる停止を防ぐようにしました。([#12954](https://github.com/StarRocks/starrocks/pull/12954))
- FE の起動スクリプトを最適化し、Java のバージョンをチェックするようにしました。([#14094](https://github.com/StarRocks/starrocks/pull/14094))
- Primary Key での大量データ削除をサポートしました。([#4772](https://github.com/StarRocks/starrocks/issues/4772))

### 問題修正

以下の問題を修正しました：

- 複数のテーブルを合併 (union) するビューで、左側のノードに NULL 定数が存在する場合、BE がクラッシュする問題。([#13792](https://github.com/StarRocks/starrocks/pull/13792))
- Parquet ファイルと Hive テーブルの列の型が一致しない場合、クエリ中に BE がクラッシュする問題。([#8848](https://github.com/StarRocks/starrocks/issues/8848))
- 多すぎる OR 文が Planner に過剰な再帰を引き起こし、クエリタイムアウトを引き起こす問題。([#12788](https://github.com/StarRocks/starrocks/pull/12788))
- サブクエリに LIMIT がある場合、結果が正しくない可能性がある問題。([#12466](https://github.com/StarRocks/starrocks/pull/12466))
- ビューを作成する際に、ステートメントに引用符を含めることができない問題。([#13102](https://github.com/StarRocks/starrocks/pull/13102))

## 2.2.9

リリース日： 2022年11月15日

### 機能最適化

- `hive_partition_stats_sample_size` パラメータを追加し、統計情報を取得するパーティション数を制限し、パーティション数が多すぎて FE が Hive のメタデータを異常に取得するのを防ぎました。([#12700](https://github.com/StarRocks/starrocks/pull/12700))
- Elasticsearch 外部テーブルでカスタムタイムゾーンをサポートしました。([#12662](https://github.com/StarRocks/starrocks/pull/12662))

### 問題修正

以下の問題を修正しました：

- 外部テーブルのメタデータ同期の問題がノードのオフライン (Decommission) を停止させる問題。([#12369](https://github.com/StarRocks/starrocks/pull/12368))
- 列を追加する操作中にその列を削除すると、compaction がクラッシュする可能性がある問題。([#12907](https://github.com/StarRocks/starrocks/pull/12907))
- `SHOW CREATE VIEW` がコメントフィールドを表示しない問題。([#4163](https://github.com/StarRocks/starrocks/issues/4163))
- UDF にメモリリークが発生し、OOM になる可能性がある問題。([#12418](https://github.com/StarRocks/starrocks/pull/12418))
- Follower FE が保持するノードの生存状態 (alive) が `heartbeatRetryTimes` に依存しており、一部のシナリオで不正確である問題。新バージョンでは `HeartbeatResponse` に `aliveStatus` 属性を追加し、ノードの生存状態を表します。([#12481](https://github.com/StarRocks/starrocks/pull/12481))

### 挙動変更

Hive 外部テーブルの文字列のサポート長が 64 KB から 1 MB に拡張されました。長さが 1 MB を超える場合、クエリは Null として設定されます。([#12986](https://github.com/StarRocks/starrocks/pull/12986))

## 2.2.8

リリース日： 2022年10月17日

### 問題修正

以下の問題を修正しました：

- 初期化段階での式のエラーが BE のサービス停止を引き起こす可能性がある問題。([#11395](https://github.com/StarRocks/starrocks/pull/11395))
- 無効な JSON のインポートが BE のサービス停止を引き起こす可能性がある問題。([#10804](https://github.com/StarRocks/starrocks/issues/10804))
- Pipeline エンジンを有効にすると、並行書き込みにエラーが発生する問題。([#11451](https://github.com/StarRocks/starrocks/issues/11451))
- `ORDER BY NULL LIMIT` が BE のサービス停止を引き起こす問題。([#11648](https://github.com/StarRocks/starrocks/issues/11648))
- 外部テーブルの列の型と Parquet テーブルの型が一致しない場合に BE のサービス停止を引き起こす問題。([#11839](https://github.com/StarRocks/starrocks/issues/11839))

## 2.2.7

リリース日： 2022年9月23日

### 問題修正

以下の問題を修正しました：

- JSON データのインポート時にデータが失われる可能性がある問題。([#11054](https://github.com/StarRocks/starrocks/issues/11054))
- `SHOW FULL TABLES` の結果が誤っている問題。([#11126](https://github.com/StarRocks/starrocks/issues/11126))
- ビューの権限に関する問題で、以前のバージョンではベーステーブルとビューの両方の権限が必要でしたが、修正後はビューの権限のみでビューデータにアクセスできるようになりました。([#11290](https://github.com/StarRocks/starrocks/pull/11290))
- 複雑なクエリでの exists/in サブクエリのバグ。([#11415](https://github.com/StarRocks/starrocks/pull/11415))
- Hive で schema change を行った後に `REFRESH EXTERNAL TABLE` が失敗する問題。([#11406](https://github.com/StarRocks/starrocks/pull/11406))
- FE が bitmap インデックスの作成操作を再生する際にエラーが発生する可能性がある問題。([#11261](https://github.com/StarRocks/starrocks/pull/11261))

## 2.2.6

リリース日： 2022年9月14日

### 問題修正

以下の問題を修正しました：

- サブクエリに LIMIT がある場合、`order by...limit...offset...` の結果が正確でない問題。([#9698](https://github.com/StarRocks/starrocks/issues/9698))
- 大規模データの Partial update が BE のクラッシュを引き起こす問題。([#9809](https://github.com/StarRocks/starrocks/issues/9809))
- Bitmap が 2 GB を超えると、compaction がクラッシュする問題。([#11159](https://github.com/StarRocks/starrocks/pull/11159))
- `like()` および `regexp()` 関数で pattern が 16 KB を超えると使用できない問題。([#10364](https://github.com/StarRocks/starrocks/issues/10364))

### 挙動調整

- 結果に表示される `ARRAY<JSON>` の形式を変更し、出力結果ではエスケープ文字ではなくシングルクォートを使用します。([#10790](https://github.com/StarRocks/starrocks/issues/10790))

## 2.2.5

リリース日： 2022年8月18日

### 機能最適化

- Pipeline エンジンを有効にした後のシステムパフォーマンスを最適化しました。[#9580](https://github.com/StarRocks/starrocks/pull/9580)
- インデックスメタデータのメモリ統計の精度を最適化しました。[#9837](https://github.com/StarRocks/starrocks/pull/9837)

### 問題修正

以下の問題を修正しました：

- BE が Routine Load を実行する際に `get_partition_offset` で停止する可能性がある問題。[#9937](https://github.com/StarRocks/starrocks/pull/9937)
- 異なるクラスターが Broker Load を使用して同じ HDFS ファイルをインポートするとエラーが発生する問題。[#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

リリース日： 2022年8月3日

### 機能最適化

- Hive 外部テーブルが Schema Change の同期をサポートしました。[#9010](https://github.com/StarRocks/starrocks/pull/9010)
- Broker Load が Parquet ファイル内の ARRAY 型データのインポートをサポートしました。[#9131](https://github.com/StarRocks/starrocks/pull/9131)

### 問題修正

以下の問題を修正しました：

- Kerberos 認証を使用して Broker Load を実行する際に複数の keytab ファイルを使用できない問題。[#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)
- **stop_be.sh** を実行した後にすぐにプロセスを終了し、Supervisor がサービスを再起動するのに失敗する可能性がある問題。[#9175](https://github.com/StarRocks/starrocks/pull/9175)
- 不正確な Join Reorder 優先度により、Join フィールドで「Column cannot be resolved」というエラーが発生しました。[#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

リリース日： 2022年7月24日

### 問題修正

以下の問題を修正しました：

- リソースグループの削除プロセス中のエラーを修正しました。[#8036](https://github.com/StarRocks/starrocks/pull/8036)
- スレッドリソースが不足して Thrift server が終了する問題を修正しました。[#7974](https://github.com/StarRocks/starrocks/pull/7974)
- 特定のシナリオで CBO の join reorder が結果を出力できない問題を修正しました。[#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

リリース日： 2022年6月29日

### 機能最適化

- データベース間で UDF を使用できるようにしました。[#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- スキーマ変更 (Schema Change) などの内部処理の並行制御を最適化し、FE のメタデータへの負荷を軽減し、高並行性と大量データインポートのシナリオで発生しやすいインポートのバックログと遅延の問題を減少させました。[#6838](https://github.com/StarRocks/starrocks/pull/6838)

### 問題修正

以下の問題を修正しました：

- CTAS 実行時に新しいテーブルのレプリカ数が誤っていた問題を修正しました（`replication_num`）。[#7036](https://github.com/StarRocks/starrocks/pull/7036)
- ALTER ROUTINE LOAD 実行後にメタデータが失われる可能性がある問題を修正しました。[#7068](https://github.com/StarRocks/starrocks/pull/7068)
- Runtime filter がウィンドウにプッシュダウンされない問題を修正しました。[#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)
- Pipeline 内の潜在的なメモリリーク問題を修正しました。[#7295](https://github.com/StarRocks/starrocks/pull/7295)

- Routine Load タスクを停止するとデッドロックが発生する可能性がある問題を修正しました。[#6849](https://github.com/StarRocks/starrocks/pull/6849)

- いくつかのプロファイル統計情報を修正しました。[#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string 関数が JSON 配列を誤って処理する問題を修正しました。[#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

リリース日： 2022年6月2日

### 機能最適化

- ホットスポットコードのリファクタリングとロックの粒度を下げることでインポート性能を最適化し、テールレイテンシーを減少させました。[#6641](https://github.com/StarRocks/starrocks/pull/6641)
- FE の監査ログに各クエリが消費する BE マシンの CPU とメモリ情報を追加しました。[#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)
- 主キーモデルテーブルと更新モデルテーブルで JSON データ型を使用できるようにしました。[#6544](https://github.com/StarRocks/starrocks/pull/6544)
- ロックの粒度を下げ、BE のレポート (report) リクエストの重複を排除することで FE の負荷を減少させ、多数の BE をデプロイする際のレポート性能を最適化し、大規模クラスターで Routine Load タスクが停止する問題を解決しました。[#6293](https://github.com/StarRocks/starrocks/pull/6293)

### 問題修正

以下の問題を修正しました：

- SHOW FULL TABLES FROM DatabaseName 文でエスケープ文字の解析エラーが発生する問題を修正しました。[#6559](https://github.com/StarRocks/starrocks/issues/6559)
- FE のディスクスペースが急速に占有される問題を修正しました（BDBJE のバージョンをロールバックしてこのバグを修正）。[#6708](https://github.com/StarRocks/starrocks/pull/6708)
- 列式スキャンを有効にした後 (`enable_docvalue_scan=true`)、関連するフィールドがデータに含まれていないために BE がクラッシュする問題を修正しました。[#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

リリース日： 2022年5月22日

### 新機能

- 【パブリックテスト中】リソースグループ管理機能をリリースしました。リソースグループを使用して CPU とメモリのリソースを制御し、異なるテナントのクエリを同じクラスターで実行する際に、リソースの隔離を実現しつつ合理的にリソースを使用できるようにします。関連ドキュメントは[リソースグループ](../administration/resource_group.md)を参照してください。
- 【パブリックテスト中】Java UDF フレームワークを実装し、Java 言語で UDF（ユーザー定義関数）を記述して StarRocks の関数機能を拡張できるようにしました。関連ドキュメントは [Java UDF](../sql-reference/sql-functions/JAVA_UDF.md) を参照してください。
- 【パブリックテスト中】主キーモデルにデータをインポートする際に、一部の列の更新をサポートしました。注文の更新や複数のストリーム JOIN などのリアルタイムデータ更新シナリオでは、ビジネスに関連する列のみを更新する必要があります。関連ドキュメントは [主キーモデルのテーブルが部分更新をサポート](../loading/Load_to_Primary_Key_tables.md#部分更新) を参照してください。
- 【パブリックテスト中】JSON データ型と関数をサポートしました。関連ドキュメントは [JSON](../sql-reference/sql-statements/data-types/JSON.md) を参照してください。
- 外部テーブルを通じて Apache Hudi のデータをクエリすることで、データレイク分析の機能をさらに充実させました。関連ドキュメントは [Apache Hudi 外部テーブル](../data_source/External_table.md#deprecated-hudi-外部表) を参照してください。
- 新しい関数を追加しました：
  - ARRAY 関数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、[array_sort](../sql-reference/sql-functions/array-functions/array_sort.md)、[array_distinct](../sql-reference/sql-functions/array-functions/array_distinct.md)、[array_join](../sql-reference/sql-functions/array-functions/array_join.md)、[reverse](../sql-reference/sql-functions/string-functions/reverse.md)、[array_slice](../sql-reference/sql-functions/array-functions/array_slice.md)、[array_concat](../sql-reference/sql-functions/array-functions/array_concat.md)、[array_difference](../sql-reference/sql-functions/array-functions/array_difference.md)、[arrays_overlap](../sql-reference/sql-functions/array-functions/arrays_overlap.md)、[array_intersect](../sql-reference/sql-functions/array-functions/array_intersect.md)。
  - BITMAP 関数：[bitmap_max](../sql-reference/sql-functions/bitmap-functions/bitmap_max.md)、[bitmap_min](../sql-reference/sql-functions/bitmap-functions/bitmap_min.md)。
  - その他の関数：[retention](../sql-reference/sql-functions/aggregate-functions/retention.md)、[square](../sql-reference/sql-functions/math-functions/square.md)。

### 機能最適化

- CBO オプティマイザーの Parser と Analyzer をリファクタリングし、コード構造を最適化して Insert with CTE などの構文をサポートしました。公共テーブル表現（Common Table Expression、CTE）の再利用など、複雑なクエリのパフォーマンスを向上させました。
- Apache Hive 外部テーブルのクエリ性能を最適化し、オブジェクトストレージ（Amazon S3、阿里云OSS、腾讯云COS）に基づく外部テーブルのクエリ性能を HDFS ベースのクエリ性能とほぼ同等にしました。ORC 形式のファイルの遅延マテリアライゼーションをサポートし、小さなファイルのクエリパフォーマンスを向上させました。関連ドキュメントは [Apache Hive 外部テーブル](../data_source/External_table.md#deprecated-hive-外部表) を参照してください。
- Apache Hive のデータを外部テーブルを通じてクエリする際、Hive Metastore のイベント（データ変更、パーティション変更など）を定期的に消費してメタデータを自動的に増分更新します。さらに、Apache Hive の DECIMAL と ARRAY 型のデータをクエリできるようになりました。関連ドキュメントは [Apache Hive 外部テーブル](../data_source/External_table.md#deprecated-hive-外部表) を参照してください。
- UNION ALL 演算子のパフォーマンスを最適化し、2〜25倍のパフォーマンス向上を実現しました。
- Pipeline エンジンを正式にリリースし、クエリの並行度を自動的に調整し、Pipeline エンジンの Profile を最適化しました。高並行性のシナリオでの小規模クエリのパフォーマンスを向上させました。
- CSV ファイルをインポートする際に、複数の文字を行区切り文字として使用できるようにしました。

### 問題修正

以下の問題を修正しました：

- 主キーモデルのテーブルにデータをインポートし、COMMIT する際にデッドロックが発生する問題を修正しました。[#4998](https://github.com/StarRocks/starrocks/pull/4998)
- FE（BDBJE を含む）の一連の安定性問題を解決しました。[#4428](https://github.com/StarRocks/starrocks/pull/4428)、[#4666](https://github.com/StarRocks/starrocks/pull/4666)、[#2](https://github.com/StarRocks/bdb-je/pull/2)
- 大量のデータに対する SUM 関数の合計時に結果がオーバーフローする問題を修正しました。[#3944](https://github.com/StarRocks/starrocks/pull/3944)
- ROUND と TRUNCATE 関数の結果の精度問題を修正しました。[#4256](https://github.com/StarRocks/starrocks/pull/4256)
- SQLancer が発見した一連の問題を修正しました。SQLancer [関連 issues](https://github.com/StarRocks/starrocks/issues?q=is%3Aissue++label%3Asqlancer++milestone%3A2.2) を参照してください。

### その他

Flink コネクタ flink-connector-starrocks は Flink 1.14 バージョンをサポートしています。

### アップグレードに関する注意事項


- バージョン番号が 2.0.4 未満、または 2.1.x で 2.1.6 未満のユーザーは、[StarRocks アップグレードの注意事項](https://forum.starrocks.com/t/topic/2228)を参照してアップグレードしてください。
- アップグレード後に問題が発生し、ロールバックが必要な場合は、fe.conf ファイルに `ignore_unknown_log_id=true` を追加してください。これは新しいバージョンのメタデータログに新しいタイプが追加されたためで、このパラメータを追加しないとロールバックできません。チェックポイントが完了した後に `ignore_unknown_log_id=false` に設定し、FE を再起動して通常の設定に戻すことをお勧めします。
