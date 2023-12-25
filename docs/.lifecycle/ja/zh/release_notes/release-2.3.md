---
displayed_sidebar: Chinese
---

# StarRocks バージョン 2.3

## 2.3.18

リリース日： 2023年10月11日

### 問題修正

以下の問題を修正しました：

- サードパーティライブラリ librdkafka の不具合により、Routine Load でのインポートタスクが消費中に停止し、新しいインポートタスクも実行できなくなる問題。[#28301](https://github.com/StarRocks/starrocks/pull/28301)
- メモリ統計の不正確さが原因で、Spark または Flink connector がデータを読み取る際にエラーが発生する可能性がある問題。[#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- Stream Load で `if` キーワードを含むと、BE がクラッシュする問題。[#31926](https://github.com/StarRocks/starrocks/pull/31926)
- パーティション付き StarRocks 外部テーブルにデータを書き込む際に `"get TableMeta failed from TNetworkAddress"` というエラーが発生する問題。[#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

リリース日： 2023年9月4日

### 問題修正

以下の問題を修正しました：

- Routine Load の消費に失敗する問題。[#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

リリース日： 2023年8月4日

### 問題修正

以下の問題を修正しました：

- LabelCleaner スレッドが停止し、FE のメモリリークを引き起こす問題。[#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

リリース日： 2023年7月31日

### 機能最適化

- Tablet スケジューリングロジックを最適化し、特定の状況で Tablet が長時間サスペンド状態になったり、FE がクラッシュするのを避ける。[#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- TabletChecker のスケジューリングロジックを最適化し、一時的に修復不可能な Tablet の再スケジューリングを避ける。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- パーティションのメタデータに visibleTxnId を記録し、対応する visible version を追跡し、Tablet のレプリカバージョンが一致しない場合に、そのバージョンを作成した txn を追跡しやすくする。[#27924](https://github.com/StarRocks/starrocks/pull/27924)

### 問題修正

以下の問題を修正しました：

- FE でのテーブルレベルのスキャン統計情報が誤っており、テーブルのクエリとインポートの metrics 情報が正しくない問題。[#28022](https://github.com/StarRocks/starrocks/pull/28022)
- Join 列が BINARY 型でサイズが大きすぎる場合に BE がクラッシュする問題。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 特定の状況で Aggregate オペレータがスレッドセーフの問題を引き起こし、BE がクラッシュする問題。[#26092](https://github.com/StarRocks/starrocks/pull/26092)
- [RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) 後に同じ Tablet のバージョンが BE と FE で一致しない問題。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) でのテーブルの自動パーティション作成に失敗する問題。[#26813](https://github.com/StarRocks/starrocks/pull/26813)
- 厳格モードで INSERT INTO のデータに品質問題があり、インポートトランザクションが pending 状態になり続け、DDL ステートメントが停止する問題。[#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 低カーディナリティ最適化が有効な場合、特定の状況で INSERT INTO インポートタスクがエラー `[42000][1064] Dict Decode failed, Dict can't take cover all key :0` を報告する問題。[#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 特定の状況で INSERT INTO SELECT が Pipeline が無効の場合にタイムアウトする問題。[#26594](https://github.com/StarRocks/starrocks/pull/26594)
- クエリ条件が `WHERE partition_column < xxx` であり、使用されるパーティション列の値が時間までしか正確でなく、分秒まで正確でない場合（例：`2023-7-21 22`）、結果が空になる問題。[#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

リリース日： 2023年6月28日

### 機能最適化

- CREATE TABLE のタイムアウトエラーメッセージを最適化し、パラメータ調整の提案を追加。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 主キーモデルのテーブルが大量の Tablet バージョンを蓄積した後のメモリ使用量を最適化。[#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks 外部テーブルのメタデータ同期をデータロード時に行うように変更。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- NetworkTime がシステムクロックに依存しないようにし、システムクロックの誤差が Exchange ネットワークの時間推定に異常を引き起こす問題を解決。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### 問題修正

以下の問題を修正しました：

- 頻繁に TRUNCATE される小さなテーブルで低カーディナリティ辞書最適化を適用するとクエリエラーが発生する問題。[#23185](https://github.com/StarRocks/starrocks/pull/23185)
- UNION を含み、最初の子が NULL 定数の View をクエリすると、BE がクラッシュする問題。[#13792](https://github.com/StarRocks/starrocks/pull/13792)  
- Bitmap Index を使用したクエリが特定の状況で誤った結果を返す問題。[#23484](https://github.com/StarRocks/starrocks/pull/23484)
- BE での ROUND DOUBLE/FLOAT を DECIMAL に変換する処理が FE と一致しない問題。[#23152](https://github.com/StarRocks/starrocks/pull/23152)
- Schema change とデータインポートが同時に行われると、Schema change が時々停止する問題。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- Broker Load、Spark Connector、または Flink Connector を使用して Parquet ファイルをインポートすると、時々 BE が OOM になる問題。[#25254](https://github.com/StarRocks/starrocks/pull/25254)
- クエリステートメントで ORDER BY の後に定数があり、LIMIT がある場合に `unknown error` というエラーが発生する問題。[#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

リリース日： 2023年6月1日

### 機能最適化

- `thrift_server_max_worker_threads` の設定が小さすぎるために INSERT INTO ... SELECT がタイムアウトするシナリオでのエラーメッセージを最適化。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- 複数のテーブルを結合する際に `bitmap_contains` 関数を使用するとメモリ消費が減少し、パフォーマンスが向上する。[#20617](https://github.com/StarRocks/starrocks/pull/20617)、[#20653](https://github.com/StarRocks/starrocks/pull/20653)

### 問題修正

以下の問題を修正しました：

- Truncate 操作がパーティション名の大文字と小文字を区別するため、Truncate Partition が失敗する問題。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- Parquet 形式のファイルに int96 timestamp 型のデータが含まれていると、データがオーバーフローする問題。[#22355](https://github.com/StarRocks/starrocks/issues/22355)
- 物化ビューを削除した後に DECOMMISSION を使用して BE ノードをオフラインにすると失敗する問題。[#22743](https://github.com/StarRocks/starrocks/issues/22743)
- クエリの実行計画に BroadcastJoin ノードから BucketShuffleJoin ノードへの経路が含まれ、例えば `SELECT * FROM t1 JOIN [Broadcast] t2 ON t1.a = t2.b JOIN [Bucket] t3 ON t2.b = t3.c;` のようなクエリで、BroadcastJoin の左表の等価 Join のキーのデータが BucketShuffleJoin の前に削除された場合、BE がクラッシュする問題。[#23227](https://github.com/StarRocks/starrocks/pull/23227)
- クエリの実行計画に CrossJoin ノードから HashJoin ノードへの経路が含まれ、fragment instance 内の HashJoin の右表が空の場合、結果が正しくない問題。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- 物化ビューが一時パーティションの作成に失敗し、BE のオフラインが停止する問題。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- SQL ステートメント内の STRING 型の値に複数のエスケープ文字が含まれている場合、SQL ステートメントの解析に失敗する問題。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- パーティション列の最大値のデータをクエリできない問題。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocks 2.4 から 2.3 にロールバックした後、インポートジョブでエラーが発生する問題。[#23642](https://github.com/StarRocks/starrocks/pull/23642)
- 列のトリミング再利用の問題。[#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

リリース日： 2023年4月25日

### 機能最適化

式の戻り値が Boolean 値に合法的に変換できる場合、暗黙的な変換が行われます。[#21792](https://github.com/StarRocks/starrocks/pull/21792)

### 問題修正

以下の問題を修正しました：

- ユーザーの LOAD 権限がテーブルレベルである場合、インポートジョブが失敗した後のトランザクションのロールバック時に `Access denied; you need (at least one of) the LOAD privilege(s) for this operation` というエラーメッセージが表示される問題。[#21129](https://github.com/StarRocks/starrocks/issues/21129)
- ALTER SYSTEM DROP BACKEND を実行して BE を削除した後、その BE 上のテーブルの2つのレプリカが修復できず、結果としてインポートジョブが利用可能なレプリカがないために失敗する問題。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- サポートされていないデータ型を使用してテーブルを作成する際に、誤ったエラーメッセージが表示される問題。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- Broadcast Join のショートサーキットロジックが異常で、クエリ結果が正しくない問題。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 物化ビューを使用した後のディスク使用率が大幅に増加する問題。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- Audit Loader プラグインが完全にアンインストールされない問題。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT` の結果として表示される行数と `SELECT COUNT(*) FROM XXX` の結果として表示される行数が一致しない問題。[#20084](https://github.com/StarRocks/starrocks/issues/20084)

- サブクエリがウィンドウ関数を使用し、親クエリがGROUP BY句を使用する場合、クエリ結果は集約できません。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- BEを起動した後、BEプロセスは存在しますが、BEの全てのポートが開けません。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- ディスクIOの利用率が高すぎると、プライマリキーモデルのテーブルのトランザクションコミットが遅くなり、そのテーブルをクエリすると`backend not found`のエラーが返される可能性があります。[#18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

リリース日： 2023年3月28日

### 機能最適化

- 複雑なクエリは多数の`ColumnRefOperators`を引き起こします。以前は`BitSet`を使用して`ColumnRefOperator::id`を格納していましたが、メモリを多く消費します。メモリ使用量を減らすために、現在は`RoaringBitMap`を使用して`ColumnRefOperator::id`を格納します。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 新しいI/Oスケジューリング戦略を追加し、大規模クエリが小規模クエリのパフォーマンスに与える影響を軽減します。新しいI/Oスケジューリング戦略を有効にするには、**be.conf**でBEの静的パラメータ`pipeline_scan_queue_mode=1`を設定し、BEを再起動する必要があります。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### 問題修正

以下の問題を修正しました：

- テーブルの期限切れデータが正常に回収されない場合、そのテーブルはより多くのディスク容量を占めます。[#19796](https://github.com/StarRocks/starrocks/pull/19796)
- Broker Loadを使用してParquetファイルをインポートする際に、NOT NULLの列に`NULL`値をインポートすると、エラーメッセージが不明確です。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 頻繁に多数の一時パーティションを作成して既存のパーティションを置き換えると、FEのメモリリークとFull GCが発生します。[#19283](https://github.com/StarRocks/starrocks/pull/19283)
- Colocationテーブルに対して、`ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`のコマンドで手動でレプリカの状態をbadに指定できますが、BEの数がレプリカの数以下の場合、そのレプリカは修復できません。[#19443](https://github.com/StarRocks/starrocks/pull/19443)
- Follower FEにINSERT INTO SELECTリクエストを送信するとき、パラメータ`parallel_fragment_exec_instance_num`が効果を発揮しません。[#18841](https://github.com/StarRocks/starrocks/pull/18841)
- 演算子`<=>`を使用して特定の値と`NULL`値を比較すると、比較結果が正しくありません。[#19210](https://github.com/StarRocks/starrocks/pull/19210)
- リソース隔離の並行制限が継続的にトリガーされると、クエリの並行数の指標がゆっくりと減少します。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 高並行インポート時に`"get database read lock timeout, database=xxx"`のエラーが発生する可能性があります。[#16748](https://github.com/StarRocks/starrocks/pull/16748) [#18992](https://github.com/StarRocks/starrocks/pull/18992)

## 2.3.10

リリース日： 2023年3月9日

### 機能最適化

`storage_medium`の推論メカニズムを最適化します。BEがSSDとHDDを同時にストレージメディアとして使用する場合、`storage_cooldown_time`が設定されていると、StarRocksは`storage_medium`を`SSD`に設定します。そうでない場合は、`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)

### 問題修正

以下の問題を修正しました：

- データレイクのParquetファイル内のARRAY型データをクエリすると、クエリが失敗する可能性があります。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- プログラムが開始したStream Loadインポートタスクが停止し、FEがプログラムからのHTTPリクエストを受信できません。[#18559](https://github.com/StarRocks/starrocks/pull/18559)
- Elasticsearch外部テーブルをクエリするとエラーが発生する可能性があります。[#13727](https://github.com/StarRocks/starrocks/pull/13727)
- 式が初期段階でエラーになると、BEがサービスを停止する可能性があります。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- クエリ文が空の配列リテラル`[]`を使用すると、クエリが失敗する可能性があります。[#18550](https://github.com/StarRocks/starrocks/pull/18550)
- バージョンが2.2以上から2.3.9以上にアップグレードされた後、Routine Loadを作成する際にCOLUMNに計算式が含まれていると、`No match for <expr> with operand types xxx and xxx`のエラーが発生する可能性があります。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BEを再起動した後、インポート操作が停止します。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- クエリ文のWHERE句にOR演算子が含まれている場合、余分なパーティションがスキャンされます。[#18610](https://github.com/StarRocks/starrocks/pull/18610)

## 2.3.9

リリース日： 2023年2月20日

### 問題修正

- Schema Changeのプロセス中にTablet Cloneが発生し、レプリカがあるBEノードが変更されると、Schema Changeが失敗する可能性があります。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- `group_concat()`関数の戻り値が切り捨てられます。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- Tencent Big Data Suite（TBDS）を使用してHDFSからデータをインポートする際に`invalid hadoop.security.authentication.tbds.securekey`エラーが発生し、TBDSが提供するHDFS認証アクセスをサポートしていないことが示されます。[#14125](https://github.com/StarRocks/starrocks/pull/14125)、[#15693](https://github.com/StarRocks/starrocks/pull/15693)
- 一部の状況で、CBOが演算子が等しいかどうかのロジックに誤りがあります。[#17227](https://github.com/StarRocks/starrocks/pull/17227)、[#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 非Leader FEノードに接続し、`USE <catalog_name>.<database_name>`リクエストを送信すると、非LeaderノードがリクエストをLeader FEに転送する際に`catalog_name`を転送せず、Leader FEが`default_catalog`を使用するため、対応するデータベースが見つかりません。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

リリース日： 2023年2月2日

### 問題修正

以下の問題を修正しました：

- 大規模クエリが完了した後にリソースを解放すると、他のクエリが遅くなる可能性があります。特にリソースグループが有効になっている場合や大規模クエリが異常終了した場合です。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- 主キーモデルのテーブルに対して、あるレプリカのメタデータバージョンが遅れている場合、インクリメンタルクローンによって多くのバージョンのメタデータが溜まり、タイムリーにGCされないことがあり、BEでOOMが発生する可能性があります。 [#15935](https://github.com/StarRocks/starrocks/pull/15935)
- FEがBEに一度きりのハートビートを送信し、ハートビート接続がタイムアウトすると、FEはそのBEが利用不可であると判断し、最終的にそのBE上でのトランザクションが失敗します。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- StarRocksの外部テーブル機能を使用してデータをインポートする際に、ソースStarRocksクラスタが低バージョンで、ターゲットStarRocksクラスタが高バージョンの場合（高バージョンが2.2.8〜2.2.11、2.3.4〜2.3.7、2.4.1または2.4.2の場合）、データインポートが失敗します。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- クエリの高並行性とメモリ使用率が高い場合、BEがクラッシュする可能性があります。 [#16047](https://github.com/StarRocks/starrocks/pull/16047)
- テーブルに動的パーティションが有効になっており、一部のパーティションが動的に削除された場合、TRUNCATE TABLEを実行すると`NullPointerException`エラーが発生し、その時点でのデータインポートによってFEがクラッシュし、再起動できなくなります。[#16822](https://github.com/StarRocks/starrocks/pull/16822)

## 2.3.7

リリース日： 2022年12月30日

### 問題修正

以下の問題を修正しました：

- StarRocksのテーブルの一部の列がNULLを許容するように設定されている場合、そのテーブルを基にビューを作成すると、ビュー内のその列が誤ってNOT NULLとして設定されます。 [#15749](https://github.com/StarRocks/starrocks/pull/15749)
- 新しいTabletバージョンがインポート中に生成されたが、FEがまだ新しいTabletバージョンを認識していないため、その時点でFEが下したクエリ実行計画では、BEに対してそのTabletの過去のバージョンを読み取るよう要求します。その時点でガベージコレクションがその過去のバージョンを回収していた場合、その時点でのクエリはその過去のバージョンを読み取ることができず、"Not found: get_applied_rowsets(version xxxx) failed tablet:xxx #version:x [xxxxxx]"というエラーが発生します。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- 高頻度のインポート時にFEが多くのメモリを消費します。 [#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 集約および複数テーブルのJOINクエリに対して、統計情報が不正確に収集され、CROSS JOINが発生し、クエリの実行時間が長くなります。[#12067](https://github.com/StarRocks/starrocks/pull/12067) [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

リリース日： 2022年12月22日

### 機能最適化

- Pipeline実行エンジンがINSERT INTO文をサポートします。有効にするには、FEの設定項目`enable_pipeline_load_for_insert`を`true`に設定する必要があります。 [#14723](https://github.com/StarRocks/starrocks/pull/14723)
- 主キーモデルのCompactionフェーズで使用されるメモリを最適化します。[#13861](https://github.com/StarRocks/starrocks/pull/13861)、[#13862](https://github.com/StarRocks/starrocks/pull/13862)

### 問題修正

以下の問題を修正しました：

- 修正された問題：リソース隔離を有効にした後、複数のリソースグループが同時にクエリを実行すると、BEがクラッシュする可能性がある。[#14905](https://github.com/StarRocks/starrocks/pull/14905)
- 物化ビューの作成 `CREATE MATERIALIZED VIEW AS SELECT` では、`SELECT` クエリで集約関数を使用せずに GROUP BY を使用する場合（例：`CREATE MATERIALIZED VIEW test_view AS select a,b from test group by b,a order by a;`）、BEノードがすべてクラッシュする。[#13743](https://github.com/StarRocks/starrocks/pull/13743)
- 主キーモデルのテーブルに対して高頻度で INSERT INTO を実行し、データ変更後すぐに BE を再起動すると、再起動が遅くなる。[#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 環境に JRE のみがインストールされており JDK がインストールされていない場合、FE の再起動後にクエリが失敗する。修正後、その環境では FE を正常に再起動できず、`Error: JAVA_HOME cannot be jre` というエラーが直接表示されるため、環境に JDK をインストールする必要があります。[#14332](https://github.com/StarRocks/starrocks/pull/14332)
- クエリにより BE がクラッシュする。[#14221](https://github.com/StarRocks/starrocks/pull/14221)
- `exec_mem_limit` の設定時に式を使用することはサポートされていない。[#13647](https://github.com/StarRocks/starrocks/pull/13647)
- サブクエリの結果に基づいて同期リフレッシュされる物化ビューを作成する際に失敗する。[#13507](https://github.com/StarRocks/starrocks/pull/13507)
- Hive 外部テーブルの手動リフレッシュ後に、列のコメントがクリアされる。[#13742](https://github.com/StarRocks/starrocks/pull/13742)
- テーブル A と B を結合クエリする際に、まず大きな右表 B を計算し、その過程で左表 A のデータに Compaction が実行されると、BE ノードがクラッシュする。[#14070](https://github.com/StarRocks/starrocks/pull/14070)
- Parquet ファイルの列名が大文字と小文字を区別する場合、クエリ条件で Parquet ファイルの大文字の列名を使用すると、クエリ結果が空になる。[#13860](https://github.com/StarRocks/starrocks/pull/13860)、[#14773](https://github.com/StarRocks/starrocks/pull/14773)
- バルクインポート時に Broker の接続数が多すぎてデフォルトの最大接続数を超え、Broker の接続が切断され、最終的にインポートに失敗し、`list path error` というエラーが発生する。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- BE の負荷が高い時に、リソースグループの監視指標 `starrocks_be_resource_group_running_queries` の統計が誤っている。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- クエリ文で OUTER JOIN を使用すると、BE ノードがクラッシュする可能性がある。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- バージョン 2.4 で非同期物化ビューを使用した後にバージョン 2.3 にロールバックすると、FE が起動できなくなる。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- 主キーモデルのテーブルで `delete_range` を使用した部分があり、パフォーマンスが悪いと、RocksDB からのデータ読み取りが遅くなり、CPU リソースの使用率が高くなる可能性がある。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

### 行動変更

- FE のパラメータ `default_storage_medium` を廃止し、テーブルのストレージメディアをシステムが自動的に推測するように変更。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.3.5

リリース日：2022年11月30日

### 機能最適化

- Colocate Join が等価 Join をサポート。[#13546](https://github.com/StarRocks/starrocks/pull/13546)
- 高頻度インポート時に WAL レコードが連続して追加され、主キーインデックスファイルが過大になる問題を解決。[#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FE のバックグラウンドスレッドが全量検出 tablet 時に DB ロックを長時間保持する問題を最適化。[#13070](https://github.com/StarRocks/starrocks/pull/13070)

### 問題修正

以下の問題を修正しました：

- UNION ALL の上に作成されたビューで、UNION ALL の入力列に定数 NULL がある場合、ビューのスキーマが誤っている問題を修正。通常、列のタイプは UNION ALL のすべての入力列のタイプであるべきですが、実際には NULL_TYPE でした。[#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...` と `SELECT * FROM ... LIMIT ...` のクエリ結果が一致しない問題を修正。[#13585](https://github.com/StarRocks/starrocks/pull/13585)
- FE が外部テーブルの tablet メタデータを同期する際に、ローカルの tablet メタデータを上書きして Flink からのデータインポートに失敗する可能性がある問題を修正。[#12579](https://github.com/StarRocks/starrocks/pull/12579)
- Runtime Filter の null filter が定数リテラルを処理する際に BE ノードがクラッシュする問題を修正。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- CTAS の実行時にエラーが発生する問題を修正。[#12388](https://github.com/StarRocks/starrocks/pull/12388)
- 監査ログ (audit log) において、Pipeline Engine が収集した `ScanRows` に誤りがある問題を修正。[#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 圧縮形式の HIVE データをクエリした際に結果が正しくない問題を修正。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- BE ノードがクラッシュした後、クエリがタイムアウトし、応答が遅くなる問題を修正。[#12955](https://github.com/StarRocks/starrocks/pull/12955)
- Broker Load を使用してデータをインポートする際に Kerberos 認証に失敗するエラーが発生する問題を修正。[#13355](https://github.com/StarRocks/starrocks/pull/13355)
- 過多な OR 述語により統計情報の推定に時間がかかりすぎる問題を修正。[#13086](https://github.com/StarRocks/starrocks/pull/13086)
- Broker Load で大文字の列名を含む ORC ファイル（Snappy 圧縮）をインポートした後に BE ノードがクラッシュする問題を修正。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- Primary Key テーブルのエクスポートまたはクエリが 30 分を超えるとエラーが発生する問題を修正。[#13403](https://github.com/StarRocks/starrocks/pull/13403)
- broker を使用して HDFS にバックアップする際に、大量のデータが原因でバックアップに失敗する問題を修正。[#12836](https://github.com/StarRocks/starrocks/pull/12836)
- `parquet_late_materialization_enable` パラメータが Iceberg データの読み取り正確性に影響を与える問題を修正。[#13132](https://github.com/StarRocks/starrocks/pull/13132)
- ビューの作成時に `failed to init view stmt` エラーが発生する問題を修正。[#13102](https://github.com/StarRocks/starrocks/pull/13102)
- JDBC 経由で StarRocks に接続し SQL を実行する際にエラーが発生する問題を修正。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- バケットが多すぎると Tablet hint を使用したクエリがタイムアウトする問題を修正。[#13272](https://github.com/StarRocks/starrocks/pull/13272)
- BE ノードがクラッシュし再起動できなくなり、その状態でテーブルを作成した後のインポートジョブがエラーになる問題を修正。[#13701](https://github.com/StarRocks/starrocks/pull/13701)
- 物化ビューの作成によりすべての BE ノードがクラッシュする問題を修正。[#13184](https://github.com/StarRocks/starrocks/pull/13184)
- ALTER ROUTINE LOAD を実行して消費パーティションの offset を更新する際に `The specified partition 1 is not in the consumed partitions` エラーが発生し、最終的に Follower がクラッシュする問題を修正。[#12227](https://github.com/StarRocks/starrocks/pull/12227)

## 2.3.4

リリース日：2022年11月10日

### 機能最適化

- Routine Load の作成に失敗した際のエラーメッセージを最適化。[#12204](https://github.com/StarRocks/starrocks/pull/12204)
- Hive をクエリする際に CSV データの解析に失敗すると直接エラーが発生する。[#13013](https://github.com/StarRocks/starrocks/pull/13013)

### 問題修正

以下の問題を修正しました：

- HDFS のファイルパスに `()` が含まれているとクエリがエラーになる問題を修正。[#12660](https://github.com/StarRocks/starrocks/pull/12660)
- サブクエリに LIMIT があり、ORDER BY ... LIMIT ... OFFSET を使用して結果セットをソートするとクエリ結果が誤っている問題を修正。[#9698](https://github.com/StarRocks/starrocks/issues/9698)
- ORC ファイルをクエリする際に大文字と小文字が区別されない問題を修正。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- RuntimeFilter が正常に閉じられずに BE がクラッシュする問題を修正。[#12895](https://github.com/StarRocks/starrocks/pull/12895)
- 列を追加した直後にデータを削除するとクエリ結果が誤っている問題を修正。[#12907](https://github.com/StarRocks/starrocks/pull/12923)
- StarRocks と MySQL クライアントが同じローカルネットワークにない場合、KILL を一度実行しても INSERT INTO SELECT のインポートジョブを正常に終了できない問題を修正。[#11879](https://github.com/StarRocks/starrocks/pull/11897)
- 監査ログ (audit log) において、Pipeline Engine が収集した `ScanRows` に誤りがある問題を修正。[#12185](https://github.com/StarRocks/starrocks/pull/12185)

## 2.3.3

リリース日：2022年9月27日

### 問題修正

以下の問題を修正しました：

- Textfile 形式の Hive テーブルをクエリすると結果が不正確になる問題を修正。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- Parquet 形式のファイルをクエリする際に、ARRAY のネストをサポートしていない問題を修正。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- 外部データソースと StarRocks テーブルを読み取るクエリ、または外部データソースと StarRocks テーブルを並行して読み取るクエリが一つのリソースグループにルーティングされると、クエリがタイムアウトする可能性がある問題を修正。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- Pipeline 実行エンジンをデフォルトで有効にした後、パラメータ `parallel_fragment_exec_instance_num` が 1 になり、INSERT INTO によるインポートが遅くなる問題を修正。[#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 式が初期段階でエラーになると BE がサービスを停止する可能性がある問題を修正。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- 修正了执行 ORDER BY LIMIT 时导致堆溢出的问题。 [#11185](https://github.com/StarRocks/starrocks/pull/11185)
- 修正了重启 Leader FE 时导致 schema change 执行失败的问题。 [#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

发布日期：2022年9月7日

### 新特性

- 支持延迟物化，提升小范围过滤场景下 Parquet 外表的查询性能。 [#9738](https://github.com/StarRocks/starrocks/pull/9738)
- 新增 SHOW AUTHENTICATION 语句，用于查询用户认证相关的信息。 [#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 功能优化

- 查询配置了分桶的 Hive 表时，支持设置是否递归遍历该分桶 Hive 表的所有数据文件。 [#10239](https://github.com/StarRocks/starrocks/pull/10239)
- 将资源组类型 `realtime` 更改为 `short_query`。 [#10247](https://github.com/StarRocks/starrocks/pull/10247)
- 优化外表查询机制，在查询 Hive 外表时默认忽略大小写。[#10187](https://github.com/StarRocks/starrocks/pull/10187)

### 问题修复

修复了以下问题：

- 修正了 Elasticsearch 外表在多个 Shard 下查询可能意外退出的问题。 [#10369](https://github.com/StarRocks/starrocks/pull/10369)
- 修正了重写子查询为公用表表达式 (CTE) 时出现错误的问题。 [#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 修正了导入大量数据时出现错误的问题。[#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 修正了多个 Catalog 配置相同 Thrift 服务地址时，删除其中一个 Catalog 会导致其他 Catalog 的增量元数据同步失效的问题。 [#10511](https://github.com/StarRocks/starrocks/pull/10511)
- 修正了 BE 内存占用统计不准确的问题。 [#9837](https://github.com/StarRocks/starrocks/pull/9837)
- 修正了完整克隆 (Full Clone) 后，查询主键模型表时出现错误的问题。[#10811](https://github.com/StarRocks/starrocks/pull/10811)
- 修正了拥有逻辑视图的 SELECT 权限但无法查询的问题。[#10563](https://github.com/StarRocks/starrocks/pull/10563)
- 修正了逻辑视图命名无限制的问题。逻辑视图的命名规范与数据库表的命名规范一致。[#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 行为变更

- 新增 BE 配置项 `max_length_for_bitmap_function`（默认为1000000字节）和 `max_length_for_to_base64`（默认为200000字节），以控制 bitmap 函数和 to_base64() 函数的输入值的最大长度。[#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

发布日期：2022年8月22日

### 功能优化

- Broker Load 现支持将 Parquet 文件的 List 列转换为非嵌套 ARRAY 数据类型。[#9150](https://github.com/StarRocks/starrocks/pull/9150)
- 优化了 JSON 类型相关函数（json_query、get_json_string 和 get_json_int）的性能。[#9623](https://github.com/StarRocks/starrocks/pull/9623)
- 优化报错信息：查询 Hive、Iceberg、Hudi 外表时，如果查询的数据类型未被支持，系统会针对相应的列报错。[#10139](https://github.com/StarRocks/starrocks/pull/10139)
- 降低资源组调度延迟，优化资源隔离性能。[#10122](https://github.com/StarRocks/starrocks/pull/10122)

### 问题修复

修复了以下问题：

- 修正了查询 Elasticsearch 外表时，`limit` 算子下推导致返回错误结果的问题。[#9952](https://github.com/StarRocks/starrocks/pull/9952)
- 修正了使用 `limit` 算子查询 Oracle 外表失败的问题。[#9542](https://github.com/StarRocks/starrocks/pull/9542)
- 修正了 Routine Load 中 Kafka Broker 停止工作导致 BE 卡死的问题。[#9935](https://github.com/StarRocks/starrocks/pull/9935)
- 修正了查询与外表数据类型不匹配的 Parquet 文件导致 BE 停止工作的问题。[#10107](https://github.com/StarRocks/starrocks/pull/10107)
- 修正了外表 Scan 范围为空导致查询阻塞超时的问题。[#10091](https://github.com/StarRocks/starrocks/pull/10091)
- 修正了子查询中包含 ORDER BY 子句时系统报错的问题。[#10180](https://github.com/StarRocks/starrocks/pull/10180)
- 修正了并发重载 Hive 元数据导致 Hive Metastore 挂起的问题。[#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

发布日期：2022年7月29日

### 新特性

- 主键模型现支持完整的 DELETE WHERE 语法。相关文档，请参见 [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-与主键类型表)。
- 主键模型支持持久化主键索引，基于磁盘而不是内存维护索引，大幅降低内存使用。相关文档，请参见[主键模型](../table_design/table_types/primary_key_table.md#使用说明)。
- 全局低基数字典优化支持实时数据导入，实时场景下字符串数据的查询性能提升一倍。
- 支持以异步方式执行 CTAS，并将结果写入新表。相关文档，请参见 [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- 资源组相关功能：
  - 支持监控资源组：可在审计日志中查看查询所属的资源组，并通过相关 API 获取资源组的监控信息。相关文档，请参见[监控指标](../administration/Monitor_and_Alert.md#监控指标)。
  - 支持限制大查询的 CPU、内存或 I/O 资源；可通过匹配分类器将查询路由至资源组，或者设置会话变量直接为查询指定资源组。相关文档，请参见[资源隔离](../administration/resource_group.md)。
- 支持 JDBC 外表，可以轻松访问 Oracle、PostgreSQL、MySQL、SQLServer、ClickHouse 等数据库，并且查询时支持谓词下推，提高查询性能。相关文档，请参见 [更多数据库（JDBC）的外部表](../data_source/External_table.md#更多数据库jdbc的外部表)。
- 【プレビュー】发布全新数据源 Connector 框架，支持创建外部数据目录（External Catalog），从而无需创建外部表即可直接查询 Apache Hive™。相关文档，请参见[查询外部数据](../data_source/catalog/query_external_data.md)。
- 新增以下函数：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)、[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)、[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 功能优化

- 优化合并机制（Compaction），对较大的元数据进行合并操作，避免因数据高频更新而导致短时间内元数据挤压，占用较多磁盘空间。
- 优化导入 Parquet 文件和压缩文件格式的性能。
- 优化创建物化视图的性能，在部分场景下创建速度提升近10倍。
- 优化算子性能：
  - TopN、sort 算子。
  - 包含函数的等值比较运算符下推至 scan 算子时，支持使用 Zone Map 索引。
- 优化 Apache Hive™ 外表功能。
  - 当 Apache Hive™ 的数据存储采用 Parquet、ORC、CSV 格式时，支持 Hive 表执行 ADD COLUMN、REPLACE COLUMN 等表结构变更（Schema Change）。相关文档，请参见 [Hive 外部表](../data_source/External_table.md#deprecated-hive-外部表)。
  - 支持 Hive 资源修改 `hive.metastore.uris`。相关文档，请参见 [ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)。
- 优化 Apache Iceberg 外表功能，创建 Iceberg 资源时支持使用自定义目录（Catalog）。相关文档，请参见 [Apache Iceberg 外表](../data_source/External_table.md#deprecated-iceberg-外部表)。
- 优化 Elasticsearch 外表功能，支持取消探测 Elasticsearch 集群数据节点的地址。相关文档，请参见 [Elasticsearch 外部表](../data_source/External_table.md#deprecated-elasticsearch-外部表)。
- 当 sum() 中输入的值为 STRING 类型且为数字时，进行自动隐式转换。
- year、month、day 函数支持 DATE 数据类型。

### 问题修复

修复了以下问题：

- 修正了 Tablet 过多导致 CPU 占用率过高的问题。
- 修正了出现 "Fail to Prepare Tablet Reader" 报错提示的问题。
- 修正了 FE 重启失败的问题。[#5642](https://github.com/StarRocks/starrocks/issues/5642)、[#4969](https://github.com/StarRocks/starrocks/issues/4969)、[#5580](https://github.com/StarRocks/starrocks/issues/5580)
- 修正了 CTAS 语句中调用 JSON 函数时报错的问题。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### 其他

- 【Preview】提供集群管理工具 StarGo，提供集群部署、启停、升级、回滚、多集群管理等多种能力。相关文档，请参见[通过 StarGo 部署 StarRocks 集群](../administration/stargo.md)。
- 部署或者升级至 2.3 版本，默认开启 Pipeline 执行引擎，预期在高并发小查询、复杂大查询场景下获得明显的性能优势。如果使用 2.3 版本时遇到明显的性能回退，则可以通过设置 `SET GLOBAL enable_pipeline_engine = false;` 来关闭 Pipeline 执行引擎。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) 语句兼容 MySQL 语法，显示授权 GRANT 语句。
- シングル Schema Change タスクのデータが使用するメモリ上限 `memory_limitation_per_thread_for_schema_change`(BE 設定項目)はデフォルト値の 2 GB に保つことをお勧めします。データが上限を超えた場合はディスクに書き込まれます。以前にこのパラメータを増やした場合は、2 GB に戻すことをお勧めします。そうしないと、シングル Schema Change タスクが大量のメモリを消費する問題が発生する可能性があります。

### アップグレード時の注意点

アップグレード後に問題が発生し、ロールバックが必要な場合は、**fe.conf** ファイルに `ignore_unknown_log_id=true` を追加してください。これは新しいバージョンのメタデータログに新しいタイプが追加されたためで、このパラメータを追加しないとロールバックできません。チェックポイントを完了した後に `ignore_unknown_log_id=false` に設定し、FE を再起動して通常の設定に戻すことをお勧めします。
