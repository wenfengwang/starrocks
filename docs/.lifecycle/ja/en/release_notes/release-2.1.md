---
displayed_sidebar: English
---

# StarRocks バージョン 2.1

## 2.1.13

リリース日: 2022年9月6日

### 改善点

- 読み込まれたデータの長さをチェックするBE設定項目 `enable_check_string_lengths` を追加しました。このメカニズムは、VARCHARデータサイズが範囲外になることによるコンパクション失敗を防ぐのに役立ちます。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- クエリに1000個以上のOR演算子が含まれる場合のクエリパフォーマンスを最適化しました。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### バグ修正

以下のバグが修正されました：

- 集約テーブルを使用してテーブルからARRAY列（REPLACE_IF_NOT_NULL関数を使用して計算された）をクエリする際に、エラーが発生し、BEがクラッシュすることがあります。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- クエリに複数のIFNULL()関数がネストされている場合、クエリ結果が不正確になります。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 動的パーティションが切り捨てられた後、パーティション内のタブレット数が動的パーティショニングによって設定された値からデフォルト値に変更されます。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- Routine Loadを使用してStarRocksにデータをロードする際にKafkaクラスタが停止していると、デッドロックが発生し、クエリパフォーマンスに影響を与える可能性があります。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- クエリにサブクエリとORDER BY句が含まれている場合、エラーが発生します。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

リリース日: 2022年8月9日

### 改善点

BDB JEでのメタデータクリーンアップを高速化するための2つのパラメータ `bdbje_cleaner_threads` と `bdbje_replay_cost_percent` を追加しました。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### バグ修正

以下のバグが修正されました：

- 一部のクエリがLeader FEに転送され、`/api/query_detail` アクションがSHOW FRONTENDSなどのSQLステートメントに関する誤った実行情報を返します。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- BEが終了した後、現在のプロセスが完全に終了しないため、BEの再起動が失敗します。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 同じHDFSデータファイルをロードするために複数のBroker Loadジョブが作成された場合、1つのジョブが例外に遭遇すると、他のジョブがデータを適切に読み取れず、結果的に失敗する可能性があります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- テーブルのスキーマが変更されたときに関連する変数がリセットされず、テーブルをクエリする際にエラー（`no delete vector found tablet`）が発生します。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

リリース日: 2022年7月9日

### バグ修正

以下のバグが修正されました：

- 主キーテーブルのテーブルにデータをロードする際、そのテーブルに頻繁にデータロードが行われると、ロードが中断されます。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 低カーディナリティ最適化中に集約式が誤った順序で処理され、`count distinct`関数が予期しない結果を返すことがあります。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT句による結果が返されないことがあります。これは、句内のプルーニングルールが適切に処理されないためです。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 低カーディナリティ最適化のグローバル辞書が、クエリの結合条件として定義された列に適用されると、クエリが予期しない結果を返すことがあります。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

リリース日: 2022年6月24日

### バグ修正

以下のバグが修正されました：

- Leader FEノードを繰り返し切り替えると、すべてのロードジョブがハングアップし、失敗する可能性があります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- DECIMAL(18,2)型のフィールドが、`DESC` SQLでテーブルスキーマをチェックする際にDECIMAL64(18,2)として表示されます。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- MemTableのメモリ使用量の見積もりが4GBを超えると、BEがクラッシュすることがあります。これは、ロード中のデータスキューにより、一部のフィールドが大量のメモリリソースを占有する可能性があるためです。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- コンパクションに多数の入力行がある場合、max_rows_per_segmentの計算のオーバーフローにより、多数の小さなセグメントファイルが作成されます。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

リリース日: 2022年6月9日

### 改善点

- スキーマ変更などの内部処理ワークロードに使用される並行制御メカニズムが最適化され、フロントエンド（FE）メタデータへの圧力が軽減されました。これにより、大量のデータを同時にロードするロードジョブが積み重なって速度が低下する可能性が低くなります。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 高頻度でデータをロードする際のStarRocksのパフォーマンスが向上しました。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### バグ修正

以下のバグが修正されました：

- ALTER操作ログはLOADステートメントに関するすべての情報を記録していません。そのため、ルーチンロードジョブにALTER操作を実行した後、チェックポイントが作成された後にジョブのメタデータが失われます。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- ルーチンロードジョブを停止するとデッドロックが発生する可能性があります。[#6450](https://github.com/StarRocks/starrocks/issues/6450)

- デフォルトでは、バックエンド (BE) はロードジョブにデフォルトの UTC+8 タイムゾーンを使用します。サーバーが UTC タイムゾーンを使用している場合、Spark ロードジョブを使用してロードされたテーブルの DateTime 列のタイムスタンプには 8 時間が加算されます。 [#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING 関数は非JSON文字列を処理できません。JSON オブジェクトまたは配列から JSON 値を抽出する場合、関数は NULL を返します。この関数は JSON オブジェクトまたは配列に対して同等の JSON 形式の STRING 値を返すよう最適化されています。 [#6426](https://github.com/StarRocks/starrocks/issues/6426)
- データ量が多い場合、過剰なメモリ消費によりスキーマ変更が失敗する可能性があります。スキーマ変更の全段階でメモリ消費の上限を指定できるように最適化が行われました。 [#6705](https://github.com/StarRocks/starrocks/pull/6705)
- コンパクションを行っているテーブルの列にある重複値の数が 0x40000000 を超えると、コンパクションは中断されます。 [#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FEが再起動された後、BDB JE v7.3.8 のいくつかの問題により高い I/O と異常に増加するディスク使用量が発生し、正常な状態に戻る兆候が見られません。FEは BDB JE v7.3.7 にロールバックした後、正常な状態に復旧します。 [#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

リリース日: 2022年5月26日

### 改善点

フレームが ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW に設定されているウィンドウ関数では、計算に関与するパーティションが大きい場合、StarRocks は計算を実行する前にパーティションの全データをキャッシュします。この状況では、大量のメモリリソースが消費されます。StarRocksは、この状況でパーティションの全データをキャッシュしないように最適化されています。 [#5829](https://github.com/StarRocks/starrocks/issues/5829)

### バグ修正

以下のバグが修正されました:

- プライマリキーテーブルを使用するテーブルにデータをロードする際、システム時刻が過去に戻るなどの理由でシステムに保存されている各データバージョンの作成時刻が単調に増加しない場合、データ処理エラーが発生し、バックエンド (BE) が停止する可能性があります。 [#6046](https://github.com/StarRocks/starrocks/issues/6046)
- 一部のグラフィカルユーザーインターフェース (GUI) ツールは set_sql_limit 変数を自動的に設定します。その結果、SQL ステートメントの ORDER BY LIMIT が無視され、クエリの結果として誤った行数が返されます。 [#5966](https://github.com/StarRocks/starrocks/issues/5966)
- データベースに対して DROP SCHEMA ステートメントを実行すると、データベースが強制的に削除され、復元することができません。 [#6201](https://github.com/StarRocks/starrocks/issues/6201)
- JSON 形式のデータをロードする際、データに JSON 形式のエラーが含まれている場合、BEは停止します。例えば、キーと値のペアがコンマ (,) で区切られていない場合です。[#6098](https://github.com/StarRocks/starrocks/issues/6098)
- 大量のデータを高い並行性でロードする際、ディスクへのデータ書き込みを行うタスクがバックエンド (BE) に蓄積されます。この状況では、BEが停止する可能性があります。 [#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocksは、テーブルに対するスキーマ変更を実行する前に必要なメモリ量を見積もります。テーブルに多くのSTRINGフィールドが含まれている場合、メモリ推定結果が不正確になる可能性があります。この状況で、推定される必要メモリ量が単一のスキーマ変更操作に許可される最大メモリ量を超えると、正常に実行されるべきスキーマ変更操作でエラーが発生します。 [#6322](https://github.com/StarRocks/starrocks/issues/6322)
- プライマリキーテーブルを使用するテーブルにスキーマ変更を行った後、そのテーブルにデータをロードする際に「重複キー xxx」というエラーが発生することがあります。 [#5878](https://github.com/StarRocks/starrocks/issues/5878)
- Shuffle Join 操作中に低カーディナリティ最適化を行うと、パーティションエラーが発生する可能性があります。 [#4890](https://github.com/StarRocks/starrocks/issues/4890)
- コロケーショングループ (CG) に多くのテーブルが含まれ、テーブルに頻繁にデータがロードされる場合、CGは安定状態を維持できないことがあります。この場合、JOIN ステートメントは Colocate Join 操作をサポートしません。StarRocksはデータロード中にもう少し待機するように最適化されており、データがロードされるタブレットレプリカの整合性を最大限に保つことができます。

## 2.1.6

リリース日: 2022年5月10日

### バグ修正

以下のバグが修正されました:

- 複数の DELETE 操作を実行した後にクエリを実行すると、低カーディナリティ列の最適化が行われた場合、誤ったクエリ結果が得られることがあります。 [#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 特定のデータインジェストフェーズでタブレットが移行されると、データはタブレットが保存されている元のディスクに引き続き書き込まれます。結果としてデータが失われ、クエリが正常に実行できなくなります。 [#5160](https://github.com/StarRocks/starrocks/issues/5160)
- DECIMAL と STRING データ型間で値を変換する際、戻り値が予期しない精度である可能性があります。 [#5608](https://github.com/StarRocks/starrocks/issues/5608)
- DECIMAL 値を BIGINT 値で乗算すると、算術オーバーフローが発生することがあります。このバグを修正するためにいくつかの調整と最適化が行われています。 [#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

リリース日: 2022年4月27日

### バグ修正

以下のバグが修正されました:

- 10進数の乗算がオーバーフローした場合、計算結果が正しくありませんでした。バグが修正された後、10進数の乗算がオーバーフローすると NULL が返されます。
- 実際の統計から大きく乖離している統計がある場合、Collocate Joinの優先度はBroadcast Joinよりも低くなる可能性があります。その結果、クエリプランナーはCollocate Joinをより適切な結合戦略として選択しないことがあります。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 4つ以上のテーブルを結合する際、複雑な式のプランが誤っているとクエリが失敗することがあります。
- シャッフル列が低カーディナリティ列の場合、Shuffle JoinでBEが停止することがあります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- SPLIT関数にNULLパラメータが使用されると、BEが停止することがあります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

リリース日: 2022年4月8日

### 新機能

- `UUID_NUMERIC`関数がサポートされ、LARGEINT値を返します。`UUID`関数と比較して、`UUID_NUMERIC`関数のパフォーマンスは約2桁向上します。

### バグ修正

以下のバグが修正されました：

- 列の削除、新しいパーティションの追加、タブレットのクローン後、古いタブレットと新しいタブレットの列の一意IDが異なる場合があり、システムが共有タブレットスキーマを使用するため、BEが停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks外部テーブルへのデータロード時に、ターゲットStarRocksクラスタの設定されたFEがリーダーでない場合、FEが停止することがあります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- StarRocksのバージョン1.19と2.1で`CAST`関数の結果が異なります。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- Duplicate Keyテーブルがスキーマ変更を行い、同時にマテリアライズドビューを作成すると、クエリ結果が不正確になることがあります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

リリース日: 2022年3月19日

### バグ修正

以下のバグが修正されました：

- BEの障害によるデータ損失の可能性の問題（Batch publish versionを使用して解決）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 不適切な実行プランにより、一部のクエリでメモリ制限を超えるエラーが発生することがあります。
- 異なるコンパクションプロセスでレプリカ間のチェックサムが一致しないことがあります。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- JSONの並べ替えプロジェクションが正しく処理されない場合、クエリが失敗することがあります。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

リリース日: 2022年3月14日

### バグ修正

以下のバグが修正されました：

- バージョン1.19から2.1へのローリングアップグレード中に、2つのバージョン間でチャンクサイズが一致しないため、BEノードが停止することがあります。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- StarRocksがバージョン2.0から2.1にアップデート中に、ローディングタスクが失敗することがあります。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 単一タブレットテーブル結合に適切な実行プランがない場合、クエリが失敗することがあります。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FEノードが低カーディナリティ最適化のためのグローバルディクショナリを構築する情報を収集する際にデッドロックが発生することがあります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- BEノードがデッドロックにより応答しなくなると、クエリが失敗することがあります。
- BIツールがSHOW VARIABLESコマンドの失敗によりStarRocksに接続できないことがあります。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

リリース日: 2022年2月24日

### 新機能

- [プレビュー] StarRocksはIceberg外部テーブルをサポートするようになりました。
- [プレビュー] パイプラインエンジンが利用可能になりました。これはマルチコアスケジューリング用に設計された新しい実行エンジンで、クエリの並列性をparallel_fragment_exec_instance_numパラメータを設定せずに適応的に調整できます。これにより、高並行性シナリオでのパフォーマンスが向上します。
- CTAS（CREATE TABLE AS SELECT）ステートメントがサポートされ、ETLとテーブル作成が容易になります。
- SQLフィンガープリントがサポートされています。SQLフィンガープリントはaudit.logに生成され、遅いクエリの特定を容易にします。
- ANY_VALUE、ARRAY_REMOVE、SHA2関数がサポートされています。

### 改善点

- コンパクションが最適化されました。フラットテーブルは最大10,000列を含むことができます。
- 初回スキャンとページキャッシュのパフォーマンスが最適化されました。ランダムI/Oが削減され、初回スキャンのパフォーマンスが向上します。SATAディスクでの初回スキャン時にこの改善がより顕著です。StarRocksのページキャッシュは元データを保存でき、ビットシャッフルエンコーディングや不要なデコードが不要になります。これによりキャッシュヒット率とクエリ効率が向上します。
- Primary Keyテーブルでのスキーマ変更がサポートされています。`ALTER TABLE`を使用してビットマップインデックスを追加、削除、変更できます。
- [プレビュー] 文字列のサイズは最大1MBまでです。
- JSONロードのパフォーマンスが最適化されました。単一ファイルで100MBを超えるJSONデータをロードできます。
- ビットマップインデックスのパフォーマンスが最適化されました。
- StarRocks Hive外部テーブルのパフォーマンスが最適化されました。CSV形式のデータを読み込むことができます。
- `CREATE TABLE`ステートメントでDEFAULT CURRENT_TIMESTAMPがサポートされています。
- StarRocksは、複数の区切り文字を持つCSVファイルのロードをサポートしています。

### バグ修正

以下のバグが修正されました：

- JSONデータのロードに使用されるコマンドでjsonpathsが指定されている場合、自動的な__opマッピングが有効にならない問題。
- Broker Loadを使用してデータロード中にソースデータが変更されると、BEノードが停止する問題。
- 実体化ビューが作成された後、一部のSQLステートメントがエラーを報告します。
- 引用符で囲まれたjsonpathsが原因でルーチンロードが機能しません。
- クエリする列の数が200を超えると、クエリの同時実行性が急激に低下します。

### 動作変更

コロケーショングループを無効にするAPIが `DELETE /api/colocate/group_stable` から `POST /api/colocate/group_unstable` に変更されました。

### その他

flink-connector-starrocks がFlinkで利用可能になり、StarRocksデータをバッチで読み込むことができるようになりました。これはJDBCコネクタと比較してデータ読み取り効率を向上させます。
