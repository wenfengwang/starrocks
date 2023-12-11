---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.1

## 2.1.13

リリース日: 2022年9月6日

### 改善点

- BEの設定項目 `enable_check_string_lengths` を追加し、ロードされたデータの長さを確認するメカニズムを追加しました。このメカニズムにより、VARCHARデータサイズが範囲外になることによるコンパクションの失敗を防ぎます。 [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- クエリに1000個を超えるOR演算子が含まれる場合のクエリパフォーマンスを最適化しました。 [#9332](https://github.com/StarRocks/starrocks/pull/9332)

### バグ修正

以下のバグが修正されました:

- REPLACE_IF_NOT_NULL 関数を使用して計算されたARRAY列を集約テーブルからクエリする際にエラーが発生し、BEがクラッシュする場合があります。 [#10144](https://github.com/StarRocks/starrocks/issues/10144)
- クエリに複数のIFNULL()関数がネストされている場合、クエリ結果が正しくありません。 [#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 動的パーティションが切り詰められた後、パーティション内のタブレット数が動的パーティショニングで構成された値からデフォルト値に変更されます。 [#10435](https://github.com/StarRocks/starrocks/issues/10435)
- StarRocksにデータをロードする際にKafkaクラスタが停止すると、クエリパフォーマンスに影響を及ぼすデッドロックが発生する場合があります。 [#8947](https://github.com/StarRocks/starrocks/issues/8947)
- クエリにサブクエリとORDER BY句の両方が含まれる場合、エラーが発生します。 [#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

リリース日: 2022年8月9日

### 改善点

BDB JEのメタデータのクリーンアップを高速化するために、 `bdbje_cleaner_threads` と `bdbje_replay_cost_percent` の2つのパラメータを追加しました。 [#8371](https://github.com/StarRocks/starrocks/pull/8371)

### バグ修正

以下のバグが修正されました:

- いくつかのクエリがリーダーFEに転送され、 `/api/query_detail` アクションがSHOW FRONTENDSなどのSQLステートメントの実行情報を誤って返すことがあります。 [#9185](https://github.com/StarRocks/starrocks/issues/9185)
- BEが終了すると、現在のプロセスが完全に終了しないため、BEの再起動に失敗します。 [#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 同じHDFSデータファイルを読み込むために複数のBroker Loadジョブが作成された場合、1つのジョブで例外が発生すると、他のジョブがデータを正しく読み取れなくなり、結果的に失敗します。 [#9506](https://github.com/StarRocks/starrocks/issues/9506)
- 関連する変数がリセットされないため、テーブルのスキーマが変更されるとエラーが発生します（`no delete vector found tablet`）。 [#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

リリース日: 2022年7月9日

### バグ修正

以下のバグが修正されました:

- プライマリキーテーブルにデータをロードする際にデータロードが頻繁に発生すると一時停止する場合があります。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 低カーディナリティ最適化中に処理される集約式が不正なシーケンスで処理され、`count distinct` 関数が予期しない結果を返すことがあります。 [#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT句の結果が返されない場合があります。これは、句内のプルーニングルールが正しく処理されないためです。 [#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 低カーディナリティ最適化のグローバル辞書がクエリの結合条件として定義されたカラムに適用される場合、クエリが予期しない結果を返します。 [#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

リリース日: 2022年6月24日

### バグ修正

以下のバグが修正されました:

- Leader FEノードを繰り返し切り替えると、すべてのロードジョブがハングおよび失敗する場合があります。 [#7350](https://github.com/StarRocks/starrocks/issues/7350)
- `DESC` SQLでテーブルスキーマをチェックすると、DECIMAL(18,2)型のフィールドが DECIMAL64(18,2)と表示されます。 [#7309](https://github.com/StarRocks/starrocks/pull/7309)
- データスキュー中にMemTableのメモリ使用量の推定が4GBを超えると、BEがクラッシュします。 [#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 多くの入力行がある場合にコンパクションでmax_rows_per_segmentの計算オーバーフローが発生し、多数の小さなセグメントファイルが作成されます。 [#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

リリース日: 2022年6月9日

### 改善点

- スキーマ変更などの内部処理ワークロードに使用されている並行制御メカニズムを最適化し、フロントエンド（FE）メタデータへの圧力を軽減しました。そのため、これらのロードジョブが同時に実行されて大量のデータをロードする場合に、ロードジョブがたまることなく遅くなることが少なくなります。 [#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- 高頻度でデータをロードする際のStarRocksのパフォーマンスが向上しました。 [#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### バグ修正

以下のバグが修正されました:

- ALTER操作ログがLOADステートメントに関するすべての情報を記録しない場合、定期的なロードジョブにALTER操作を実行した後、チェックポイントが作成された後にジョブのメタデータが失われます。 [#6936](https://github.com/StarRocks/starrocks/issues/6936)
- 定期的なロードジョブを停止するとデッドロックが発生する場合があります。 [#6450](https://github.com/StarRocks/starrocks/issues/6450)
- バックエンド（BE）はデフォルトでUTC+8タイムゾーンを使用してロードジョブに対してデフォルトのUTCタイムゾーンを使用します。サーバーがUTCタイムゾーンを使用している場合、Sparkロードジョブを使用してロードされたテーブルのDateTime列のタイムスタンプに8時間が追加されます。 [#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING 関数は非JSON文字列を処理できません。 JSONオブジェクトや配列からJSON値を抽出する場合、関数はNULLを返します。関数が最適化され、JSON形式の等価のSTRING値を返すようになりました。 [#6426](https://github.com/StarRocks/starrocks/issues/6426)
- データ量が大きい場合、スキーマ変更が過剰なメモリ消費のため失敗することがあります。スキーマ変更のすべての段階でメモリ消費制限を指定できるように最適化されています。 [#6705](https://github.com/StarRocks/starrocks/pull/6705)
- コンパクション中に、コンパクション対象の列の重複値の数が0x40000000を超えると、コンパクションが中断されます。 [#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FEが再起動すると高いI/Oおよび異常に増加するディスク使用量が発生し、BDB JE v7.3.8のいくつかの問題により正常に戻る兆候がなくなります。 FEはBDB JE v7.3.7にロールバックした後、正常な状態に復元されます。 [#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

リリース日: 2022年5月26日

### 改善点

フレームが `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` に設定されたウィンドウ関数において、計算に関与するパーティションが大きい場合、StarRocksは計算を行う前にパーティションのすべてのデータをキャッシュします。 この状況では大量のメモリリソースが消費されます。 StarRocksはこのような状況でのパーティションのすべてのデータをキャッシュしないように最適化されました。 [5829](https://github.com/StarRocks/starrocks/issues/5829)

### バグ修正

以下のバグが修正されました:

- プライマリキーテーブルを使用するテーブルにデータがロードされると、各データバージョンの作成時刻が逆戻りするシステム時刻の移動や関連する未知のバグなどの理由でモノトニックに増加しない場合、データ処理エラーが発生する可能性があります。そのようなデータ処理エラーがBEを停止させます。 [#6046](https://github.com/StarRocks/starrocks/issues/6046)
- グラフィカルユーザーインターフェイス（GUI）ツールは、set_sql_limit変数を自動で構成することがあります。その結果、SQLステートメントのORDER BY LIMITが無視され、クエリの戻り値が正しくない場合があります。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- データベースでDROP SCHEMAステートメントを実行すると、データベースは強制的に削除され、復元できなくなります。[#6201](https://github.com/StarRocks/starrocks/issues/6201)
- JSON形式のデータをロードする際、BEがJSONフォーマットエラーを含むデータを受け取ると、停止することがあります。たとえば、キーと値のペアがカンマ（,）で区切られていない場合があります。[#6098](https://github.com/StarRocks/starrocks/issues/6098)
- 非常に並列的な方法で大量のデータをロードすると、データのディスクへの書き込みタスクがBEに積み重なることがあります。この状況では、BEは停止する可能性があります。[#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocksは、テーブルのスキーマ変更前に必要なメモリ量を推定します。テーブルに多数のSTRINGフィールドが含まれている場合、メモリの推定結果が正確でない可能性があります。この場合、必要なメモリ量が単一のスキーマ変更操作に許可されている最大メモリ量を超える場合、正しく実行されるはずのスキーマ変更操作でエラーが発生する可能性があります。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- プライマリキーテーブルを使用するテーブルにスキーマ変更が行われた後、そのテーブルにデータをロードする際に「重複キーxxx」エラーが発生する可能性があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- シャッフル結合操作中に低カーディナリティ最適化が実行されると、パーティションエラーが発生する可能性があります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- コロケーション・グループ（CG）に多数のテーブルが含まれ、テーブルに頻繁にデータがロードされると、CGは安定状態に留まれない可能性があります。この場合、JOINステートメントはコロケート結合操作をサポートしません。StarRocksはデータのロード中に少し長く待機することで、データがロードされるタブレットレプリカの整合性を最大限に確保できるように最適化されています。

## 2.1.6

リリース日:2022年5月10日

### バグ修正

以下のバグが修正されました。
- 複数のDELETE操作を実行した後にクエリを実行すると、クエリの最適化により低カーディナリティ列のクエリが実行された場合、正しくないクエリ結果が得られることがあります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 特定のデータ取り込みフェーズでタブレットが移行されると、元のディスクにデータが書き込まれ続けます。その結果、データが失われ、クエリが正しく実行できなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- DECIMALとSTRINGデータ型間で値を変換すると、予期しない精度で戻り値が得られる場合があります。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- DECIMAL値をBIGINT値で乗算すると、算術オーバーフローが発生する可能性があります。このバグを修正するために、一部の調整と最適化が行われました。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

リリース日:2022年4月27日

### バグ修正

以下のバグが修正されました。

- DECIMALの乗算でオーバーフローが発生する場合、計算結果が正しくありません。バグの修正後、DECIMALの乗算でオーバーフローが発生した場合、NULLが返されます。
- 統計情報が実際の統計情報からかなりの逸脱がある場合、Collocate Joinの優先度がBroadcast Joinよりも低くなることがあります。その結果、クエリプランナーは適切なジョイン戦略としてColocate Joinを選択しないことがあります。 [#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 複数のテーブルを結合する場合に、複雑な式のための計画が誤っているためにクエリが失敗することがあります。
- シャッフル列が低カーディナリティ列である場合、Shuffle Joinの下でBEが停止することがあります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- SPLIT関数がNULLパラメータを使用した場合、BEが停止することがあります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

リリース日:2022年4月8日

### 新機能

- `UUID_NUMERIC`関数がサポートされました。これにより、`UUID`関数よりも2桁のパフォーマンスが向上します。

### バグ修正

以下のバグが修正されました。
- 列を削除した後、新しいパーティションを追加し、タブレットをクローンすると、古いタブレットと新しいタブレットの列のユニークなIDが同じでない可能性があります。これにより、BEが共有タブレットスキーマを使用するため、BEが停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks外部テーブルにデータをロードする際、対象のStarRocksクラスターの構成FEがLeaderでない場合、FEが停止する可能性があります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- `CAST`関数の結果が、StarRocksバージョン1.19と2.1で異なる場合があります。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- 重複キーテーブルがスキーマ変更を実行し、同時にマテリアライズド・ビューを作成する場合、クエリ結果が正しくないことがあります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

リリース日:2022年3月19日

### バグ修正

以下のバグが修正されました。
- BEの障害によるデータ損失の可能性の問題（バッチ発行バージョンを使用することで解決）[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 不適切な実行計画により、一部のクエリがメモリ制限を超える可能性があります。
- 異なるコンパクションプロセスでレプリカ間のチェックサムが一貫していない場合があります。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- JSONの再順序プロジェクションが正しく処理されない場合、クエリが一部の状況で失敗する可能性があります。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

リリース日:2022年3月14日

### バグ修正

以下のバグが修正されました。
- StarRocksバージョン1.19から2.1へのローリングアップグレードでは、2つのバージョン間で一致しないチャンクサイズのためにBEノードが停止することがあります。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- StarRocksがバージョン2.0から2.1にアップグレード中にローディングタスクが失敗することがあります。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- シングルテーブレットテーブルの適切な実行計画がない場合、クエリが失敗することがあります。
- 低カーディナリティ最適化のために、FEノードがグローバル辞書を構築するための情報収集中にデッドロックの問題が発生する可能性があります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- BEノードがデッドロックに陥ることで停止している場合、クエリが失敗することがあります。
- SHOW VARIABLESコマンドの失敗により、BIツールがStarRocksに接続できなくなる場合があります。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

リリース日:2022年2月24日

### 新機能

- [プレビュー]StarRocksは現在Iceberg外部テーブルをサポートしています。
- [プレビュー]パイプラインエンジンが利用可能です。これはマルチコアスケジューリング向けの新しい実行エンジンで、クエリの並列性を適応的に調整できます。このため、parallel_fragment_exec_instance_numパラメータを設定する必要はありません。これにより、高並行性シナリオでのパフォーマンスが向上します。
- `CREATE TABLE AS SELECT`（CTAS）ステートメントがサポートされ、ETLとテーブルの作成が容易になります。
- SQLフィンガープリントがサポートされています。SQLフィンガープリントはaudit.logで生成され、遅いクエリの場所を特定しやすくします。
- `ANY_VALUE`、`ARRAY_REMOVE`、`SHA2`関数がサポートされています。

### 改善点

- コンパクションが最適化されました。フラットテーブルには最大で1万の列を含めることができます。
- 最初のスキャンとページキャッシュのパフォーマンスが最適化されました。ランダムI/Oが減少し、最初のスキャンのパフォーマンスが向上しました。SATAディスクで最初のスキャンが発生した場合、改善がより目立ちます。StarRocksのページキャッシュは元のデータを格納できるため、bitshuffleエンコーディングや不必要なデコードが不要になります。これにより、キャッシュヒット率とクエリ効率が向上します。
- プライマリキーテーブルでスキーマ変更がサポートされています。`Alter table`を使用して、ビットマップインデックスを追加、削除、修正できます。
- [プレビュー]文字列のサイズが最大で1 MBになります。
- JSONのロードパフォーマンスが最適化されました。1つのファイルに100 MBを超えるJSONデータをロードできます。
- ビットマップインデックスのパフォーマンスが最適化されました。
- StarRocks Hive外部テーブルのパフォーマンスが最適化されました。CSV形式のデータを読み込むことができます。
- `CREATE TABLE`ステートメントで`DEFAULT CURRENT_TIMESTAMP`がサポートされています。
- StarRocksは複数の区切り記号を持つCSVファイルの読み込みをサポートしています。

### バグ修正

以下のバグが修正されています。

- JSONデータを読み込むために使用されるコマンドにjsonpathsが指定されている場合、自動__opマッピングが効果を発揮しません。
- データの読み込み中にソースデータが変更されるため、BEノードが失敗します。
- マテリアライズドビューが作成された後、一部のSQLステートメントでエラーが発生します。
- 引用符付きjsonpathsのため、ルーチン読み込みが機能しません。
- クエリの列数が200を超えると、クエリの同時実行が急激に減少します。

### 動作の変更

Colocation Groupを無効にするAPIは、`DELETE /api/colocate/group_stable`から `POST /api/colocate/group_unstable`に変更されました。

### その他

flink-connector-starrocksにより、FlinkがStarRocksデータをバッチで読み取ることができるようになりました。これによりJDBCコネクタと比べてデータの読み取り効率が向上します。