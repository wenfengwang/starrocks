---
displayed_sidebar: English
---

# StarRocks バージョン 2.3

## 2.3.18

リリース日: 2023年10月11日

### バグ修正

以下の問題を修正しました:

- サードパーティライブラリ librdkafka のバグが原因で、Routine Load ジョブのロードタスクがデータロード中に停止し、新しく作成されたロードタスクも実行に失敗する問題。 [#28301](https://github.com/StarRocks/starrocks/pull/28301)
- Spark または Flink コネクタが、メモリ統計の不正確さのためにデータエクスポートに失敗する問題。 [#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- Stream Load ジョブが `if` キーワードを使用すると、BE がクラッシュする問題。 [#31926](https://github.com/StarRocks/starrocks/pull/31926)
- パーティション化された StarRocks 外部テーブルにデータをロードする際に `"get TableMeta failed from TNetworkAddress"` というエラーが報告される問題。  [#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

リリース日: 2023年9月4日

### バグ修正

以下の問題を修正しました:

- Routine Load ジョブがデータを消費できない問題。 [#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

リリース日: 2023年8月4日

### バグ修正

以下の問題を修正しました:

- LabelCleaner スレッドのブロックによって引き起こされる FE のメモリリーク。 [#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

リリース日: 2023年7月31日

## 改善点

- タブレットのスケジューリングロジックを最適化し、タブレットが長期間ペンディング状態になることや、特定の状況下で FE がクラッシュすることを防ぐ改善。 [#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- TabletChecker のスケジューリングロジックを最適化し、修復されていないタブレットを繰り返しスケジュールしないようにする改善。 [#27648](https://github.com/StarRocks/starrocks/pull/27648)
- パーティションメタデータが visibleTxnId を記録し、タブレットレプリカの表示バージョンに対応することで、レプリカのバージョンが他と矛盾する場合に、そのトランザクションを簡単に追跡できるようにする改善。 [#27924](https://github.com/StarRocks/starrocks/pull/27924)

### バグ修正

以下の問題を修正しました:

- FE でのテーブルレベルのスキャン統計が不正確で、テーブルクエリとローディングに関連するメトリクスが不正確になる問題。 [#28022](https://github.com/StarRocks/starrocks/pull/28022)
- Join キーが大きな BINARY 列である場合に BE がクラッシュする問題。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 特定のシナリオで集約演算子がスレッドセーフティの問題を引き起こし、BE がクラッシュする問題。 [#26092](https://github.com/StarRocks/starrocks/pull/26092)
- [RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) を使用してデータを復元した後、タブレットのバージョン番号が BE と FE で一致しない問題。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) を使用してテーブルを復旧した後にパーティションが自動的に作成されない問題。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)
- INSERT INTO を使用してロードするデータが品質要件を満たさない場合、データロードの厳格モードが有効であると、ローディングトランザクションが Pending 状態でスタックし、DDL ステートメントがハングする問題。 [#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 低カーディナリティ最適化が有効な場合に、一部の INSERT ジョブが `[42000][1064] Dict Decode failed, Dict can't take cover all key :0` というエラーを返す問題。 [#27395](https://github.com/StarRocks/starrocks/pull/27395)
- Pipeline が有効でない場合、INSERT INTO SELECT 操作がタイムアウトする問題。 [#26594](https://github.com/StarRocks/starrocks/pull/26594)
- クエリ条件が `WHERE partition_column < xxx` で、`xxx` の値が時間までしか正確でない場合（例: `2023-7-21 22`）、クエリがデータを返さない問題。 [#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

リリース日: 2023年6月28日

### 改善点

- CREATE TABLE がタイムアウトする際に返されるエラーメッセージを最適化し、パラメータチューニングのヒントを追加した改善。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 多数のタブレットバージョンが蓄積されたプライマリキーテーブルのメモリ使用量を最適化した改善。 [#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks 外部テーブルメタデータの同期をデータロード中に行うよう変更した改善。 [#24739](https://github.com/StarRocks/starrocks/pull/24739)
- サーバー間でシステムクロックが一致しないことによる不正な NetworkTime を修正するために、NetworkTime のシステムクロックへの依存を削除した改善。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題を修正しました:

- 頻繁に TRUNCATE 操作を行う小規模なテーブルに対して低カーディナリティ辞書最適化を適用すると、クエリでエラーが発生する問題。 [#23185](https://github.com/StarRocks/starrocks/pull/23185)
- 最初の子が定数 NULL の UNION を含むビューをクエリすると、BE がクラッシュする問題。 [#13792](https://github.com/StarRocks/starrocks/pull/13792)  
- 一部の場合に Bitmap Index を使用したクエリがエラーを返す問題。 [#23484](https://github.com/StarRocks/starrocks/pull/23484)
- DOUBLE または FLOAT 値を DECIMAL 値に丸める際の BE の結果が FE の結果と一致しない問題。 [#23152](https://github.com/StarRocks/starrocks/pull/23152)
- スキーマ変更とデータロードが同時に発生すると、スキーマ変更がハングする問題。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- Broker Load、Spark コネクタ、または Flink コネクタを使用して Parquet ファイルを StarRocks にロードする際に BE OOM の問題が発生する問題。 [#25254](https://github.com/StarRocks/starrocks/pull/25254)
- ORDER BY 句に定数が指定され、クエリに LIMIT 句がある場合に `unknown error` というエラーメッセージが返される問題。 [#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

リリース日: 2023年6月1日

### 改善点

- `thrift_server_max_worker_thread` の値が小さいために INSERT INTO ... SELECT がタイムアウトする際に報告されるエラーメッセージを最適化した改善。 [#21964](https://github.com/StarRocks/starrocks/pull/21964)
- `bitmap_contains` 関数を使用する複数テーブルの結合のメモリ消費を削減し、パフォーマンスを最適化した改善。 [#20617](https://github.com/StarRocks/starrocks/pull/20617) [#20653](https://github.com/StarRocks/starrocks/pull/20653)

### バグ修正

以下のバグが修正されました:


- パーティション名に対する TRUNCATE 操作は大文字小文字を区別するため、パーティションの切り捨てが失敗することがあります。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- Parquet ファイルからの int96 タイムスタンプデータのロードにより、データオーバーフローが発生することがあります。[#22355](https://github.com/StarRocks/starrocks/issues/22355)
- Materialized View が削除された後、BE のデコミッションが失敗することがあります。[#22743](https://github.com/StarRocks/starrocks/issues/22743)
- クエリの実行計画に Broadcast Join に続いて Bucket Shuffle Join が含まれる場合（例：`SELECT * FROM t1 JOIN [Broadcast] t2 ON t1.a = t2.b JOIN [Bucket] t3 ON t2.b = t3.c`）、Broadcast Join の左テーブルの等価結合キーのデータが Bucket Shuffle Join への送信前に削除されると、BE がクラッシュする可能性があります。[#23227](https://github.com/StarRocks/starrocks/pull/23227)
- クエリの実行計画に Cross Join に続いて Hash Join が含まれ、フラグメントインスタンス内の Hash Join の右テーブルが空の場合、返される結果が不正確になることがあります。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- Materialized View の一時パーティションの作成に失敗すると、BE のデコミッションが失敗することがあります。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- SQL ステートメントに複数のエスケープ文字を含む STRING 値が含まれる場合、SQL ステートメントの解析ができないことがあります。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- パーティション列の最大値をクエリすると失敗することがあります。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocks がバージョン 2.4 から 2.3 にロールバックした後、ロードジョブが失敗することがあります。[#23642](https://github.com/StarRocks/starrocks/pull/23642)
- 列のプルーニングと再利用に関連する問題があります。[#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

リリース日: 2023年4月25日

### 改善点

式の戻り値が有効な Boolean 値に変換可能であれば、暗黙の型変換をサポートします。[#21792](https://github.com/StarRocks/starrocks/pull/21792)

### バグ修正

以下のバグが修正されました：

- ユーザーにテーブルレベルで LOAD_PRIV が付与されている場合、ロードジョブの失敗によるトランザクションのロールバック時に `Access denied; you need (at least one of) the LOAD privilege(s) for this operation` というエラーメッセージが返されます。[#21129](https://github.com/StarRocks/starrocks/issues/21129)
- ALTER SYSTEM DROP BACKEND を実行して BE を削除した後、その BE 上でレプリケーション数が 2 に設定されたテーブルのレプリカが修復できないため、これらのテーブルへのデータロードが失敗します。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- CREATE TABLE でサポートされていないデータ型を使用すると、NPE が返されることがあります。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- Broadcast Join のショートサーキットロジックが異常で、クエリ結果が不正確になることがあります。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- Materialized View を使用すると、ディスク使用量が大幅に増加することがあります。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- Audit Loader プラグインを完全にアンインストールできないことがあります。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT` の結果に表示される行数が `SELECT COUNT(*) FROM XXX` の結果と一致しないことがあります。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- サブクエリがウィンドウ関数を使用し、親クエリが GROUP BY 句を使用する場合、クエリ結果が集約されないことがあります。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- BE が起動すると、BE プロセスは存在するものの、BE のポートが開かないことがあります。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- ディスク I/O が非常に高い場合、Primary Key テーブルのトランザクションのコミットが遅くなり、結果としてこれらのテーブルのクエリが "backend not found" エラーを返すことがあります。[#18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

リリース日: 2023年3月28日

### 改善点

- 多くの式を含む複雑なクエリを実行すると、多数の `ColumnRefOperators` が生成されます。元々 StarRocks は `ColumnRefOperator::id` を格納するために大量のメモリを消費する `BitSet` を使用していました。メモリ使用量を削減するため、StarRocks は `ColumnRefOperator::id` の格納に `RoaringBitMap` を使用するようになりました。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 新しい I/O スケジューリング戦略が導入され、大規模なクエリが小規模なクエリに与えるパフォーマンスへの影響を軽減します。新しい I/O スケジューリング戦略を有効にするには、**be.conf** 内の BE 静的パラメーター `pipeline_scan_queue_mode=1` を設定し、その後 BE を再起動してください。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### バグ修正

以下のバグが修正されました：

- 期限切れデータが適切にリサイクルされていないテーブルが、ディスクスペースの比較的大きな部分を占めることがあります。[#19796](https://github.com/StarRocks/starrocks/pull/19796)
- Broker Load ジョブが Parquet ファイルを StarRocks にロードし、`NULL` 値が NOT NULL 列にロードされた場合、表示されるエラーメッセージが不十分です。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 既存のパーティションを置き換えるために頻繁に多数の一時パーティションを作成すると、FE ノードでメモリリークとフル GC が発生することがあります。[#19283](https://github.com/StarRocks/starrocks/pull/19283)
- Colocation テーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");` といったステートメントを使用してレプリカのステータスを手動で `bad` として指定することができます。BE の数がレプリカ数以下である場合、破損したレプリカは修復できないことがあります。[#19443](https://github.com/StarRocks/starrocks/pull/19443)
- フォロワー FE に `INSERT INTO SELECT` リクエストが送信された場合、パラメーター `parallel_fragment_exec_instance_num` が効果を発揮しないことがあります。[#18841](https://github.com/StarRocks/starrocks/pull/18841)
- 演算子 `<=>` を使用して NULL 値との比較を行った場合、比較結果が正しくないことがあります。[#19210](https://github.com/StarRocks/starrocks/pull/19210)
- リソースグループのクエリ同時実行制限に連続して達すると、クエリの同時実行メトリックが遅く減少することがあります。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 高い同時実行データロードジョブが `"get database read lock timeout, database=xxx"` というエラーを引き起こすことがあります。[#16748](https://github.com/StarRocks/starrocks/pull/16748) [#18992](https://github.com/StarRocks/starrocks/pull/18992)

## 2.3.10

リリース日: 2023年3月9日

### 改善点


`storage_medium`の推論を最適化しました。BEがSSDとHDDを両方ともストレージデバイスとして使用している場合、`storage_cooldown_time`プロパティが指定されていれば、StarRocksは`storage_medium`を`SSD`に設定します。指定されていない場合は`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)

### バグ修正

以下のバグが修正されました：

- Data LakeのParquetファイルからARRAYデータをクエリすると、クエリが失敗することがあります。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- プログラムによって開始されたStream Loadジョブがハングし、FEがプログラムから送信されたHTTPリクエストを受信しません。[#18559](https://github.com/StarRocks/starrocks/pull/18559)
- Elasticsearch外部テーブルをクエリする際にエラーが発生することがあります。[#13727](https://github.com/StarRocks/starrocks/pull/13727)
- 初期化中に式でエラーが発生した場合、BEがクラッシュすることがあります。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- SQLステートメントで空の配列リテラル`[]`を使用すると、クエリが失敗することがあります。[#18550](https://github.com/StarRocks/starrocks/pull/18550)
- StarRocksをバージョン2.2以降からバージョン2.3.9以降にアップグレードした後、`COLUMN`パラメータで計算式を指定してRoutine Loadジョブを作成すると、`No match for <expr> with operand types xxx and xxx`というエラーが発生することがあります。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BEの再起動後にロードジョブがハングすることがあります。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- SELECTステートメントでWHERE句にOR演算子を使用すると、余計なパーティションがスキャンされることがあります。[#18610](https://github.com/StarRocks/starrocks/pull/18610)

## 2.3.9

リリース日：2023年2月20日

### バグ修正

- スキーマ変更中にタブレットクローンがトリガーされ、タブレットレプリカが存在するBEノードが変更されると、スキーマ変更が失敗します。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- `group_concat()`関数によって返される文字列が切り捨てられます。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- Tencent Big Data Suite（TBDS）を介してHDFSからデータをロードする際にBroker Loadを使用すると、`invalid hadoop.security.authentication.tbds.securekey`というエラーが発生し、StarRocksがTBDSによって提供される認証情報を使用してHDFSにアクセスできないことを示します。[#14125](https://github.com/StarRocks/starrocks/pull/14125) [#15693](https://github.com/StarRocks/starrocks/pull/15693)
- 場合によっては、CBOが2つの演算子が等価であるかどうかを比較する際に誤ったロジックを使用することがあります。[#17227](https://github.com/StarRocks/starrocks/pull/17227) [#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 非Leader FEノードに接続し、`USE <catalog_name>.<database_name>`というSQLステートメントを送信すると、非Leader FEノードは`<catalog_name>`を除外してSQLステートメントをLeader FEノードに転送します。結果として、Leader FEノードは`default_catalog`を使用しようとし、指定されたデータベースを見つけることができません。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

リリース日：2023年2月2日

### バグ修正

以下のバグが修正されました：

- 大規模なクエリが完了した後にリソースが解放されると、他のクエリが遅くなる可能性があります。この問題は、リソースグループが有効になっているか、大規模なクエリが予期せず終了した場合により発生しやすくなります。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- 主キーテーブルの場合、レプリカのメタデータバージョンが遅れていると、StarRocksは他のレプリカから欠落しているメタデータを増分的にクローンします。このプロセスで、StarRocksは多数のメタデータバージョンを引き出し、タイムリーなGCが行われないとメタデータのバージョンが多すぎてメモリを過剰に消費し、結果としてBEがOOM例外に遭遇する可能性があります。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- FEが不定期にBEにハートビートを送信し、ハートビート接続がタイムアウトすると、FEはBEが利用不可であると判断し、BEでトランザクションが失敗します。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- StarRocks外部テーブルを使用してStarRocksクラスタ間でデータをロードする場合、ソースStarRocksクラスタが以前のバージョンで、ターゲットStarRocksクラスタが新しいバージョン（2.2.8〜2.2.11、2.3.4〜2.3.7、2.4.1または2.4.2）の場合、データロードが失敗します。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- 複数のクエリが同時に実行され、メモリ使用量が比較的高い場合にBEがクラッシュすることがあります。[#16047](https://github.com/StarRocks/starrocks/pull/16047)
- テーブルに動的パーティショニングが有効で、一部のパーティションが動的に削除された場合、`TRUNCATE TABLE`を実行すると`NullPointerException`エラーが返されます。また、テーブルにデータをロードすると、FEがクラッシュし、再起動できなくなることがあります。[#16822](https://github.com/StarRocks/starrocks/pull/16822)

## 2.3.7

リリース日：2022年12月30日

### バグ修正

以下のバグが修正されました：

- StarRocksテーブルでNULLを許容するカラムが、そのテーブルから作成されたビューで誤ってNOT NULLとして設定されていました。[#15749](https://github.com/StarRocks/starrocks/pull/15749)
- StarRocksにデータをロードすると新しいタブレットバージョンが生成されますが、FEが新しいタブレットバージョンをまだ検出していない場合、BEはタブレットの履歴バージョンを読み取る必要があります。ガベージコレクションメカニズムが履歴バージョンを削除すると、クエリは履歴バージョンを見つけられず、`Not found: get_applied_rowsets(version xxxx) failed tablet:xxx #version:x [xxxxxxx]`というエラーが返されます。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- データが頻繁にロードされると、FEが過剰にメモリを消費します。[#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 集約クエリや複数テーブルJOINクエリでは、統計が正確に収集されず、実行プランにCROSS JOINが発生し、クエリのレイテンシが長くなります。[#12067](https://github.com/StarRocks/starrocks/pull/12067) [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

リリース日：2022年12月22日

### 改善点

- パイプライン実行エンジンがINSERT INTOステートメントをサポートします。これを有効にするには、FE設定項目`enable_pipeline_load_for_insert`を`true`に設定してください。[#14723](https://github.com/StarRocks/starrocks/pull/14723)
- 主キーテーブルのCompactionに使用されるメモリが削減されました。[#13861](https://github.com/StarRocks/starrocks/pull/13861) [#13862](https://github.com/StarRocks/starrocks/pull/13862)

### 動作変更

- FEパラメーター`default_storage_medium`を非推奨にしました。テーブルのストレージ媒体はシステムによって自動的に推測されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグ修正

以下のバグが修正されました：

- リソースグループ機能が有効になっており、複数のリソースグループが同時にクエリを実行する場合、BEがハングアップすることがあります。[#14905](https://github.com/StarRocks/starrocks/pull/14905)
- `CREATE MATERIALIZED VIEW AS SELECT`を使用してマテリアライズドビューを作成する際、SELECT句で集約関数を使用せず、GROUP BYを使用すると、例えば`CREATE MATERIALIZED VIEW test_view AS SELECT a,b from test group by b,a order by a;`の場合、BEノードが全てクラッシュします。[#13743](https://github.com/StarRocks/starrocks/pull/13743)
- `INSERT INTO`を使用してプライマリキーテーブルに頻繁にデータをロードし、データ変更を行った直後にBEを再起動すると、BEの再起動が非常に遅くなることがあります。[#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 環境にJREのみがインストールされており、JDKがインストールされていない場合、FEの再起動後にクエリが失敗します。このバグが修正された後、その環境ではFEは再起動できず、エラー`JAVA_HOME cannot be jre`が返されます。FEを正常に再起動するには、環境にJDKをインストールする必要があります。[#14332](https://github.com/StarRocks/starrocks/pull/14332)
- クエリがBEのクラッシュを引き起こします。[#14221](https://github.com/StarRocks/starrocks/pull/14221)
- `exec_mem_limit`を式で設定することはできません。[#13647](https://github.com/StarRocks/starrocks/pull/13647)
- サブクエリの結果に基づく同期リフレッシュ型マテリアライズドビューを作成することはできません。[#13507](https://github.com/StarRocks/starrocks/pull/13507)
- Hive外部テーブルをリフレッシュした後、カラムのコメントが削除されます。[#13742](https://github.com/StarRocks/starrocks/pull/13742)
- 相関JOINを行う際、右側のテーブルが左側のテーブルより先に処理され、右側のテーブルが非常に大きい場合、左側のテーブルにコンパクションが実行されている間にBEノードがクラッシュします。[#14070](https://github.com/StarRocks/starrocks/pull/14070)
- Parquetファイルのカラム名が大文字小文字を区別し、クエリ条件でParquetファイルの大文字のカラム名を使用する場合、クエリは結果を返しません。[#13860](https://github.com/StarRocks/starrocks/pull/13860) [#14773](https://github.com/StarRocks/starrocks/pull/14773)
- バルクロード中にBrokerへの接続数がデフォルトの最大接続数を超えると、Brokerが切断され、ロードジョブはエラーメッセージ`list path error`で失敗します。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- BEが高負荷時、リソースグループのメトリック`starrocks_be_resource_group_running_queries`が不正確になることがあります。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- クエリステートメントでOUTER JOINを使用すると、BEノードがクラッシュすることがあります。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- StarRocks 2.4を使用して非同期マテリアライズドビューを作成し、それを2.3にロールバックすると、FEが起動に失敗することがあります。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- プライマリキーテーブルが`delete_range`を使用し、パフォーマンスが悪い場合、RocksDBからのデータ読み取りが遅くなり、CPU使用率が高くなることがあります。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.5

リリース日：2022年11月30日

### 改善点

- Colocate JoinがEqui Joinをサポートします。[#13546](https://github.com/StarRocks/starrocks/pull/13546)
- データが頻繁にロードされる際にWALレコードが連続して追加されることにより、プライマリキーインデックスファイルが過大になる問題を修正しました。[#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FEがすべてのタブレットをバッチでスキャンし、長時間`db.readLock`を保持することを避けるために、スキャン間隔で`db.readLock`を解放します。[#13070](https://github.com/StarRocks/starrocks/pull/13070)

### バグ修正

以下のバグが修正されました：

- UNION ALLの結果に直接基づいてビューが作成され、UNION ALL演算子の入力列にNULL値が含まれている場合、ビューのスキーマが不正確になり、列のデータ型がUNION ALLの入力列ではなく`NULL_TYPE`になります。[#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...`と`SELECT * FROM ... LIMIT ...`のクエリ結果が一致しません。[#13585](https://github.com/StarRocks/starrocks/pull/13585)
- FEに同期された外部タブレットのメタデータがローカルタブレットのメタデータを上書きし、Flinkからのデータロードが失敗することがあります。[#12579](https://github.com/StarRocks/starrocks/pull/12579)
- ランタイムフィルターでnullフィルターがリテラル定数を処理すると、BEノードがクラッシュします。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- CTASを実行するとエラーが返されます。[#12388](https://github.com/StarRocks/starrocks/pull/12388)
- パイプラインエンジンによって収集された監査ログのメトリック`ScanRows`が誤っている可能性があります。[#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 圧縮されたHIVEデータをクエリすると、クエリ結果が不正確になります。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- BEノードがクラッシュした後、クエリがタイムアウトになり、StarRocksの応答が遅くなります。[#12955](https://github.com/StarRocks/starrocks/pull/12955)
- Broker Loadを使用してデータをロードする際にKerberos認証失敗のエラーが発生します。[#13355](https://github.com/StarRocks/starrocks/pull/13355)
- OR述語が多すぎると統計推定に時間がかかりすぎます。[#13086](https://github.com/StarRocks/starrocks/pull/13086)
- Broker Loadが大文字の列名を含むORCファイル（Snappy圧縮）をロードすると、BEノードがクラッシュします。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- プライマリキーテーブルのアンロードまたはクエリに30分以上かかるとエラーが返されます。[#13403](https://github.com/StarRocks/starrocks/pull/13403)
- ブローカーを使用してHDFSに大量のデータをバックアップする際にバックアップタスクが失敗します。[#12836](https://github.com/StarRocks/starrocks/pull/12836)
- `parquet_late_materialization_enable`パラメータが原因で、StarRocksがIcebergから読み取ったデータが不正確になることがあります。[#13132](https://github.com/StarRocks/starrocks/pull/13132)
- ビューを作成する際に`failed to init view stmt`のエラーが返されます。[#13102](https://github.com/StarRocks/starrocks/pull/13102)
- JDBCを使用してStarRocksに接続し、SQLステートメントを実行するとエラーが返されます。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- クエリが多数のバケットを含み、タブレットヒントを使用するとクエリがタイムアウトになります。[#13272](https://github.com/StarRocks/starrocks/pull/13272)
- BEノードがクラッシュし、再起動できなくなり、同時に新しく構築されたテーブルへのロードジョブがエラーを報告します。[#13701](https://github.com/StarRocks/starrocks/pull/13701)
- マテリアライズドビューが作成されるとすべてのBEノードがクラッシュします。[#13184](https://github.com/StarRocks/starrocks/pull/13184)
- ALTER ROUTINE LOADを実行して消費済みパーティションのオフセットを更新すると、エラー`The specified partition 1 is not in the consumed partitions`が返され、フォロワーが最終的にクラッシュする可能性があります。 [#12227](https://github.com/StarRocks/starrocks/pull/12227)

## 2.3.4

リリース日: 2022年11月10日

### 改善点

- 実行中のRoutine Loadジョブの数が制限を超えたためにStarRocksがRoutine Loadジョブの作成に失敗した場合、エラーメッセージが解決策を提供します。 [#12204](https://github.com/StarRocks/starrocks/pull/12204)
- StarRocksがHiveからデータをクエリする際にCSVファイルの解析に失敗すると、クエリが失敗します。 [#13013](https://github.com/StarRocks/starrocks/pull/13013)

### バグ修正

以下のバグが修正されました:

- HDFSファイルパスに`()`が含まれているとクエリが失敗する可能性があります。 [#12660](https://github.com/StarRocks/starrocks/pull/12660)
- サブクエリにLIMITが含まれている場合、ORDER BY ... LIMIT ... OFFSETの結果が不正確になることがあります。 [#9698](https://github.com/StarRocks/starrocks/issues/9698)
- StarRocksはORCファイルをクエリする際に大文字と小文字を区別しません。 [#12724](https://github.com/StarRocks/starrocks/pull/12724)
- RuntimeFilterがprepareメソッドを呼び出さずに閉じられた場合、BEがクラッシュする可能性があります。 [#12906](https://github.com/StarRocks/starrocks/issues/12906)
- メモリリークが原因でBEがクラッシュする可能性があります。 [#12906](https://github.com/StarRocks/starrocks/issues/12906)
- 新しいカラムを追加して直ちにデータを削除すると、クエリ結果が不正確になることがあります。 [#12907](https://github.com/StarRocks/starrocks/pull/12907)
- データのソート中にBEがクラッシュする可能性があります。 [#11185](https://github.com/StarRocks/starrocks/pull/11185)
- StarRocksとMySQLクライアントが同一LAN上にない場合、INSERT INTO SELECTを使用して作成されたローディングジョブは、KILLを一度だけ実行しても正常に終了しないことがあります。 [#11879](https://github.com/StarRocks/starrocks/pull/11897)
- 監査ログにおけるパイプラインエンジンによって収集された`ScanRows`メトリックが誤っている可能性があります。 [#12185](https://github.com/StarRocks/starrocks/pull/12185)

## 2.3.3

リリース日: 2022年9月27日

### バグ修正

以下のバグが修正されました:

- テキストファイルとして保存されたHive外部テーブルをクエリすると、クエリ結果が不正確になる可能性があります。 [#11546](https://github.com/StarRocks/starrocks/pull/11546)
- Parquetファイルをクエリする際にネストされた配列がサポートされていません。 [#10983](https://github.com/StarRocks/starrocks/pull/10983)
- StarRocksと外部データソースからデータを読み込む同時クエリが同じリソースグループにルーティングされるか、クエリがStarRocksと外部データソースからデータを読み込む場合、クエリがタイムアウトすることがあります。 [#10983](https://github.com/StarRocks/starrocks/pull/10983)
- パイプライン実行エンジンがデフォルトで有効になっている場合、parallel_fragment_exec_instance_numパラメータが1に設定されます。これによりINSERT INTOを使用したデータロードが遅くなることがあります。 [#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 式の初期化時にエラーがあるとBEがクラッシュすることがあります。 [#11396](https://github.com/StarRocks/starrocks/pull/11396)
- ORDER BY LIMITを実行するとheap-buffer-overflowエラーが発生することがあります。 [#11185](https://github.com/StarRocks/starrocks/pull/11185)
- Leader FEを再起動するとスキーマ変更が失敗することがあります。 [#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

リリース日: 2022年9月7日

### 新機能

- Parquet形式の外部テーブルに対する範囲フィルタベースのクエリを高速化するために、遅延マテリアライゼーションがサポートされています。 [#9738](https://github.com/StarRocks/starrocks/pull/9738)
- ユーザ認証関連情報を表示するSHOW AUTHENTICATIONステートメントが追加されました。 [#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改善点

- StarRocksがデータをクエリするバケット化されたHiveテーブルのすべてのデータファイルを再帰的にトラバースするかどうかを制御する設定項目が提供されています。 [#10239](https://github.com/StarRocks/starrocks/pull/10239)
- リソースグループタイプ`realtime`が`short_query`に名称変更されました。 [#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocksはデフォルトでHive外部テーブルの大文字と小文字を区別しなくなりました。 [#10187](https://github.com/StarRocks/starrocks/pull/10187)

### バグ修正

以下のバグが修正されました:

- Elasticsearch外部テーブルに対するクエリが、テーブルが複数のシャードに分割されている場合に予期せず終了することがあります。 [#10369](https://github.com/StarRocks/starrocks/pull/10369)
- サブクエリが共通テーブル式(CTE)として書き換えられた場合、StarRocksがエラーを投げることがあります。 [#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 大量のデータをロードする際にStarRocksがエラーを投げることがあります。 [#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 複数のカタログに同じThriftサービスIPアドレスが設定されている場合、一つのカタログを削除すると、他のカタログでのインクリメンタルなメタデータ更新が無効になることがあります。 [#10511](https://github.com/StarRocks/starrocks/pull/10511)
- BEからのメモリ消費統計が不正確です。 [#9837](https://github.com/StarRocks/starrocks/pull/9837)
- StarRocksがプライマリキーテーブルに対するクエリでエラーを投げることがあります。 [#10811](https://github.com/StarRocks/starrocks/pull/10811)
- SELECT権限を持っている論理ビューに対するクエリが許可されていないことがあります。 [#10563](https://github.com/StarRocks/starrocks/pull/10563)
- 論理ビューはテーブルと同じ命名規則に従う必要があるため、StarRocksは論理ビューの命名に制限を課しました。 [#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 動作変更

- ビットマップ関数用にデフォルト値1000000のBE設定`max_length_for_bitmap_function`を追加し、base64用にデフォルト値200000の`max_length_for_to_base64`を追加してクラッシュを防ぎます。 [#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

リリース日: 2022年8月22日

### 改善点

- Broker LoadがParquetファイル内のList型をネストされていないARRAYデータ型に変換することをサポートします。 [#9150](https://github.com/StarRocks/starrocks/pull/9150)
- JSON関連関数(json_query、get_json_string、get_json_int)のパフォーマンスが最適化されました。 [#9623](https://github.com/StarRocks/starrocks/pull/9623)
- Hive、Iceberg、またはHudiでのクエリ中にStarRocksがサポートしていないデータ型のカラムをクエリする場合、システムがそのカラムに対して例外を投げるようにエラーメッセージが最適化されました。 [#10139](https://github.com/StarRocks/starrocks/pull/10139)
- リソースグループのスケジューリング遅延が短縮され、リソース分離のパフォーマンスが最適化されました。 [#10122](https://github.com/StarRocks/starrocks/pull/10122)

### バグ修正

以下のバグが修正されました:


- `limit` 演算子のプッシュダウンが不正確であるため、Elasticsearch外部テーブルからのクエリ結果が誤っています。 [#9952](https://github.com/StarRocks/starrocks/pull/9952)
- `limit` 演算子を使用したOracle外部テーブルのクエリが失敗します。 [#9542](https://github.com/StarRocks/starrocks/pull/9542)
- Routine Load中にすべてのKafkaブローカーが停止した際に、BEがブロックされます。 [#9935](https://github.com/StarRocks/starrocks/pull/9935)
- 対応する外部テーブルのデータ型と一致しないParquetファイルのクエリ中にBEがクラッシュします。 [#10107](https://github.com/StarRocks/starrocks/pull/10107)
- 外部テーブルのスキャン範囲が空であるため、クエリがタイムアウトになります。 [#10091](https://github.com/StarRocks/starrocks/pull/10091)
- サブクエリに`ORDER BY`句が含まれると、システムが例外を投げます。 [#10180](https://github.com/StarRocks/starrocks/pull/10180)
- Hiveメタデータが非同期でリロードされる際に、Hive Metastoreがハングアップします。 [#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

リリース日: 2022年7月29日

### 新機能

- 主キーテーブルが完全な`DELETE WHERE`構文をサポートします。詳細は[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)を参照してください。
- 主キーテーブルは永続的な主キーインデックスをサポートします。ディスク上に主キーインデックスを永続化することで、メモリ使用量を大幅に削減できます。詳細は[主キーテーブル](../table_design/table_types/primary_key_table.md)を参照してください。
- リアルタイムデータ取り込み中にグローバルディクショナリを更新でき、文字列データのクエリパフォーマンスを2倍に向上させ、クエリパフォーマンスを最適化します。
- `CREATE TABLE AS SELECT`ステートメントは非同期で実行可能です。詳細は[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を参照してください。
- 以下のリソースグループ関連機能をサポートします：
  - リソースグループの監視：監査ログでクエリのリソースグループを表示し、APIを呼び出してリソースグループのメトリクスを取得できます。詳細は[監視とアラート](../administration/Monitor_and_Alert.md#monitor-and-alerting)を参照してください。
  - CPU、メモリ、I/Oリソースに対する大規模クエリの消費を制限：分類器に基づいて、またはセッション変数を設定することで、クエリを特定のリソースグループにルーティングできます。詳細は[リソースグループ](../administration/resource_group.md)を参照してください。
- JDBC外部テーブルを使用して、Oracle、PostgreSQL、MySQL、SQLServer、ClickHouseなどのデータベースのデータを簡単にクエリできます。StarRocksは述語のプッシュダウンもサポートし、クエリパフォーマンスを向上させます。詳細は[JDBC互換データベースの外部テーブル](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)を参照してください。
- [プレビュー] 外部カタログをサポートする新しいData Source Connectorフレームワークがリリースされました。外部カタログを使用して、外部テーブルを作成せずに直接Hiveデータにアクセスしクエリを実行できます。詳細は[カタログを使用して内部データと外部データを管理する](../data_source/catalog/query_external_data.md)を参照してください。
- 以下の関数を追加しました：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)、[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)、[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改善点

- コンパクションメカニズムが大量のメタデータをより迅速にマージできるようになりました。これにより、頻繁なデータ更新後に発生するメタデータの圧迫と過剰なディスク使用を防ぐことができます。
- Parquetファイルと圧縮ファイルのロードパフォーマンスが最適化されました。
- マテリアライズドビューの作成メカニズムが最適化され、以前より最大10倍速く作成できるようになりました。
- 以下の演算子のパフォーマンスが最適化されました：
  - TopNおよびソート演算子
  - 関数を含む等価比較演算子は、スキャン演算子にプッシュダウンされた際にZone Mapインデックスを使用できます。
- Apache Hive™外部テーブルのパフォーマンスが最適化されました。
  - Apache Hive™テーブルがParquet、ORC、CSV形式で格納されている場合、Hive外部テーブルに対して`REFRESH`ステートメントを実行すると、Hiveでの`ADD COLUMN`や`REPLACE COLUMN`によるスキーマ変更がStarRocksに同期されます。詳細は[Hive外部テーブル](../data_source/External_table.md#hive-external-table)を参照してください。
  - `hive.metastore.uris`はHiveリソースに対して変更可能です。詳細は[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。
- Apache Iceberg外部テーブルのパフォーマンスが最適化されました。カスタムカタログを使用してIcebergリソースを作成できます。詳細は[Apache Iceberg外部テーブル](../data_source/External_table.md#apache-iceberg-external-table)を参照してください。
- Elasticsearch外部テーブルのパフォーマンスが最適化されました。Elasticsearchクラスターのデータノードアドレスのスニッフィングを無効にすることができます。詳細は[Elasticsearch外部テーブル](../data_source/External_table.md#elasticsearch-external-table)を参照してください。
- `sum()`関数が数値文字列を受け取る場合、それを暗黙的に数値に変換します。
- `year()`、`month()`、`day()`関数はDATEデータ型をサポートします。

### バグ修正

以下のバグを修正しました：

- タブレットの数が多すぎるためにCPU使用率が急増します。
- 「タブレットリーダーの準備に失敗する」という問題。
- FEが再起動できない問題。[#5642](https://github.com/StarRocks/starrocks/issues/5642)  [#4969](https://github.com/StarRocks/starrocks/issues/4969)  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- CTASステートメントにJSON関数が含まれている場合に、CTASステートメントを正常に実行できない問題。 [#6498](https://github.com/StarRocks/starrocks/issues/6498)

### その他

- StarGoはクラスタ管理ツールで、クラスタのデプロイ、起動、アップグレード、ロールバックを行い、複数のクラスタを管理できます。詳細は[StarGoを使用したStarRocksのデプロイ](../administration/stargo.md)を参照してください。
- StarRocksをバージョン2.3にアップグレードするか、StarRocksをデプロイする際に、パイプラインエンジンがデフォルトで有効になります。パイプラインエンジンは、高い並行性のシナリオと複雑なクエリでシンプルなクエリのパフォーマンスを向上させます。StarRocks 2.3の使用中にパフォーマンスの大幅な低下が検出された場合は、`SET GLOBAL`ステートメントを実行して`enable_pipeline_engine`を`false`に設定することで、パイプラインエンジンを無効にできます。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) ステートメントは MySQL 構文と互換性があり、ユーザーに割り当てられた権限を GRANT ステートメントの形で表示します。
- `memory_limitation_per_thread_for_schema_change`（BE 設定項目）は、デフォルト値の 2 GB を使用することを推奨し、データ量がこの制限を超えた場合はディスクにデータを書き込むようになります。したがって、以前にこのパラメータをより大きな値に設定していた場合は、2 GB に設定することを推奨します。そうしないと、スキーマ変更タスクが大量のメモリを消費する可能性があります。

### アップグレードノート

アップグレード前に使用していたバージョンにロールバックするには、各 FE の **fe.conf** ファイルに `ignore_unknown_log_id` パラメータを追加し、そのパラメータを `true` に設定します。StarRocks v2.2.0 で新しいタイプのログが追加されたため、このパラメータは必須です。パラメータを追加しないと、以前のバージョンにロールバックできません。チェックポイントが作成された後、各 FE の **fe.conf** ファイルで `ignore_unknown_log_id` パラメータを `false` に設定することを推奨します。その後、FE を再起動して、以前の設定に復元します。
