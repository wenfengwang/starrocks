---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.2

リリース日: 2023年4月6日

## 改善点

- bitmap_contains() 関数を最適化し、一部のシナリオにおけるメモリ消費を減らし、パフォーマンスを向上させました。[#20616](https://github.com/StarRocks/starrocks/issues/20616)
- Compaction フレームワークを最適化し、CPU リソース消費を削減しました。[#11746](https://github.com/StarRocks/starrocks/issues/11746)

## バグ修正

以下のバグが修正されました:

- Stream Load ジョブでリクエストされたURLが正しくない場合、責任を持つ FE がハングして HTTP リクエストを処理できなくなります。[#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 責任を持つ FE が統計情報を収集する際、異常に大きなメモリ量を消費する可能性があり、OOM を引き起こすことがあります。[#16331](https://github.com/StarRocks/starrocks/issues/16331)
- クエリのメモリ解放が適切に処理されない場合、BE がクラッシュします。[#11395](https://github.com/StarRocks/starrocks/issues/11395)
- TRUNCATE TABLE コマンドを実行した後、NullPointerException が発生し、責任を持つ FE が再起動に失敗することがあります。[#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

リリース日: 2022年12月2日

### 改善点

- Routine Load ジョブ向けのエラーメッセージを最適化しました。[#12203](https://github.com/StarRocks/starrocks/pull/12203)
- 論理演算子 `&&` をサポートしました。[#11819](https://github.com/StarRocks/starrocks/issues/11819)
- BE クラッシュ時にクエリを即座にキャンセルし、クエリのタイムアウトによるシステムの停止問題を防止しました。[#12954](https://github.com/StarRocks/starrocks/pull/12954)
- FE 開始スクリプトを最適化しました。FE 開始時に Java バージョンをチェックするようになりました。[#14094](https://github.com/StarRocks/starrocks/pull/14094)
- プライマリキー テーブルから大量のデータを削除することをサポートしました。[#4772](https://github.com/StarRocks/starrocks/issues/4772)

### バグ修正

以下のバグが修正されました:

- ユーザーが複数のテーブル（UNION）からビューを作成する際、UNION 操作の左端の子が NULL 定数を使用すると、BE がクラッシュします。([#13792](https://github.com/StarRocks/starrocks/pull/13792))
- クエリで問い合わせる Parquet ファイルの列タイプが Hive テーブルのスキーマと一致しない場合、BE がクラッシュします。[#8848](https://github.com/StarRocks/starrocks/issues/8848)
- クエリに大量の OR 演算子を含む場合、プランナーが過剰な再帰的な計算を実行する必要があり、クエリがタイムアウトすることがあります。[#12788](https://github.com/StarRocks/starrocks/pull/12788)
- サブクエリに LIMIT 句が含まれる場合、クエリの結果が正しくありません。[#12466](https://github.com/StarRocks/starrocks/pull/12466)
- SELECT 句で二重引用符と単一引用符が混在すると、CREATE VIEW ステートメントが失敗します。[#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

リリース日: 2022年11月15日

### 改善点

- 収集する Hive パーティションの数を制御するためのセッション変数 `hive_partition_stats_sample_size` を追加しました。過剰なパーティションの数は、Hive メタデータの取得におけるエラーを引き起こすことがあります。[#12700](https://github.com/StarRocks/starrocks/pull/12700)
- Elasticsearch 外部テーブルがカスタムタイムゾーンをサポートするようにしました。[#12662](https://github.com/StarRocks/starrocks/pull/12662)

### バグ修正

以下のバグが修正されました:

- 外部テーブルのメタデータ同期中にエラーが発生すると、DECOMMISSION 操作がスタックします。[#12369](https://github.com/StarRocks/starrocks/pull/12368)
- 新しく追加された列が削除されると、Compaction がクラッシュします。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- SHOW CREATE VIEW は、ビュー作成時に追加されたコメントを表示しません。[#4163](https://github.com/StarRocks/starrocks/issues/4163)
- Java UDF のメモリリークが OOM を引き起こす可能性があります。[#12418](https://github.com/StarRocks/starrocks/pull/12418)
- 補助 FE に格納されたノードの生存状態が `heartbeatRetryTimes` に依存しているため、一部のシナリオで正確でない場合があります。この問題を修正するために、 `HeartbeatResponse` にプロパティ `aliveStatus` が追加されました。[#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 動作の変更

StarRocks がクエリ可能な Hive STRING 列の長さを 64 KB から 1 MB に拡張しました。STRING 列が 1 MB を超える場合、クエリ中に NULL 列として処理されます。[#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

リリース日: 2022年10月17日

### バグ修正

以下のバグが修正されました:

- 初期化段階で式がエラーに遭遇した場合、BE がクラッシュする可能性があります。[#11395](https://github.com/StarRocks/starrocks/pull/11395)
- 無効な JSON データがロードされると、BE がクラッシュする可能性があります。[#10804](https://github.com/StarRocks/starrocks/issues/10804)
- パイプラインエンジンが有効になっている場合、並列書き込みでエラーが発生します。[#11451](https://github.com/StarRocks/starrocks/issues/11451)
- ORDER BY NULL LIMIT 句が使用されると、BE がクラッシュします。[#11648](https://github.com/StarRocks/starrocks/issues/11648)
- クエリで問い合わせる Parquet ファイルの列タイプが Hive テーブルのスキーマと不一致の場合、BE がクラッシュします。[#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

リリース日: 2022年9月23日

### バグ修正

以下のバグが修正されました:

- ユーザーが JSON データを StarRocks にロードすると、データが失われる可能性があります。[#11054](https://github.com/StarRocks/starrocks/issues/11054)
- SHOW FULL TABLES の出力が正しくありません。[#11126](https://github.com/StarRocks/starrocks/issues/11126)
- 以前のバージョンでは、ビューでデータにアクセスするためには、ユーザーはベーステーブルとビューの両方に権限を持っている必要がありました。現在のバージョンでは、ビューに対する権限のみが必要です。[#11290](https://github.com/StarRocks/starrocks/pull/11290)
- EXISTS や IN でネストされた複雑なクエリの結果が正しくありません。[#11415](https://github.com/StarRocks/starrocks/pull/11415)
- 対応する Hive テーブルのスキーマが変更された場合、REFRESH EXTERNAL TABLE が失敗します。[#11406](https://github.com/StarRocks/starrocks/pull/11406)
- リーダーではない FE がビットマップ索引作成操作を再実行すると、エラーが発生する可能性があります。[#11261](https://github.com/StarRocks/starrocks/issues/11261)

## 2.2.6

リリース日: 2022年9月14日

### バグ修正

以下のバグが修正されました:

- サブクエリに LIMIT 句が含まれる場合、`order by... limit...offset` の結果が正しくありません。[#9698](https://github.com/StarRocks/starrocks/issues/9698)
- 大容量のデータを持つテーブルで部分更新を実行すると、BE がクラッシュします。[#9809](https://github.com/StarRocks/starrocks/issues/9809)
- Compaction が、コンパクト化する BITMAP データのサイズが 2 GB を超える場合、BE がクラッシュします。[#11159](https://github.com/StarRocks/starrocks/pull/11159)
- like() 関数や regexp() 関数が、パターンの長さが 16 KB を超えると機能しなくなります。[#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 動作の変更

配列内の JSON 値を表すために使用されるフォーマットが変更されました。返される JSON 値にはエスケープ文字が使用されなくなりました。[#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

リリース日: 2022年8月18日

### 改善点

- パイプラインエンジンが有効になっている場合のシステムパフォーマンスを向上させました。[#9580](https://github.com/StarRocks/starrocks/pull/9580)
- インデックスメタデータのメモリ統計の精度を向上させました。[#9837](https://github.com/StarRocks/starrocks/pull/9837)

### バグ修正

以下のバグが修正されました:

- Routine Load 中の Kafka パーティションオフセット（`get_partition_offset`）のクエリで BE がスタックすることがあります。[#9937](https://github.com/StarRocks/starrocks/pull/9937)
- 複数のブローカーロードスレッドが同じHDFSファイルをロードしようとするとエラーが発生します。[#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

リリース日：2022年8月3日

### 改善点

- Hiveテーブルのスキーマ変更を対応する外部テーブルに同期するようにサポートされました。[#9010](https://github.com/StarRocks/starrocks/pull/9010)

- ブローカーロードを介してParquetファイルからARRAYデータをロードするようにサポートされました。[#9131](https://github.com/StarRocks/starrocks/pull/9131)

### バグ修正

以下のバグが修正されました：

- ブローカーロードが複数のKeytabファイルでのKerberosログインを処理できない場合があります。[#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- **stop_be.sh** が実行された直後にすぐに終了すると、スーパーバイザがサービスの再起動に失敗する場合があります。[#9175](https://github.com/StarRocks/starrocks/pull/9175)

- 不正なJoin Reorderの先行順位がエラー "Column cannot be resolved" を引き起こす可能性があります。[#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

リリース日：2022年7月24日

### バグ修正

以下のバグが修正されました：

- ユーザーがリソースグループを削除する際にエラーが発生する場合があります。[#8036](https://github.com/StarRocks/starrocks/pull/8036)

- スレッド数が不十分な場合にThriftサーバーが終了する場合があります。[#7974](https://github.com/StarRocks/starrocks/pull/7974)

- 特定のシナリオでCBOのジョインリオーダが結果を返さないことがあります。[#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

リリース日：2022年6月29日

### 改善点

- UDFをデータベース全体で使用できるようになりました。[#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- スキーマ変更などの内部処理の並行制御が最適化されました。これにより、高い並行性で大量のデータをロードするシナリオで、ロードジョブが積み上がる可能性や遅延する可能性が低減されます。[#6838](https://github.com/StarRocks/starrocks/pull/6838)

### バグ修正

以下のバグが修正されました：

- CTASを使用して作成されたレプリカ数 (`replication_num`) が正しくありません。[#7036](https://github.com/StarRocks/starrocks/pull/7036)

- ALTER ROUTINE LOADを実行した後にメタデータが失われる可能性があります。[#7068](https://github.com/StarRocks/starrocks/pull/7068)

- ランタイムフィルタがプッシュダウンされないことがあります。[#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- メモリリークを引き起こす可能性のあるパイプラインの問題。[#7295](https://github.com/StarRocks/starrocks/pull/7295)

- ルーチンロードジョブが中断されたときにデッドロックが発生する可能性があります。[#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 一部のプロファイル統計情報が不正確である可能性があります。[#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string 関数がJSON配列を誤って処理することがあります。[#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

リリース日：2022年6月2日

### 改善点

- ホットスポットコードの一部を再構築し、ロックの粒度を減らすことでデータロードのパフォーマンスを最適化し、長いテールレイテンシを低減しました。[#6641](https://github.com/StarRocks/starrocks/pull/6641)

- BEが展開されているマシンのCPU使用率とメモリ使用量の情報を、各クエリのFE監査ログに追加しました。[#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- Primari KeyテーブルとUnique KeyテーブルでJSONデータ型をサポートしました。[#6544](https://github.com/StarRocks/starrocks/pull/6544)

- ロックの粒度を減らすことでFEの負荷を軽減し、BEレポートリクエストの重複を解消しました。[#6293](https://github.com/StarRocks/starrocks/pull/6293)

### バグ修正

以下のバグが修正されました：

- StarRocksが `SHOW FULL TABLES FROM DatabaseName` ステートメントで指定されたエスケープ文字を解析する際にエラーが発生する場合があります。[#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FEディスクスペースの使用率が急上昇します（BDBJEバージョンをロールバックすることでこのバグを修正しました）。[#6708](https://github.com/StarRocks/starrocks/pull/6708)

- カラムスキャンを有効にした後にデータ内から関連フィールドが見つからないため、BEが故障する場合があります。[#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

リリース日：2022年5月22日

### 新機能

- [プレビュー] リソースグループ管理機能がリリースされました。この機能により、StarRocksは同じクラスター内の異なるテナントからの複雑なクエリとシンプルなクエリの両方を効率的に処理し、CPUとメモリリソースを分離して効率的に使用できるようになります。

- [プレビュー] Javaベースのユーザ定義関数（UDF）フレームワークが実装されました。このフレームワークは、Javaの構文に準拠してコンパイルされたUDFをサポートし、StarRocksの機能を拡張します。

- [プレビュー] Primari Keyテーブルが、リアルタイムデータ更新シナリオ（注文更新やマルチストリームジョインなど）でプライマリキーテーブルにデータをロードする際に特定のカラムのみを更新することをサポートします。

- [プレビュー] JSONデータ型とJSON関数がサポートされました。

- 外部テーブルを使用してApache Hudiからデータをクエリできるようになりました。これにより、StarRocksを使用したユーザのデータレイク分析体験がさらに向上します。詳細については、[External tables](../data_source/External_table.md)を参照してください。

- 次の機能が追加されました：
  - ARRAY関数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md), array_sort, array_distinct, array_join, reverse, array_slice, array_concat, array_difference, arrays_overlap, および array_intersect
  - BITMAP関数：bitmap_max および bitmap_min
  - retention および squareなどのその他の関数

### 改善点

- コストベースの最適化機能（CBO）のパーサーとアナライザーが再構築され、コード構造が最適化され、CTEを使ったINSERTなどの構文がサポートされるようになりました。これにより、CTEの再利用を含む複雑なクエリのパフォーマンスが向上しました。

- Apache Hive™外部テーブルのクエリのパフォーマンスが最適化されました。これにより、オブジェクトストレージベースのクエリのパフォーマンスがHDFSベースのクエリと比較可能になりました。さらに、ORCファイルの遅延素材化がサポートされ、小さなファイルのクエリが加速されました。詳細については、[Apache Hive™外部テーブル](../data_source/External_table.md)を参照してください。

- Apache Hive™からのクエリを実行する際、StarRocksはHiveメタストアイベントを消費してキャッシュされたメタデータの増分更新を自動的に行い、DECIMALおよびARRAYタイプのデータのクエリをサポートします。詳細については、[Apache Hive™外部テーブル](../data_source/External_table.md)を参照してください。

- UNION ALL演算子が以前よりも2〜25倍速く実行されるように最適化されました。

- 適応型並列性をサポートし、最適化されたプロファイルを提供するパイプラインエンジンがリリースされ、高い並行性シナリオでのシンプルなクエリのパフォーマンスが向上しました。

- 1つの行の区切り文字として複数の文字を組み合わせてCSVファイルをインポートできるようになりました。

### バグ修正

- Primari Keyテーブルに基づいたテーブルにデータがロードされたり変更がコミットされると、デッドロックが発生する可能性があります。[#4998](https://github.com/StarRocks/starrocks/pull/4998)
- フロントエンド（FE）は不安定です。これには、Oracle Berkeley DB Java Edition（BDB JE）を実行するFEも含まれます。[#4428](https://github.com/StarRocks/starrocks/pull/4428)、[#4666](https://github.com/StarRocks/starrocks/pull/4666)、[#2](https://github.com/StarRocks/bdb-je/pull/2)

- SUM 関数によって返される結果は、大量のデータに対して関数が呼び出される場合に算術オーバーフローが発生します。[#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUND 関数および TRUNCATE 関数によって返される結果の精度が不満です。[#4256](https://github.com/StarRocks/starrocks/pull/4256)

- 合成クエリランサー（SQLancer）によっていくつかのバグが検出されます。詳細は [SQLancer 関連 issues](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2) を参照してください。

### その他

Flink-connector-starrocks は Apache Flink® v1.14 をサポートしています。

### アップグレード注意事項

- 2.0.4 より新しい StarRocks バージョンまたは 2.1.6 より新しい StarRocks バージョン 2.1.x を使用している場合、アップグレード前にタブレットクローン機能を無効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");` および `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。アップグレード後にこの機能を有効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");` および `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。

- アップグレード前に使用していた以前のバージョンにロールバックするには、各 FE の **fe.conf** ファイルに `ignore_unknown_log_id` パラメータを追加し、パラメータを `true` に設定します。このパラメータは StarRocks v2.2.0 で新しい種類のログが追加されたために必要です。パラメータを追加しないと、以前のバージョンにロールバックすることはできません。チェックポイントが作成された後に、各 FE の **fe.conf** ファイルで `ignore_unknown_log_id` パラメータを `false` に設定することをお勧めします。その後、FE を再起動して前の設定に戻します。