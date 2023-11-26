---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.1

## 2.1.13

リリース日: 2022年9月6日

### 改善点

- BE 設定項目 `enable_check_string_lengths` を追加し、ロードされたデータの長さをチェックするようにしました。このメカニズムにより、VARCHAR データのサイズが範囲外になることによるコンパクションの失敗を防ぐことができます。[#10380](https://github.com/StarRocks/starrocks/issues/10380)
- クエリに1000以上の OR 演算子が含まれる場合のクエリパフォーマンスを最適化しました。[#9332](https://github.com/StarRocks/starrocks/pull/9332)

### バグ修正

以下のバグが修正されました:

- 集計テーブルを使用してテーブルから ARRAY 列（REPLACE_IF_NOT_NULL 関数を使用して計算されたもの）をクエリするとエラーが発生し、BE がクラッシュする場合があります。[#10144](https://github.com/StarRocks/starrocks/issues/10144)
- クエリに複数の IFNULL() 関数がネストされている場合、クエリ結果が正しくありません。[#5028](https://github.com/StarRocks/starrocks/issues/5028) [#10486](https://github.com/StarRocks/starrocks/pull/10486)
- 動的パーティションが切り捨てられると、パーティション内のタブレット数が動的パーティショニングで設定された値からデフォルト値に変更されます。[#10435](https://github.com/StarRocks/starrocks/issues/10435)
- StarRocks にデータをロードするために Routine Load を使用する際に Kafka クラスタが停止すると、デッドロックが発生し、クエリのパフォーマンスに影響を与える場合があります。[#8947](https://github.com/StarRocks/starrocks/issues/8947)
- クエリにサブクエリと ORDER BY 句の両方が含まれる場合、エラーが発生します。[#10066](https://github.com/StarRocks/starrocks/pull/10066)

## 2.1.12

リリース日: 2022年8月9日

### 改善点

BDB JE のメタデータクリーンアップを高速化するために、`bdbje_cleaner_threads` と `bdbje_replay_cost_percent` の2つのパラメータを追加しました。[#8371](https://github.com/StarRocks/starrocks/pull/8371)

### バグ修正

以下のバグが修正されました:

- 特定のクエリが Leader FE に転送され、`/api/query_detail` アクションが SHOW FRONTENDS などの SQL ステートメントの実行情報を正しく返さない場合があります。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- BE が終了した後、現在のプロセスが完全に終了しないため、BE の再起動に失敗する場合があります。[#9175](https://github.com/StarRocks/starrocks/pull/9267)
- 同じ HDFS データファイルをロードするために複数の Broker Load ジョブが作成された場合、1つのジョブで例外が発生すると、他のジョブはデータを正しく読み取ることができず、結果的に失敗する場合があります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)
- テーブルのスキーマが変更された場合、関連する変数がリセットされず、テーブルをクエリする際にエラー (`no delete vector found tablet`) が発生する場合があります。[#9192](https://github.com/StarRocks/starrocks/issues/9192)

## 2.1.11

リリース日: 2022年7月9日

### バグ修正

以下のバグが修正されました:

- プライマリキーテーブルにデータが頻繁にロードされると、データのロードが中断される場合があります。[#7763](https://github.com/StarRocks/starrocks/issues/7763)
- 低基数最適化中に集計式の処理順序が正しくないため、`count distinct` 関数が予期しない結果を返す場合があります。[#7659](https://github.com/StarRocks/starrocks/issues/7659)
- LIMIT 句に対して結果が返されない場合があります。これは、クエリのプルーニングルールが正しく処理されないためです。[#7894](https://github.com/StarRocks/starrocks/pull/7894)
- 低基数最適化のグローバル辞書がクエリの結合条件として定義された列に適用される場合、クエリは予期しない結果を返します。[#8302](https://github.com/StarRocks/starrocks/issues/8302)

## 2.1.10

リリース日: 2022年6月24日

### バグ修正

以下のバグが修正されました:

- リーダー FE ノードの切り替えを繰り返すと、すべてのロードジョブが停止し、失敗する場合があります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- DECIMAL(18,2) 型のフィールドが `DESC` SQL でテーブルスキーマをチェックすると DECIMAL64(18,2) と表示されます。[#7309](https://github.com/StarRocks/starrocks/pull/7309)
- MemTable のメモリ使用量の推定が4GBを超えると BE がクラッシュする場合があります。これは、ロード中のデータのスキューにより、一部のフィールドが大量のメモリリソースを占有する可能性があるためです。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- コンパクション時に入力行が多い場合、max_rows_per_segment の計算でオーバーフローが発生し、大量の小さなセグメントファイルが作成されます。[#5610](https://github.com/StarRocks/starrocks/issues/5610)

## 2.1.8

リリース日: 2022年6月9日

### 改善点

- スキーマ変更などの内部処理ワークロードに使用される並行制御メカニズムが最適化され、フロントエンド（FE）メタデータへの負荷が軽減されました。そのため、これらのロードジョブが同時に実行されて大量のデータをロードする場合でも、ロードジョブが積み重なって遅くなることは少なくなります。[#6560](https://github.com/StarRocks/starrocks/pull/6560) [#6804](https://github.com/StarRocks/starrocks/pull/6804)
- StarRocks の高頻度データロードのパフォーマンスが向上しました。[#6532](https://github.com/StarRocks/starrocks/pull/6532) [#6533](https://github.com/StarRocks/starrocks/pull/6533)

### バグ修正

以下のバグが修正されました:

- ALTER 操作のログが LOAD ステートメントのすべての情報を記録しないため、ルーチンロードジョブに ALTER 操作を実行した後、チェックポイントが作成された後にジョブのメタデータが失われる場合があります。[#6936](https://github.com/StarRocks/starrocks/issues/6936)
- ルーチンロードジョブを停止するとデッドロックが発生する場合があります。[#6450](https://github.com/StarRocks/starrocks/issues/6450)
- バックエンド（BE）は、デフォルトの UTC+8 タイムゾーンを使用してロードジョブを実行します。サーバーが UTC タイムゾーンを使用している場合、Spark ロードジョブでロードされるテーブルの DateTime 列のタイムスタンプに8時間が追加されます。[#6592](https://github.com/StarRocks/starrocks/issues/6592)
- GET_JSON_STRING 関数は、非 JSON 文字列を処理できません。JSON オブジェクトや配列から JSON 値を抽出する場合、関数は NULL を返します。関数は、JSON オブジェクトや配列に対して等価な JSON 形式の STRING 値を返すように最適化されました。[#6426](https://github.com/StarRocks/starrocks/issues/6426)
- データ量が大きい場合、スキーマ変更がメモリの消費量が過剰になるために失敗する場合があります。スキーマ変更の各段階でメモリ消費量の制限を指定できるように最適化が行われました。[#6705](https://github.com/StarRocks/starrocks/pull/6705)
- コンパクション中に列の重複値の数が0x40000000を超える場合、コンパクションが中断されます。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE を再起動すると、BDB JE v7.3.8 のいくつかの問題により、I/O が高くなり、ディスク使用量が異常に増加し、正常に復元する兆候がありません。FE は BDB JE v7.3.7 にロールバックすることで正常に復元されます。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.1.7

リリース日: 2022年5月26日

### 改善点

ROW BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW のフレームが設定されたウィンドウ関数において、計算に関与するパーティションが大きい場合、StarRocks は計算を実行する前にパーティションのすべてのデータをキャッシュします。この状況では、大量のメモリリソースが消費されます。StarRocks はこの状況でパーティションのすべてのデータをキャッシュしないように最適化されました。[5829](https://github.com/StarRocks/starrocks/issues/5829)

### バグ修正

以下のバグが修正されました:

- プライマリキーテーブルを使用するテーブルにデータがロードされると、システムに格納されている各データバージョンの作成時刻が単調に増加しない場合、データ処理エラーが発生する可能性があります。このようなデータ処理エラーにより、バックエンド（BE）が停止します。[#6046](https://github.com/StarRocks/starrocks/issues/6046)
- 一部のグラフィカルユーザーインターフェース（GUI）ツールは、set_sql_limit 変数を自動的に設定します。そのため、SQL ステートメント ORDER BY LIMIT が無視され、クエリの結果として正しくない行数が返されます。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- データベースに DROP SCHEMA ステートメントを実行すると、データベースが強制的に削除され、復元できなくなります。[#6201](https://github.com/StarRocks/starrocks/issues/6201)
- JSON 形式のデータがロードされると、データに JSON 形式のエラーが含まれる場合、BE が停止します。たとえば、キーと値がカンマ（,）で区切られていない場合などです。[#6098](https://github.com/StarRocks/starrocks/issues/6098)
- 大量のデータが高い並行性でロードされている場合、データをディスクに書き込むために実行されるタスクが BE 上で積み重なる場合があります。この状況では、BE が停止する可能性があります。[#3877](https://github.com/StarRocks/starrocks/issues/3877)
- StarRocks はテーブルのスキーマ変更を実行する前に必要なメモリ量を推定します。テーブルに大量の STRING フィールドが含まれる場合、メモリの推定結果が不正確になる場合があります。この状況では、必要なメモリ量の推定値が単一のスキーマ変更操作に許可される最大メモリを超える場合、正しく実行されるはずのスキーマ変更操作がエラーになります。[#6322](https://github.com/StarRocks/starrocks/issues/6322)
- プライマリキーテーブルを使用するテーブルにスキーマ変更が行われた後、そのテーブルにデータをロードする際に "duplicate key xxx" エラーが発生する場合があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- シャッフル結合時に低基数最適化が実行されると、パーティションエラーが発生する場合があります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- コロケーショングループ（CG）に多数のテーブルが含まれ、テーブルに頻繁にデータがロードされる場合、CG が安定状態になることができない場合があります。この場合、JOIN ステートメントは Colocate Join 操作をサポートしません。StarRocks はデータロード時に少し長く待機するように最適化されました。これにより、データがロードされるタブレットレプリカの整合性を最大化することができます。

## 2.1.6

リリース日: 2022年5月10日

### バグ修正

以下のバグが修正されました:

- 複数の DELETE 操作を実行した後にクエリを実行すると、低基数列の最適化がクエリに対して実行された場合、正しいクエリ結果が得られない場合があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 特定のデータ取り込みフェーズでタブレットが移行されると、タブレットが格納されている元のディスクにデータが引き続き書き込まれます。その結果、データが失われ、クエリが正常に実行できなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- DECIMAL 値と STRING 値の間で値を変換すると、返される値の精度が予期しないものになる場合があります。[#5608](https://github.com/StarRocks/starrocks/issues/5608)
- DECIMAL 値に BIGINT 値を乗算すると、算術オーバーフローが発生する場合があります。このバグを修正するために、いくつかの調整と最適化が行われました。[#4211](https://github.com/StarRocks/starrocks/pull/4211)

## 2.1.5

リリース日: 2022年4月27日

### バグ修正

以下のバグが修正されました:

- DECIMAL の乗算がオーバーフローすると計算結果が正しくありません。バグ修正後は、DECIMAL の乗算がオーバーフローすると NULL が返されます。
- 統計情報が実際の統計情報から大きく逸脱している場合、Collocate Join の優先度が Broadcast Join よりも低くなる場合があります。その結果、クエリプランナーはより適切な Join 戦略として Colocate Join を選択しない場合があります。[#4817](https://github.com/StarRocks/starrocks/pull/4817)
- 4つ以上のテーブルを結合する場合、複雑な式のための計画が間違っているため、クエリが失敗する場合があります。
- シャッフル結合時にシャッフル列が低基数列の場合、BE が停止する場合があります。[#4890](https://github.com/StarRocks/starrocks/issues/4890)
- SPLIT 関数が NULL パラメータを使用する場合、BE が停止する場合があります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

## 2.1.4

リリース日: 2022年4月8日

### 新機能

- `UUID_NUMERIC` 関数がサポートされ、LARGEINT 値を返すようになりました。`UUID` 関数と比較して、`UUID_NUMERIC` 関数のパフォーマンスは2桁近く向上します。

### バグ修正

以下のバグが修正されました:

- 列の削除、新しいパーティションの追加、タブレットのクローンを行った後、古いタブレットと新しいタブレットの列の一意の ID が一致しない場合があります。これは、システムが共有タブレットスキーマを使用しているため、BE が停止する原因となります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks 外部テーブルにデータをロードする際、対象の StarRocks クラスタの設定された FE がリーダーでない場合、FE が停止します。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- `CAST` 関数の結果が StarRocks バージョン 1.19 と 2.1 で異なる場合があります。[#4701](https://github.com/StarRocks/starrocks/pull/4701)
- ダブルキー テーブルでスキーマ変更が実行され、同時にマテリアライズドビューが作成された場合、クエリが "duplicate key xxx" エラーを返す場合があります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)

## 2.1.3

リリース日: 2022年3月19日

### バグ修正

以下のバグが修正されました:

- BE の障害によるデータ損失の可能性の問題（バッチパブリッシュバージョンを使用して解決）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 不適切な実行計画により、一部のクエリがメモリ制限超過エラーを報告する場合があります。
- 異なるコンパクションプロセスでレプリカ間のチェックサムが一貫性がない場合があります。[#3438](https://github.com/StarRocks/starrocks/issues/3438)
- JSON の再順序化プロジェクションが正しく処理されない場合、一部のクエリが失敗する場合があります。[#4056](https://github.com/StarRocks/starrocks/pull/4056)

## 2.1.2

リリース日: 2022年3月14日

### バグ修正

以下のバグが修正されました:

- バージョン 1.19 から 2.1 へのローリングアップグレードでは、BE ノードがチャンクサイズの不一致により停止します。[#3834](https://github.com/StarRocks/starrocks/issues/3834)
- StarRocks のアップデート中にロードタスクが失敗する場合があります。[#3828](https://github.com/StarRocks/starrocks/issues/3828)
- 単一タブレットテーブル結合のための適切な実行計画が存在しない場合、クエリが失敗します。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FE ノードがグローバル辞書をビルドするための情報を収集する際にデッドロックの問題が発生する場合があります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)
- BE ノードがデッドロックにより停止する場合があります。
- SHOW VARIABLES コマンドが失敗すると BI ツールが StarRocks に接続できなくなります。[#3708](https://github.com/StarRocks/starrocks/issues/3708)

## 2.1.0

リリース日: 2022年2月24日

### 新機能

- [プレビュー] StarRocks は Iceberg 外部テーブルをサポートします。
- [プレビュー] パイプラインエンジンが利用可能になりました。これは、マルチコアスケジューリングに対応した新しい実行エンジンであり、パラレルフラグメント実行インスタンス数を設定する必要がなくなります。これにより、高並行性のシナリオでのパフォーマンスが向上します。
- CTAS（CREATE TABLE AS SELECT）ステートメントがサポートされ、ETL とテーブル作成が容易になりました。
- SQL フィンガープリントがサポートされました。SQL フィンガープリントは audit.log に生成され、遅いクエリの特定を容易にします。
- ANY_VALUE、ARRAY_REMOVE、SHA2 関数がサポートされました。

### 改善点

- コンパクションが最適化されました。フラットテーブルには最大で10,000列を含めることができます。
- 初回スキャンとページキャッシュのパフォーマンスが最適化されました。初回スキャンのパフォーマンスを向上させるためにランダム I/O が削減されました。SarRocks のページキャッシュは元のデータを格納することができるため、ビットシャッフルエンコーディングや不要なデコードが不要になります。これにより、キャッシュヒット率とクエリの効率が向上します。
- プライマリキーテーブルでスキーマ変更がサポートされました。`Alter table` を使用してビットマップインデックスの追加、削除、変更を行うことができます。
- [プレビュー] 文字列のサイズは最大で1MBまで指定できます。
- JSON ロードのパフォーマンスが最適化されました。1つのファイルに100MB以上のJSONデータをロードすることができます。
- ビットマップインデックスのパフォーマンスが最適化されました。
- StarRocks Hive 外部テーブルのパフォーマンスが最適化されました。CSV 形式のデータを読み取ることができます。
- `create table` ステートメントで DEFAULT CURRENT_TIMESTAMP がサポートされました。
- StarRocks は複数の区切り文字を持つ CSV ファイルのロードをサポートします。

### バグ修正

以下のバグが修正されました:

- jsonpaths が JSON データのロードコマンドで指定されている場合、自動 __op マッピングが効果を発揮しません。
- データロード中にソースデータが変更されると、Broker Load を使用してデータをロードする際に BE ノードが失敗します。
- マテリアライズドビューが作成された後、一部の SQL ステートメントがエラーを報告します。
- ルーチンロードがクォートされた jsonpaths により機能しない場合があります。
- クエリの並列度が200を超える場合、クエリの並列度が急激に低下します。

### 動作の変更

コロケーショングループを無効にするための API が `DELETE /api/colocate/group_stable` から `POST /api/colocate/group_unstable` に変更されました。

### その他

flink-connector-starrocks が利用可能になり、Flink で StarRocks データをバッチで読み取ることができます。これにより、JDBC コネクタと比較してデータの読み取り効率が向上します。
