```
---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.1

## 2.1.13

リリース日: 2022年9月6日

### 改善点

- 読み込まれたデータの長さをチェックするためにBE構成項目 `enable_check_string_lengths` を追加しました。このメカニズムにより、VARCHARデータのサイズが範囲外になることによるコンパクションの失敗を防ぐのに役立ちます。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- クエリに1000以上のOR演算子が含まれる場合のクエリパフォーマンスを最適化しました。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### バグ修正

以下のバグが修正されました：

- REPLACE_IF_NOT_NULL関数を使用して計算された配列列をAggregateテーブルを使用するテーブルからクエリする際にエラーが発生し、BEがクラッシュすることがあります。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- クエリに複数のIFNULL()関数がネストされている場合、クエリ結果が正しくありません。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- ダイナミックパーティションが切り捨てられると、動的パーティション設定されたパーティションのタブレット数がデフォルト値に変更されます。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- StarRocksにデータをロードする際にKafkaクラスターが停止すると、デッドロックが発生し、クエリのパフォーマンスに影響します。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- クエリにサブクエリとORDER BY句の両方が含まれると、エラーが発生します。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

リリース日: 2022年8月9日

### 改善点

BDB JEのメタデータクリーンアップを高速化するために、`bdbje_cleaner_threads` と `bdbje_replay_cost_percent` の2つのパラメータを追加しました。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### バグ修正

以下のバグが修正されました：

- 一部のクエリがリーダーFEに転送され、`/api/query_detail`アクションがSHOW FRONTENDSなどのSQLステートメントに関する正しくない実行情報を返す場合があります。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- BEが終了すると、現在のプロセスが完全に終了せず、BEの再起動が失敗することがあります。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 同じHDFSデータファイルを読み込むために複数のBroker Loadジョブが作成された場合、1つのジョブで例外が発生すると、他のジョブがデータを正しく読み取れない場合があり、それにより失敗します。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- 関連する変数がテーブルのスキーマが変更されたときにリセットされず、テーブルをクエリする際にエラー(`no delete vector found tablet`)が発生することがあります。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

リリース日: 2022年7月9日

### バグ修正

以下のバグが修正されました：

- プライマリキーテーブルにデータを頻繁に読み込むエラーが発生すると、プライマリキーテーブルのテーブルにデータを読み込むことが一時停止します。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 低カーディナリティ最適化中に集約式の処理が正しくない順序で実行され、`count distinct`関数が予期しない結果を返すことがあります。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT句の結果が返されない場合があります。これは、句のプルーニングルールを正しく処理できないためです。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 低カーディナリティ最適化のグローバル辞書がクエリの結合条件として定義された列に適用される場合、クエリが予期しない結果を返します。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

リリース日: 2022年6月24日

### バグ修正

以下のバグが修正されました：

- リーダーFEノードを切り替えると、すべてのロードジョブがハングして失敗することがあります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- `DESC` SQLでテーブルスキーマを確認すると、DECIMAL(18,2)タイプのフィールドがDECIMAL64(18,2)と表示されます。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- MemTableのメモリ使用量の推定が4GBを超えると、BEがクラッシュすることがあります。これは、インプットの行が多い場合にデータスキューが発生し、一部のフィールドが大量のメモリリソースを占有するためです。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- コンパクションの際にmax_rows_per_segmentの計算オーバーフローが発生し、大量の小さなセグメントファイルが作成されることがあります。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

リリース日: 2022年6月9日

### 改善点

- スキーマ変更などの内部処理ワークロードに使用される並行制御メカニズムを最適化し、フロントエンド（FE）メタデータの負荷を軽減しました。そのため、これらのロードジョブが同時に実行されて大量のデータをロードする場合、ロードジョブが積み重なって遅くなる可能性が低くなりました。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- StarRocksのデータを高頻度でロードするパフォーマンスを改善しました。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### バグ修正

以下のバグが修正されました：

- ALTER操作ログがLOADステートメントに関するすべての情報を記録しないため、ルーチンロードジョブにALTER操作を実行した後、チェックポイントが作成された後にジョブのメタデータが失われます。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- ルーチンロードジョブを停止するとデッドロックが発生する場合があります。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- バックエンド（BE）は、ロードジョブでDateTime列を使用してロードされたテーブルのタイムスタンプに8時間が追加されるため、サーバーがUTCタイムゾーンを使用している場合、デフォルトでUTC+8タイムゾーンを使用します。[#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING関数は非JSON文字列を処理できません。JSONオブジェクトや配列からJSON値を抽出する場合、関数はNULLを返します。関数は、JSONオブジェクトや配列に等しいJSON形式のSTRING値を返すように最適化されています。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- データ量が大きい場合、スキーマ変更が過剰なメモリ消費のため失敗することがあります。スキーマ変更のすべての段階でメモリ消費制限を指定できるように最適化されました。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- コンパクションを実行中のテーブルの列の重複値数が0x40000000を超えると、コンパクションが一時停止します。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FEが再起動すると、BDB JE v7.3.8のいくつかの問題により高いI/Oと異常なディスク使用量が発生し、通常状態に戻らないことがあります。BDB JE v7.3.7にロールバックすると、FEは通常状態に復元されます。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

リリース日: 2022年5月26日

### 改善点

ROW BETWEEN UNBOUNDED PRECEDING AND CURRENT ROWにフレームが設定されているウィンドウ関数の場合、計算に関与するパーティションが大きい場合、StarRocksは計算を行う前にそのパーティションのすべてのデータをキャッシュします。この状況では大量のメモリリソースが消費されます。StarRocksはこの状況でパーティションのすべてのデータをキャッシュしないように最適化されています。[5829](https://github.com/StarRocks/starrocks/issues/5829)

### バグ修正

以下のバグが修正されました：

- プライマリキーテーブルを使用するテーブルにデータを読み込む際、システム内に格納された各データバージョンの作成時刻が後方に移動するシステム時刻などの理由により、各データバージョンの作成時刻が単調に増加しない場合、データ処理エラーが発生する可能性があります。このようなデータ処理エラーにより、バックエンド（BE）が停止します。[#6046](https://github.com/StarRocks/starrocks/issues/6046)
```
- グラフィカルユーザーインターフェース（GUI）ツールは、set_sql_limit変数を自動的に構成します。その結果、SQLステートメントのORDER BY LIMITが無視され、クエリの返される行数が不正確になります。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- データベースのDROP SCHEMAステートメントが実行されると、データベースが強制的に削除され、復元することができません。[#6201](https://github.com/StarRocks/starrocks/issues/6201)
- JSON形式のデータをロードする際、BEはデータにJSON形式のエラーが含まれると停止します。たとえば、キーと値のペアがカンマ（,）で区切られていない場合です。[#6098](https://github.com/StarRocks/starrocks/issues/6098)
- 大量のデータを高い並列性でロードしている場合、データをディスクに書き込むために実行されるタスクがBEに積み上がることがあります。この状況下で、BEは停止する可能性があります。[#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocksは、テーブルのスキーマ変更を実行する前に必要なメモリ量を見積もります。テーブルに大量のSTRINGフィールドが含まれる場合、メモリの見積もり結果が不正確になる可能性があります。この状況下で、必要とされるメモリ量が単一のスキーマ変更操作に許可されている最大メモリを超える場合、適切に実行されるはずのスキーマ変更操作にエラーが発生することがあります。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- Primary Keyテーブルを使用するテーブルにスキーマ変更を実行した後、そのテーブルにデータをロードする際に「duplicate key xxx」エラーが発生する可能性があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- Shuffle Join操作中に低カーディナリティ最適化が実行されると、パーティションエラーが発生する可能性があります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- コロケーション グループ（CG）に多数のテーブルが含まれ、頻繁にテーブルにデータがロードされる場合、CGは安定状態になることができない可能性があります。この場合、JOINステートメントはコロケーションジョイン操作をサポートしません。StarRocksはデータのロード中に少し長く待機するように最適化されています。これにより、データがロードされるタブレットレプリカの整合性が最大化されます。

## 2.1.6

リリース日: 2022年5月10日

### バグ修正

以下のバグが修正されました:

- 複数のDELETE操作を実行した後にクエリを実行すると、低カーディナリティ列の最適化がクエリに対して実行された場合、正しくないクエリ結果が得られることがあります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 特定のデータ取り込み段階でタブレットが移行される場合、タブレットが格納されている元のディスクにデータの書き込みが継続されます。その結果、データが失われ、クエリが正しく実行されなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- DECIMAL型とSTRING型の値を変換すると、戻り値の精度が予期しないものになることがあります。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- DECIMAL値をBIGINT値で乗算すると、演算オーバーフローが発生する可能性があります。このバグを修正するために、いくつかの調整と最適化が行われました。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

リリース日: 2022年4月27日

### バグ修正

以下のバグが修正されました:

- DECIMAL乗算がオーバーフローした場合に、計算結果が正しくない問題が解決され、DECIMAL乗算がオーバーフローした場合はNULLが返されるようになりました。
- 統計情報が実際の統計情報とかなり異なる場合、Collocate Joinの優先度がBroadcast Joinよりも低くなる可能性があります。その結果、クエリプランナーはColocate Joinをより適切なJoin戦略として選択することができなくなります。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 複数のテーブルを結合する場合、複雑な式の計画が間違っていると、クエリが失敗することがあります。
- シャッフルカラムが低カーディナリティ列である場合、BEが停止することがあります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- SPLIT関数がNULLパラメータを使用すると、BEが停止することがあります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

リリース日: 2022年4月8日

### 新機能

- `UUID_NUMERIC`関数がサポートされ、LARGEINT値が返されます。`UUID`関数と比較して、`UUID_NUMERIC`関数のパフォーマンスがほぼ2桁向上します。

### バグ修正

以下のバグが修正されました:

- カラムを削除し、新しいパーティションを追加し、タブレットをクローンした後、古いタブレットと新しいタブレットのカラムのユニークIDが同じでない可能性があります。これにより、システムが共有タブレットスキーマを使用しているため、BEが停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks外部テーブルにデータをロードする際、対象StarRocksクラスタの構成されたFEがLeaderでない場合、FEが停止することがあります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- `CAST`関数の結果がStarRocks 1.19と2.1で異なることがあります。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- 重複キーのテーブルがスキーマ変更を実行し、同時にマテリアライズドビューを作成すると、クエリ結果が不正確になることがあります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

リリース日: 2022年3月19日

### バグ修正

以下のバグが修正されました:

- BEの障害によるデータの潜在的な損失の問題（バッチパブリッシュバージョンを使用して解決）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 不適切な実行計画により、一部のクエリでメモリ制限オーバーのエラーが発生することがあります。
- 異なる圧縮プロセスでレプリカ間のチェックサムの整合性が一貫していない場合があります。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- JSON再順列投影が正しく処理されない場合、一部の状況でクエリが失敗することがあります。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

リリース日: 2022年3月14日

### バグ修正

以下のバグが修正されました:

- StarRocksバージョン1.19から2.1へのローリングアップグレードでは、2つのバージョン間で一致しないチャンクのサイズにより、BEノードが停止することがあります。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- StarRocksが2.0から2.1にアップデートする際にローディングタスクが失敗することがあります。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 単一テーブレットテーブルの適切な実行計画が存在しない場合、クエリが失敗することがあります。
- 低カーディナリティ最適化のグローバル辞書をビルドするためにFEノードが情報を収集すると、デッドロックの問題が発生する可能性があります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- デッドロックのためにBEノードが停止している場合、クエリが失敗することがあります。
- SHOW VARIABLESコマンドが失敗した場合、BIツールはStarRocksに接続することができません。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

リリース日: 2022年2月24日

### 新機能

- [プレビュー] StarRocksは現在Iceberg外部テーブルをサポートしています。
- [プレビュー] パイプラインエンジンが利用可能です。これは、マルチコアスケジューリング向けに設計された新しい実行エンジンです。クエリの並列性を自動的に調整できるため、parallel_fragment_exec_instance_numパラメータを設定する必要がなくなります。これにより高並列のシナリオでのパフォーマンスが向上します。
- CTAS（CREATE TABLE AS SELECT）ステートメントがサポートされており、ETLとテーブル作成が容易になりました。
- SQLフィンガープリントがサポートされました。SQLフィンガープリントはaudit.logに生成され、遅いクエリの位置を特定するのに役立ちます。
- ANY_VALUE、ARRAY_REMOVE、SHA2関数がサポートされています。

### 改善

- コンパクションが最適化されました。フラットテーブルには最大で1万の列を含めることができます。
- 初回スキャンとページキャッシュのパフォーマンスが最適化されました。ランダムI/Oが削減され、初回スキャンのパフォーマンスが向上します。初回スキャンがSATAディスク上で発生する場合、改善がより顕著です。StarRocksのページキャッシュには元のデータを保存できるため、bitshuffleエンコーディングと不必要なデコーディングが不要になります。これによりキャッシュヒット率とクエリの効率が向上します。
- Primary Keyテーブルでのスキーマ変更がサポートされるようになりました。`Alter table`を使用して、ビットマップインデックスを追加、削除、および変更できます。
- [プレビュー] stringのサイズは最大で1MBになります。
- JSONのロードパフォーマンスが最適化されました。1つのファイルに100MBを超えるJSONデータをロードできます。
- ビットマップインデックスのパフォーマンスが最適化されました。
- StarRocks Hive外部テーブルのパフォーマンスが最適化されました。CSV形式のデータを読み込むことができます。
- `create table`ステートメントでDEFAULT CURRENT_TIMESTAMPがサポートされています。
- StarRocksは複数の区切り記号を持つCSVファイルのロードをサポートしています。

### バグ修正

以下のバグが修正されています:

- JSONデータのロードに使用されるコマンドでjsonpathsが指定されている場合、Auto __opマッピングが効かない問題が修正されました。
- ブローカーロードを使用したデータロード中にソースデータが変更されるため、BEノードが失敗する問題が修正されました。
- マテリアライズドビューが作成された後、一部のSQLステートメントがエラーを報告する問題が修正されました。
- クォートされたjsonpathsによって、ルーチンロードが機能しない問題が修正されました。
- クエリの同時実行が、クエリする列の数が200を超えると急激に減少する問題が修正されました。

### 動作の変更

コロケーショングループを無効にするAPIは、`DELETE /api/colocate/group_stable`から`POST /api/colocate/group_unstable`へと変更されました。

### その他

flink-connector-starrocksは、FlinkがStarRocksデータをバッチで読み取るためのAPIで、JDBCコネクタに比べてデータ読み取り効率を向上させています。