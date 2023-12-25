---
displayed_sidebar: Chinese
---
# StarRocks バージョン 2.1

## 2.1.13

リリース日：2022年9月6日

### 機能最適化

- BE設定項目 `enable_check_string_lengths` を追加し、インポート時のデータ長チェックを制御し、VARCHAR型データのオーバーフローがCompaction失敗を引き起こす問題を解決します。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- OR句が1000個以上含まれるクエリのパフォーマンスを最適化しました。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### 問題修正

以下の問題を修正しました：

- 集約モデルテーブルでARRAY型の列をクエリする際、その列でREPLACE_IF_NOT_NULL集約関数を使用している場合、クエリがエラーになり、BEが停止する可能性があります。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- クエリにIFNULL関数が1つ以上ネストしている場合、クエリ結果が正しくありません。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 動的に作成されたパーティションをTruncateした後、そのバケット数が動的パーティション設定のバケット数からデフォルトのバケット数に変わります。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- Routine Loadを使用してインポートする際にKafkaサービスがオフラインになると、StarRocksクラスターが一時的なデッドロックに陥り、クエリに影響を与える可能性があります。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- クエリ文にサブクエリとORDER BY句が同時に存在する場合、エラーが発生します。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

リリース日：2022年8月9日

### 機能最適化

`bdbje_cleaner_threads`と`bdbje_replay_cost_percent`の2つのパラメータを追加し、BDB JEのメタデータのクリーンアップを加速します。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### 問題修正

以下の問題を修正しました：

- 一部のクエリがLeader FEノードに転送され、`/api/query_detail`エンドポイントを介して取得されたSQLステートメントの実行情報が正しくない可能性があります。例えばSHOW FRONTENDSステートメント。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- BEを停止した後、現在のプロセスが完全に終了していないため、BEの再起動に失敗します。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 複数のBroker Loadジョブが同時に同じHDFSファイルからデータをインポートする場合、1つのジョブに異常が発生すると、他のジョブもデータを正常に読み取れず、最終的に失敗する可能性があります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- テーブル構造の変更後、関連する変数がリセットされず、`no delete vector found tablet`というエラーでテーブルのクエリに失敗します。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

リリース日：2022年7月9日

### 問題修正

以下の問題を修正しました：

- 主キーモデル（Primary Key）テーブルに高頻度でデータをインポートすると、処理が停止し、続行できなくなります。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 低カーディナリティ最適化で、集約式の順序処理が誤っているため、`count distinct`関数が一部の誤った結果を返します。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT句のトリミングルールの処理が誤っており、LIMIT句の実行後に結果が出ません。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- クエリのJoin条件にグローバル低カーディナリティ辞書最適化が行われている場合、クエリ結果が誤っています。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

リリース日：2022年6月24日

### 問題修正

以下の問題を修正しました：

- Leader FEノードを繰り返し切り替えると、すべてのインポートジョブが一時停止し、インポートができなくなる可能性があります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- `DESC`を使用してテーブル構造を表示すると、DECIMAL(18,2)型のフィールドがDECIMAL64(18,2)型として表示されます。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- インポートデータに偏りがある場合、一部の列が大量のメモリを消費し、MemTableのメモリ推定が4GBを超える可能性があり、その結果BEが停止します。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- Compactionの入力行数が多い場合、max_rows_per_segmentの内部推定がオーバーフローを起こし、結果として多数の小さなSegmentファイルが作成されます。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

リリース日：2022年6月9日

### 機能最適化

- スキーマ変更（Schema Change）などの内部処理の並行制御を最適化し、FEメタデータへの圧力を低減し、高並行性、大量データインポートシナリオで発生しやすいインポートのバックログや遅延を減少させます。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 高頻度インポートのパフォーマンスを最適化しました。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### 問題修正

以下の問題を修正しました：

- Routine LoadタスクにALTER操作を行った後、ALTER操作がLOADステートメントの全情報を記録していないため、Checkpoint完了後にこのインポートタスクのメタデータが失われます。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- Routine Loadタスクを停止するとデッドロックが発生する可能性があります。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- BEがデータをインポートする際、デフォルトでUTC+8タイムゾーンを使用します。ユーザーのマシンのタイムゾーンがUTCの場合、Spark Loadを使用してインポートされたデータのDateTime列は8時間増加します。[#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING関数は、非JSON string型の値を処理できません。抽出対象の値がJSONオブジェクトまたはJSON配列であり、JSON string型ではない場合、この関数はNULLを返します。現在の最適化では、JSON形式のSTRING型データを返します。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- データ量が多い場合、スキーマ変更（Schema Change）を行うと、メモリ消費が多すぎて失敗する可能性があります。現在、スキーマ変更の各段階でのメモリ使用量を制限することができます。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- Compactionを行う際、ある列の任意の値が0x40000000回以上繰り返されると、Compactionが停止します。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- BDB JE v7.3.8バージョンにはいくつかの問題があり、FEが起動した後にディスクI/Oが高くなり、ディスク使用率が異常に増加し、回復の兆しがないため、BDB JE v7.3.7バージョンに戻すとFEが正常に戻ります。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

リリース日：2022年5月26日

### 機能最適化

フレームが`ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`に設定されたウィンドウ関数について、計算中にあるパーティションが非常に大きい場合、システムはそのパーティションのすべてのデータをキャッシュしてから計算を行い、大量のメモリを消費します。現在の最適化では、このような場合にパーティションのすべてのデータをキャッシュしないようにします。[#5829](https://github.com/StarRocks/starrocks/issues/5829)

### 問題修正

以下の問題を修正しました：

- 主キーモデルテーブルにデータをインポートする際、システム内部で保存されている各データバージョンに対応する作成時間が厳密に増加していない場合（例えば、システム時間が前に調整されたり、関連する未知のバグがあるため）、処理が誤って行われ、BEが停止します。[#6046](https://github.com/StarRocks/starrocks/issues/6046)

- 一部のグラフィカルインターフェースツールは`set_sql_limit`変数を自動的に設定し、SQLステートメントのORDER BY LIMITが無視され、結果として返されるデータ行数が正しくありません。[#5966](https://github.com/StarRocks/starrocks/issues/5966)

- DROP SCHEMAステートメントを実行すると、データベースが直接強制的に削除され、削除されたデータベースは復元できません。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

- JSON形式のデータをインポートする際、JSON形式にエラーがある場合（例えば、複数のキーバリューペアの間にカンマ「,」がないなど）、BEが停止します。[#6098](https://github.com/StarRocks/starrocks/issues/6098)

- 高並行性インポートシナリオで、BEがディスクに書き込むタスクの数がバックログになり、BEが停止する可能性があります。[#3877](https://github.com/StarRocks/starrocks/issues/3877)

- 表構造変更を行う前に、システムはまずメモリの見積もりを行います。もし該当テーブルにSTRING型のフィールドが多い場合、メモリの見積もり結果が不正確になることがあります。このような状況で、見積もりされたメモリが単一のテーブル構造変更操作で許可されているメモリ上限を超えると、本来正常に実行できるはずのテーブル構造変更操作がエラーを報告することになります。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- 主キーモデルのテーブルが構造変更を経た後、データのインポート時に "duplicate key xxx" のエラーが発生する可能性があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- Shuffle Joinを行う際、低カーディナリティ最適化を使用した場合、パーティションエラーが発生する可能性があります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- Colocation Groupに含まれるテーブルが多く、インポート頻度も高い場合、そのColocation Groupが`stable`状態を維持できず、結果としてJOIN文でColocate Joinが使用できなくなることがあります。現在はデータインポート時に少し待つように最適化されており、これによりインポートされたTabletのレプリカの完全性をできるだけ保証するようにしています。

## 2.1.6

リリース日：2022年5月10日

### 問題修正

以下の問題を修正しました：

- 複数のDELETE操作を行った後、クエリ時にシステム内部で低カーディナリティ最適化を使用している場合、クエリ結果が誤っている可能性があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- データ書き込みの特定の段階で、Tabletが移行を行い完了した場合、データは引き続き元のTabletに対応するディスクに書き込まれ、データの損失が発生し、それによりクエリエラーが発生する可能性があります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- DECIMALとSTRING型の相互変換時に、精度エラーが発生する可能性があります。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- DECIMALとBIGINT型の乗算時に、計算結果がオーバーフローする可能性があります。今回はいくつかの調整と最適化を行いました。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

リリース日：2022年4月27日

### 問題修正

以下の問題を修正しました：

- 以前のDecimal乗算がオーバーフローし、計算結果が誤っていた問題を修正し、Decimal乗算がオーバーフローした場合にはNULLを返すようにしました。
- 統計情報の誤差が大きい場合、実行計画でColocate Joinの優先度がBroadcast Joinよりも低くなり、実際の実行時にColocate Joinが有効にならない可能性があります。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 4つ以上のテーブルをJoinする際に、複雑な式の計画が誤っているため、クエリエラーが発生する可能性があります。
- Shuffle Joinの下で、Shuffleする列が低カーディナリティ列である場合、BEがサービスを停止する可能性があります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- SPLIT関数にNULLパラメータを使用すると、BEがサービスを停止する可能性があります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

リリース日：2022年4月8日

### 新機能

- `UUID_NUMERIC`関数を新たに追加し、LARGEINT型の値を返します。`UUID`関数と比較して、実行性能が約2桁向上しました。

### 問題修正

以下の問題を修正しました：

- 列の削除、パーティションの追加、およびTabletのクローン後に、新旧のTabletの列Unique IDが一致しない可能性があり、システムが共有のTablet Schemaを使用しているため、BEがサービスを停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks外部テーブルへのデータインポート時に、設定されたターゲットStarRocksクラスタのFEがLeaderでない場合、FEがサービスを停止する可能性があります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- `CAST`関数がStarRocks 1.19版と2.1版で実行結果が異なる問題を修正しました。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- 明細モデルのテーブルで構造変更と物化ビューの作成を同時に行うと、データクエリエラーが発生する可能性があります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

リリース日：2022年3月19日

### 問題修正

以下の問題を修正しました：

- バッチでpublish versionを改善することにより、BEがダウンすることでデータが失われる問題を解決しました。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 実行計画が不合理であるために、一部のクエリがメモリオーバーになる問題を修正しました。
- 異なるcompactionプロセスで分片レプリカのチェックサム(checksum)が一致しない問題を修正しました。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- JOIN reorder projectionが正しく処理されていないために、クエリがエラーになる可能性がある問題を修正しました。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

リリース日：2022年3月14日

### 問題修正

以下の問題を修正しました：

- 1.19から2.1へのアップグレード時に`chunk_size`が一致しないためにBEがクラッシュする問題を修正しました。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- 2.0から2.1へのアップグレードプロセス中にインポートが行われた場合、インポートタスクが失敗する可能性がある問題を修正しました。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 単一のtabletのテーブルで集約操作を行う際に、適切な実行計画を得られないためにクエリが失敗する問題を修正しました。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FEが低カーディナリティグローバル辞書最適化で情報を収集する際にデッドロックを引き起こす可能性がある問題を修正しました。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- デッドロックによりBEノードがフリーズし、クエリが失敗する問題を修正しました。
- `SHOW VARIABLES`コマンドのエラーによりBIツールが接続できない問題を修正しました。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

リリース日：2022年2月24日

### 新機能

- 【公開テスト中】外部テーブルを通じてApache Icebergデータレイク内のデータをクエリし、データレイクの高速分析を実現する機能をサポートします。TPC-Hテストセットの結果によると、Apache Icebergデータをクエリする際のStarRocksのクエリ速度はPrestoの**3 - 5倍**です。関連ドキュメントはこちらをご覧ください [Apache Iceberg 外部テーブル](../data_source/External_table.md#deprecated-hive-外部表)。
- 【公開テスト中】Pipeline実行エンジンをリリースし、クエリの並列度を自動調整できます。セッションレベルの変数`parallel_fragment_exec_instance_num`を手動で設定する必要はありません。また、一部の高並行性シナリオでは、以前のバージョンと比較して新バージョンのパフォーマンスが2倍に向上しています。
- CTAS（CREATE TABLE AS SELECT）をサポートし、クエリ結果に基づいてテーブルを作成し、データをインポートすることで、テーブル作成とETL操作を簡素化します。関連ドキュメントはこちらをご覧ください [CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)。
- SQLフィンガープリントをサポートし、遅いクエリの各種SQL文に対してSQLフィンガープリントを計算し、遅いクエリの迅速な特定を可能にします。関連ドキュメントはこちらをご覧ください [SQL フィンガープリント](../administration/Query_planning.md#sql-フィンガープリントの表示)。
- 新しい関数[ANY_VALUE](../sql-reference/sql-functions/aggregate-functions/any_value.md)、[ARRAY_REMOVE](../sql-reference/sql-functions/array-functions/array_remove.md)、ハッシュ関数[SHA2](../sql-reference/sql-functions/crytographic-functions/sha2.md)を追加しました。

### 機能最適化

- Compactionのパフォーマンスを最適化し、10000列のデータのインポートをサポートします。
- StarRocksの最初のScanとPage Cacheのパフォーマンスを最適化しました。ランダムI/Oを低減することで、StarRocksの最初のScanのパフォーマンスを向上させ、最初のScanのディスクがSATAディスクである場合、特にパフォーマンスの向上が顕著です。さらに、StarRocksのPage Cacheは原始データを直接格納することをサポートし、Bitshuffleエンコーディングを必要としません。そのため、StarRocksのPage Cacheを読み取る際に追加のデコードは不要で、キャッシュヒット率を向上させ、それによりクエリ効率を大幅に向上させます。
- 主キーモデル（Primary Key Model）のテーブル構造（Schema Change）の変更をサポートし、`ALTER TABLE`でインデックスの追加、削除、変更を実行できます。
- JSONインポートのパフォーマンスを最適化し、JSONインポートで単一のJSONファイルのサイズが100MBを超えないという制限を撤廃しました。
- Bitmap Indexのパフォーマンスを最適化しました。
- 外部テーブルを通じてHiveデータを読み取るパフォーマンスを最適化し、HiveのストレージフォーマットがCSVであることをサポートします。
- テーブル作成ステートメントでのタイムスタンプフィールドをDEFAULT CURRENT_TIMESTAMPとして定義することをサポートします。
- 複数のデリミタを持つCSVファイルのインポートをサポートします。
- flink-source-connectorはFlinkによるStarRocksデータのバッチ読み取りをサポートし、BEノードへの直接並行読み取り、自動述語のプッシュダウンなどの特徴を実現しました。関連ドキュメントはこちらをご覧ください [Flink Connector](../unloading/Flink_connector.md)。

### 問題修正

- JSON形式データのインポートでjsonpathsを設定した後に__opフィールドを自動的に認識できない問題を修正しました。
- Broker Loadでデータをインポートする過程でソースデータが変更されたためにBEノードがダウンする問題を修正しました。
- 物化ビューを作成した後、一部のSQL文がエラーになる問題を修正しました。

- 修正了 Routine Load 中使用带引号的 jsonpaths 时会报错的问题。
- 修正了查询列数超过 200 列时，并发性能明显下降的问题。

### その他

Colocation Group を無効にする HTTP Restful API を変更しました。より直感的に理解できるように、Colocation Group を無効にする API を `POST /api/colocate/group_unstable` に変更しました（旧インターフェースは `DELETE /api/colocate/group_stable` でした）。

> Colocation Group を再度有効にする場合は、API `POST /api/colocate/group_stable` を使用できます。
