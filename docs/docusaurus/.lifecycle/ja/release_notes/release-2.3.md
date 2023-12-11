---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.3

## 2.3.18

リリース日: 2023年10月11日

### バグ修正

以下の問題が修正されました:

- サードパーティーライブラリ librdkafka のバグにより、定常ロードジョブのロードタスクがデータロード中に停止し、新しく作成されたロードタスクも実行に失敗します。 [#28301](https://github.com/StarRocks/starrocks/pull/28301)
- SparkまたはFlinkコネクタが正確でないメモリ統計のためにデータのエクスポートに失敗することがあります。 [#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- ストリームロードジョブが `if` キーワードを使用すると BE がクラッシュします。 [#31926](https://github.com/StarRocks/starrocks/pull/31926)
- パーティション化された StarRocks 外部テーブルにデータをロードする際に `"get TableMeta failed from TNetworkAddress"` というエラーが報告されます。  [#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

リリース日: 2023年9月4日

### バグ修正

以下の問題が修正されました:

- 定常ロードジョブがデータを消費できない場合があります。 [#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

リリース日: 2023年8月4日

### バグ修正

以下の問題が修正されました:

ブロックされた LabelCleaner スレッドによる FE メモリリークが修正されました。 [#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

リリース日: 2023年7月31日

## 改善点

- 特定の状況下で、タブレットのスケジューリングロジックが最適化され、タブレットが長時間保留されるか、特定の条件下で FE がクラッシュするのを防ぎます。 [#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- TabletChecker のスケジューリングロジックが最適化され、修復されないタブレットの繰り返しスケジューリングを防ぎます。 [#27648](https://github.com/StarRocks/starrocks/pull/27648)
- パーティションメタデータには可視トランザクション ID が記録され、これにより、レプリカの可視バージョンに対応します。レプリカのバージョンが他と一貫性がない場合、このバージョンを作成したトランザクションを追跡しやすくなります。 [#27924](https://github.com/StarRocks/starrocks/pull/27924)

### バグ修正

以下の問題が修正されました:

- FE での正確でないテーブルレベルのスキャン統計は、テーブルクエリやロードに関連するメトリクスが正確でない原因となります。 [#28022](https://github.com/StarRocks/starrocks/pull/28022)
- Join キーが大きな BINARY 列である場合、BE がクラッシュすることがあります。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 特定のシナリオで集約演算子がスレッドセーフティの問題を引き起こし、BE がクラッシュすることがあります。 [#26092](https://github.com/StarRocks/starrocks/pull/26092)
- [RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) を使用してデータを復元した後、テーブルのタブレットのバージョン番号が BE と FE の間で一貫していない場合があります。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) を使用してテーブルを回復した後、パーティションが自動的に作成されない場合があります。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)
- INSERT INTO を使用してロードするデータが品質要件を満たさず、データロードのために厳格モードが有効になっている場合、ロードトランザクションが保留状態になり、DDL ステートメントがハングすることがあります。 [#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 低カーディナリティ最適化が有効になっている場合、一部のINSERTジョブは`[42000][1064] Dict Decode failed, Dict can't take cover all key :0`を返す場合があります。 [#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 特定のケースでは、パイプラインが無効になっている場合に、INSERT INTO SELECT操作がタイムアウトする場合があります。 [#26594](https://github.com/StarRocks/starrocks/pull/26594)
- クエリ条件が `WHERE partition_column < xxx` でクエリが返されない場合、`xxx` の値が時間単位で正確でない（例: `2023-7-21 22`）ときに、クエリがデータを返しません。 [#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

リリース日: 2023年6月28日

### 改善点

- CREATE TABLE がタイムアウトしたときのエラーメッセージが最適化され、パラメータチューニングのヒントが追加されました。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 大量の蓄積されたタブレットバージョンを持つ主キーテーブルのメモリ使用量が最適化されました。 [#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks 外部テーブルメタデータの同期がデータロード時に変更されました。 [#24739](https://github.com/StarRocks/starrocks/pull/24739)
- システムクロックの一貫性がサーバ間で不一致することによって引き起こされた誤った NetworkTime の修正のため、NetworkTime の依存関係がシステムクロックから削除されました。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題が修正されました:

- 小さなテーブルに低カーディナリティ辞書最適化が適用され、頻繁に TRUNCATE 操作を受ける場合、クエリでエラーが発生することがあります。 [#23185](https://github.com/StarRocks/starrocks/pull/23185)
- 最初の子が定数 NULL である UNION を含むビューがクエリされたときに BE がクラッシュすることがあります。 [#13792](https://github.com/StarRocks/starrocks/pull/13792)
- Bitmap Index に基づくクエリがエラーを返すことがあります。 [#23484](https://github.com/StarRocks/starrocks/pull/23484)
- DOUBLEまたはFLOAT値をDECIMAL値に丸める場合、BEでの結果がFEでの結果と一貫していないことがあります。 [#23152](https://github.com/StarRocks/starrocks/pull/23152)
- スキーマ変更が同時にデータロードと発生すると、スキーマ変更がハングすることがあります。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- Parquet ファイルを StarRocks に Broker Load、Sparkコネクター、またはFlinkコネクターを使用してロードすると、BEのOOM問題が発生する可能性があります。 [#25254](https://github.com/StarRocks/starrocks/pull/25254)
- ORDER BY 句で定数が指定され、クエリに LIMIT 句がある場合、不明なエラーが返されます。 [#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

リリース日: 2023年6月1日

### 改善点

- `INSERT INTO ... SELECT` が`thrift_server_max_worker_thread`の値が小さいために期限切れになったときに報告されるエラーメッセージが最適化されました。 [#21964](https://github.com/StarRocks/starrocks/pull/21964)
- `bitmap_contains` 関数を使用する多数のテーブル結合のメモリ消費量が削減され、パフォーマンスが最適化されました。 [#20617](https://github.com/StarRocks/starrocks/pull/20617) [#20653](https://github.com/StarRocks/starrocks/pull/20653)

### バグ修正

以下のバグが修正されました:

- パーティション名が大文字と小文字を区別するため、パーティションの切り捨てが失敗することがあります。 [#21809](https://github.com/StarRocks/starrocks/pull/21809)
- Parquetファイルからのint96タイムスタンプデータのロードがデータオーバーフローを起こすことがあります。 [#22355](https://github.com/StarRocks/starrocks/issues/22355)
- マテリアライズドビューが削除された後に BE の解除が失敗することがあります。 [#22743](https://github.com/StarRocks/starrocks/issues/22743)
- クエリの実行計画にブロードキャストジョインの後にバケットシャッフルジョインが続く場合、左側の表の等結合キーのデータがバケットシャッフルジョインに送信される前に削除されると BE がクラッシュすることがあります。 [#23227](https://github.com/StarRocks/starrocks/pull/23227)
- クエリの実行計画にCross Joinの後にHash Joinが含まれ、かつHash Joinの右テーブルがフラグメントインスタンス内で空の場合、返される結果が正しくない場合があります。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- BEのデコミッショニングが失敗し、マテリアライズドビューの一時パーティションの作成障害が原因です。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- SQL文に複数のエスケープ文字を含むSTRING値が含まれると、SQL文を解析できません。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- パーティション列の最大値でデータをクエリすると失敗します。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocksがv2.4からv2.3にロールバックした後、ロードジョブが失敗します。[#23642](https://github.com/StarRocks/starrocks/pull/23642)
- カラムの整理と再利用に関連する問題。[#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

リリース日: 2023年4月25日

### 改善点

以下の改善が行われました。

- 式の戻り値が有効なブール値に変換できる場合、暗黙の型変換をサポートします。[# 21792](https://github.com/StarRocks/starrocks/pull/21792)

### バグ修正

以下のバグが修正されました。

- ユーザーのLOAD_PRIVがテーブルレベルで許可されている場合、ロードジョブの失敗時にトランザクションロールバックでエラーメッセージ「Access denied; you need (at least one of) the LOAD privilege(s) for this operation」が返されます。[# 21129](https://github.com/StarRocks/starrocks/issues/21129)
- ALTER SYSTEM DROP BACKENDを実行してBEを削除した後、そのBEでレプリケーション数が2に設定されているテーブルのレプリカを修復できなくなります。この状況では、これらのテーブルにデータをロードすることができません。[# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- サポートされていないデータ型がCREATE TABLEで使用されるとNPEが返されます。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- ブロードキャスト結合のショートサーキットロジックが異常であり、クエリ結果が正しくないことがあります。[# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- マテリアライズドビューの使用後、ディスク使用量が大幅に増加する場合があります。[# 20590](https://github.com/StarRocks/starrocks/pull/20590)
- Audit Loaderプラグインを完全にアンインストールできません。[# 20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT`の結果に表示される行数が`SELECT COUNT(*) FROM XXX`の結果と一致しないことがあります。[# 20084](https://github.com/StarRocks/starrocks/issues/20084)
- サブクエリがウィンドウ関数を使用し、その親クエリがGROUP BY句を使用する場合、クエリ結果を集約できません。[# 19725](https://github.com/StarRocks/starrocks/issues/19725)
- BEが起動されると、BEプロセスは存在しますが、すべてのBEのポートが開かれません。[# 19347](https://github.com/StarRocks/starrocks/pull/19347)
- ディスクI/Oが過度に高い場合、主キーテーブルのトランザクションが遅くコミットされ、その結果、これらのテーブルのクエリがエラー「backend not found」を返すことがあります。[# 18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

リリース日: 2023年3月28日

### 改善点

以下の改善が行われました。

- 多くの式を含む複雑なクエリの実行時に通常、大量の`ColumnRefOperators`が生成されます。元々StarRocksは`ColumnRefOperator::id`を保持するために`BitSet`を使用していましたが、これは多くのメモリを消費します。メモリ使用量を減らすために、StarRocksは今、`ColumnRefOperator::id`を保持するために`RoaringBitMap`を使用しています。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 大規模なクエリが小規模なクエリに与える性能影響を軽減するために、新しいI/Oスケジューリング戦略が導入されました。新しいI/Oスケジューリング戦略を有効にするには、BEの静的パラメータ`pipeline_scan_queue_mode=1`を**be.conf**で設定してからBEを再起動します。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### バグ修正

以下のバグが修正されました。

- 期限切れのデータが適切にリサイクルされないテーブルが相対的に大量のディスクスペースを占有することがあります。[#19796](https://github.com/StarRocks/starrocks/pull/19796)
- 次のシナリオで表示されるエラーメッセージが情報を提供していない: ブローカーロードジョブがParquetファイルをStarRocksにロードし、`NULL`値がNOT NULL列にロードされる場合。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 既存のパーティションを置き換えるために大量の一時パーティションを頻繁に作成すると、FEノードでメモリリークとFull GCが発生することがあります。[#19283](https://github.com/StarRocks/starrocks/pull/19283)
- 位置関連テーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`のようなステートメントを使用してレプリカステータスを手動で指定できます。BEの数がレプリカの数以下である場合、破損したレプリカを修復できません。[#19443](https://github.com/StarRocks/starrocks/pull/19443)
- SELECT文がFollower FEに送信されると、`parallel_fragment_exec_instance_num`パラメータが効果を発揮しません。[#18841](https://github.com/StarRocks/starrocks/pull/18841)
- `<=>`演算子を使用して値と`NULL`値を比較すると、比較結果が正しくありません。[#19210](https://github.com/StarRocks/starrocks/pull/19210)
- リソースグループの同時実行制限が継続的に達成されると、クエリの同時実行メトリックが遅く減少します。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 非常に並行したデータロードジョブは、エラー`"get database read lock timeout, database=xxx"`を引き起こす可能性があります。[#16748](https://github.com/StarRocks/starrocks/pull/16748) [#18992](https://github.com/StarRocks/starrocks/pull/18992)

## 2.3.10

リリース日: 2023年3月9日

### 改善点

`storage_medium`の推論を最適化しました。BEがSSDとHDDの両方をストレージデバイスとして使用する場合、`storage_cooldown_time`プロパティが指定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。さもなければ、StarRocksは`storage_medium`を`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)

### バグ修正

以下のバグが修正されました。

- データレイクのParquetファイルからのARRAYデータをクエリすると、クエリが失敗する場合があります。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- プログラムによって開始されたStream Load jobが中断し、FEがプログラムによって送信されたHTTPリクエストを受信しないことがあります。[#18559](https://github.com/StarRocks/starrocks/pull/18559)
- Elasticsearch外部テーブルがクエリされるとエラーが発生する可能性があります。[#13727](https://github.com/StarRocks/starrocks/pull/13727)
- 式が初期化中にエラーに遭遇すると、BEがクラッシュすることがあります。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- SQL文が空の配列リテラル`[]`を使用すると、クエリが失敗することがあります。[#18550](https://github.com/StarRocks/starrocks/pull/18550)
- StarRocksがバージョン2.2およびそれ以降からバージョン2.3.9およびそれ以降にアップグレードされた後、`COLUMN`パラメータで計算式が指定されている場合に、ルーチンロードジョブを作成するとエラー`No match for <expr> with operand types xxx and xxx`が発生することがあります。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BEの再起動後にロードジョブが中断します。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- SELECT文がWHERE句でOR演算子を使用すると、余分なパーティションがスキャンされます。[#18610](https://github.com/StarRocks/starrocks/pull/18610)

## 2.3.9

リリース日: 2023年2月20日

### バグ修正
- スキーマ変更中にタブレットのクローンがトリガーされ、そのタブレットレプリカが配置されているBEノードが変わると、スキーマ変更は失敗します。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- group_concat() 関数によって返される文字列が切り捨てられます。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- Tencent Big Data Suite (TBDS) を介してHDFSからデータをロードするために Broker Load を使用すると、`invalid hadoop.security.authentication.tbds.securekey` エラーが発生し、StarrRocks はTBDSが提供する認証情報を使用してHDFSにアクセスできません。[#14125](https://github.com/StarRocks/starrocks/pull/14125) [#15693](https://github.com/StarRocks/starrocks/pull/15693)
- あるケースでは、CBOが二つのオペレータが等しいかどうかを比較する際に誤ったロジックを使用する可能性があります。[#17227](https://github.com/StarRocks/starrocks/pull/17227) [#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 非Leader FEノードに接続し、SQL 文 `USE <catalog_name>.<database_name>` を送信した場合、非Leader FEノードは SQL 文を先頭の `<catalog_name>` を除いて Leader FE ノードに転送します。その結果、Leader FE ノードは `default_catalog` を使用することを選択し、指定されたデータベースが見つからずに処理が失敗します。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

リリース日: 2023年2月2日

### バグ修正

以下のバグが修正されています：

- 大きなクエリが終了した後、リソースが解放されると、他のクエリが遅くなる可能性が低くなります。特に、リソースグループが有効になっている場合や大きなクエリが予期せず終了した場合にこの問題が発生しやすくなります。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- プライマリキーに対するテーブルで、レプリカのメタデータバージョンが遅れていると、StarRocks は不足しているメタデータを他のレプリカからこのレプリカに増分的にクローンします。このプロセスで膨大な数のメタデータバージョンを取得し、適切な時期にGCが行われないと、過剰なメモリ使用量が発生し、結果的にBEがOOM例外に遭遇する可能性があります。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- FEがBEにまれにハートビートを送信し、ハートビート接続がタイムアウトした場合、FEはBEが利用できないと見なし、BE上でトランザクションの失敗につながります。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- StarRocks 外部テーブルを使用してStarRocksクラスタ間でデータをロードする場合、ソースStarRocksクラスタが以前のバージョンであるとき、およびターゲットStarRocksクラスタが後のバージョンである場合（2.2.8〜2.2.11、2.3.4〜2.3.7、2.4.1または2.4.2）、データの読み込みに失敗します。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- 複数のクエリが同時に実行され、メモリ使用量が比較的高い場合、BEがクラッシュします。[#16047](https://github.com/StarRocks/starrocks/pull/16047)
- テーブルに対して動的パーティショニングが有効になっており、一部のパーティションが動的に削除された場合、TRUNCATE TABLE を実行した場合、エラー `NullPointerException` が返されます。同時に、テーブルにデータをロードすると、FEがクラッシュし、再起動できません。[#16822](https://github.com/StarRocks/starrocks/pull/16822)

## 2.3.7

リリース日: 2022年12月30日

### バグ修正

以下のバグが修正されています：

- StarRocksテーブルでNULLが許可されている列は、そのテーブルから作成されたビューで誤ってNOT NULLに設定されます。[#15749](https://github.com/StarRocks/starrocks/pull/15749)
- StarRocksにデータをロードすると新しいタブレットバージョンが生成されます。しかし、FEはまだ新しいタブレットバージョンを検出せず、引き続きBEに過去のバージョンのタブレットを読み込むよう要求します。ガベージコレクションメカニズムが過去のバージョンを削除すると、クエリは過去のバージョンを見つけることができず、エラー `"Not found: get_applied_rowsets(version xxxx) failed tablet:xxx #version:x [xxxxxxx]"` が返されます。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- データが頻繁にロードされると、FEが過剰なメモリを消費します。[#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 集約クエリや複数テーブルのJOINクエリでは、統計情報が正確に収集されず、実行計画でCROSS JOINが発生し、クエリの遅延が長くなります。[#12067](https://github.com/StarRocks/starrocks/pull/12067)  [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

リリース日: 2022年12月22日

### 改良

- パイプライン実行エンジンが INSERT INTO ステートメントをサポートします。これを有効にするには、FEの設定項目 `enable_pipeline_load_for_insert` を `true` に設定します。  [#14723](https://github.com/StarRocks/starrocks/pull/14723)
- プライマリキーテーブルのコンパクションによって使用されるメモリが削減されました。[#13861](https://github.com/StarRocks/starrocks/pull/13861)  [#13862](https://github.com/StarRocks/starrocks/pull/13862)

### 行動変更

- FEパラメータ `default_storage_medium` を非推奨にしました。テーブルのストレージメディアはシステムによって自動的に推論されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグ修正

以下のバグが修正されています：

- リソースグループ機能が有効になっており、複数のリソースグループが同時にクエリを実行している場合に、BEがハングアップする可能性があります。[#14905](https://github.com/StarRocks/starrocks/pull/14905)
- `CREATE MATERIALIZED VIEW AS SELECT` を使用してマテリアライズドビューを作成する際、SELECT句で集約関数を使用せず、GROUP BYを使用する場合、例えば `CREATE MATERIALIZED VIEW test_view AS SELECT a,b from test group by b,a order by a;` のような場合、BEノードがすべてクラッシュします。[#13743](https://github.com/StarRocks/starrocks/pull/13743)
- 主キーテーブルに頻繁にデータをロードするためにINSERT INTOを使用した後、BEは非常に遅く再起動する場合があります。[#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 環境にJREのみがインストールされ、JDKがインストールされていない場合、FEが再起動した後にクエリが失敗します。バグ修正後、その環境でFEが再起動できず、エラー `JAVA_HOME can not be jre` が返されます。FEを正常に再起動するためには、環境にJDKをインストールする必要があります。[#14332](https://github.com/StarRocks/starrocks/pull/14332)
- クエリがBEをクラッシュさせます。[#14221](https://github.com/StarRocks/starrocks/pull/14221)
- `exec_mem_limit` を式に設定できません。[#13647](https://github.com/StarRocks/starrocks/pull/13647)
- サブクエリの結果に基づいて同期更新されたマテリアライズドビューを作成することはできません。[#13507](https://github.com/StarRocks/starrocks/pull/13507)
- Hive外部テーブルをリフレッシュすると、列のコメントが削除されます。[#13742](https://github.com/StarRocks/starrocks/pull/13742)
- 相関JOIN中に、右のテーブルが左のテーブルよりも前に処理され、右のテーブルが非常に大きい場合、左のテーブルでコンパクションが実行されると、BEノードがクラッシュします。[#14070](https://github.com/StarRocks/starrocks/pull/14070)
- Parquetファイルの列名が大文字と小文字を区別する場合、クエリ条件がParquetファイルからの大文字の列名を使用すると、クエリは結果を返しません。[#13860](https://github.com/StarRocks/starrocks/pull/13860) [#14773](https://github.com/StarRocks/starrocks/pull/14773)
- バルクロード中に、Brokerへの接続数がデフォルトの最大接続数を超えると、Brokerが切断され、ロードジョブは `list path error` エラーメッセージとともに失敗します。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- BEが高負荷の場合、リソースグループのメトリクス `starrocks_be_resource_group_running_queries` が不正確になることがあります。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- クエリステートメントでOUTER JOINを使用すると、BEノードがクラッシュする可能性があります。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- StarRocks 2.4で非同期マテリアライズド・ビューを作成した後、2.3にロールバックすると、FEの起動に失敗する場合があります。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- プライマリーキー・テーブルでdelete_rangeを使用している場合、パフォーマンスが良くないと、RocksDBからのデータ読み取りが遅くなり、CPU使用率が高くなる可能性があります。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.5

リリース日: 2022年11月30日

### 改善

- Colocate JoinがEqui Joinをサポートするようになりました。[#13546](https://github.com/StarRocks/starrocks/pull/13546)
- 頻繁なデータのロード時に、プライマリーキー・インデックスファイルが連続してWALレコードを追加することにより過大になる問題を修正しました。[#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FEはバッチ処理で全てのタブレットをスキャンするようになり、db.readLockを長時間保持することを防ぎます。[#13070](https://github.com/StarRocks/starrocks/pull/13070)

### バグ修正

次のバグが修正されました：

- UNION ALLの結果を直接基にビューを作成し、UNION ALL演算子の入力列にNULLの値が含まれる場合、ビューのスキーマが不正で、列のデータ型がUNION ALLの入力列ではなくNULL_TYPEになっている問題を修正しました。[#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...`や`SELECT * FROM ... LIMIT ...`のクエリ結果が一貫性がない問題を修正しました。[#13585](https://github.com/StarRocks/starrocks/pull/13585)
- FlinkからFEに同期された外部タブレットのメタデータがローカルのタブレットのメタデータを上書きすることで、Flinkからのデータロードが失敗する問題を修正しました。[#12579](https://github.com/StarRocks/starrocks/pull/12579)
- Runtime Filterでのリテラル定数の処理において、NULLフィルタがある場合にBEノードがクラッシュする問題を修正しました。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- CTASを実行した際にエラーが返される問題を修正しました。[#12388](https://github.com/StarRocks/starrocks/pull/12388)
- オーディットログでパイプラインエンジンによって収集された`ScanRows`のメトリクスが誤っている可能性がある問題を修正しました。[#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 圧縮されたHIVEデータをクエリする際にクエリ結果が正しくない問題を修正しました。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- BEノードがクラッシュした後にクエリがタイムアウトし、StarRocksが遅延して応答する問題を修正しました。[#12955](https://github.com/StarRocks/starrocks/pull/12955)
- Broker Loadを使用してデータをロードする際にKerberos認証エラーが発生する問題を修正しました。[#13355](https://github.com/StarRocks/starrocks/pull/13355)
- 多数のOR述語が統計推定に長時間かかる問題を修正しました。[#13086](https://github.com/StarRocks/starrocks/pull/13086)
- BEノードがクラッシュすることがある上に、大文字の列名を含むORCファイル（Snappy圧縮）をBroker Loadで読み込むとBEノードがクラッシュする問題を修正しました。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- プライマリーキー・テーブルのアンロードまたはクエリが30分を超えるとエラーが返される問題を修正しました。[#13403](https://github.com/StarRocks/starrocks/pull/13403)
- ブローカーを使用してHDFSに大容量データをバックアップする際にバックアップタスクが失敗する問題を修正しました。[#12836](https://github.com/StarRocks/starrocks/pull/12836)
- Icebergから読み込んだデータが、`parquet_late_materialization_enable`パラメータにより正しくない可能性がある問題を修正しました。[#13132](https://github.com/StarRocks/starrocks/pull/13132)
- ビューを作成した際にエラー`failed to init view stmt`が返される問題を修正しました。[#13102](https://github.com/StarRocks/starrocks/pull/13102)
- JDBCを使用してStarRockに接続しSQLステートメントを実行する際にエラーが返される問題を修正しました。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- クエリに多くのバケットが関与しタブレットヒントを使用した場合にタイムアウトする問題を修正しました。[#13272](https://github.com/StarRocks/starrocks/pull/13272)
- マテリアライズド・ビューを作成した際に全てのBEノードがクラッシュする問題を修正しました。[#13184](https://github.com/StarRocks/starrocks/pull/13184)
- ALTER ROUTINE LOADを実行して消費済みパーティションのオフセットを更新する際に`The specified partition 1 is not in the consumed partitions`というエラーが返され、フォロワーが最終的にクラッシュする問題を修正しました。[#12227](https://github.com/StarRocks/starrocks/pull/12227)

## 2.3.4

リリース日: 2022年11月10日

### 改善

- ルーチンロードジョブの実行回数が制限を超えて失敗した場合に、StarRocksが解決策を提供するエラーメッセージを表示するようになりました。[#12204](https://github.com/StarRocks/starrocks/pull/12204)
- StarRocksがHiveからデータをクエリしCSVファイルを正しく解析できない際のクエリ失敗問題を修正しました。[#13013](https://github.com/StarRocks/starrocks/pull/13013)

### バグ修正

以下のバグが修正されました：

- HDFSファイルパスに`()`が含まれるとクエリが失敗する可能性がある問題を修正しました。[#12660](https://github.com/StarRocks/starrocks/pull/12660)
- サブクエリにLIMITが含まれる場合、ORDER BY ... LIMIT ... OFFSETの結果が正しくない問題を修正しました。[#9698](https://github.com/StarRocks/starrocks/issues/9698)
- StarRocksがORCファイルをクエリする際に大文字小文字を区別しない問題を修正しました。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- RuntimeFilterがprepareメソッドを呼び出さずにクローズされるとBEがクラッシュする可能性がある問題を修正しました。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- メモリリークのためにBEがクラッシュする可能性がある問題を修正しました。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- 新しい列を追加してすぐにデータを削除すると、クエリ結果が正しくない可能性がある問題を修正しました。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- データをソートすることによりBEがクラッシュする可能性がある問題を修正しました。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- StarRocksとMySQLクライアントが同じLAN上にない場合、INSERT INTO SELECTで作成したロードジョブを一度のKILLで正常に終了できない問題を修正しました。[#11879](https://github.com/StarRocks/starrocks/pull/11897)
- オーディットログでパイプラインエンジンによって収集された`ScanRows`のメトリクスが誤っている可能性がある問題を修正しました。[#12185](https://github.com/StarRocks/starrocks/pull/12185)

## 2.3.3

リリース日: 2022年9月27日

### バグ修正

以下のバグが修正されました：

- Hiveの外部テーブルをテキストファイルとして格納する際にクエリ結果が正確でない可能性がある問題を修正しました。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- Parquetファイルをクエリする際にネストされた配列がサポートされていない問題を修正しました。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- StarRocksおよび外部データソースからデータを読み取る並行クエリが同じリソースグループにルーティングされるとクエリがタイムアウトする可能性がある問題を修正しました。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- パイプライン実行エンジンがデフォルトで有効になっている場合、`parallel_fragment_exec_instance_num`パラメータが1に変更されるため、INSERT INTOを使用したデータローディングが遅くなる問題を修正しました。[#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 式の初期化時に間違いがあるとBEがクラッシュする可能性がある問題を修正しました。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- ORDER BY LIMITを実行すると`heap-buffer-overflow`エラーが発生する可能性がある問題を修正しました。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- リーダーFEを再起動した場合にスキーマ変更が失敗する可能性がある問題を修正しました。[#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

リリース日: 2022年9月7日

### 新機能

- Parquet形式の外部テーブルで範囲フィルタに基づくクエリを高速化するために、レートマテリアル化がサポートされるようになりました。[#9738](https://github.com/StarRocks/starrocks/pull/9738)
- ユーザー認証に関連する情報を表示するための`SHOW AUTHENTICATION`ステートメントが追加されました。[#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改善
- StarRocksは、StarRocksがデータをクエリするバケティングされたHiveテーブルのすべてのデータファイルを再帰的に走査するかどうかを制御する構成項目を提供します。[#10239] (https://github.com/StarRocks/starrocks/pull/10239)
- リソースグループタイプ`realtime`が`short_query`という名前に変更されました。[#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocksは、Hive外部テーブルの大文字と小文字をデフォルトで区別しません。[#10187] (https://github.com/StarRocks/starrocks/pull/10187)

### バグ修正

以下の問題が修正されました。

- Elasticsearch外部テーブルのクエリは、テーブルが複数のシャードに分割されていると予期せずに終了する場合があります。[#10369] (https://github.com/StarRocks/starrocks/pull/10369)
- サブクエリが共通表式（CTE）として書き直されると、StarRocksはエラーをスローします。[#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 大量のデータがロードされると、StarRocksはエラーをスローします。[#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380] (https://github.com/StarRocks/starrocks/issues/10380)
- 同じThriftサービスIPアドレスが複数のカタログに設定されている場合、1つのカタログを削除すると、他のカタログでの増分メタデータの更新が無効になります。[#10511] (https://github.com/StarRocks/starrocks/pull/10511)
- BEのメモリ消費の統計情報が不正確です。[#9837](https://github.com/StarRocks/starrocks/pull/9837)
- Primary KeyテーブルのクエリでStarRocksはエラーをスローします。[#10811](https://github.com/StarRocks/starrocks/pull/10811)
- セレクト権限を持っていても、論理ビューのクエリは許可されません。[#10563](https://github.com/StarRocks/starrocks/pull/10563)
- StarRocksは、論理ビューの命名に制限を課しません。論理ビューは、テーブルと同じ命名規則に従う必要があります。[#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 振る舞いの変更

- BE構成に`bitmap関数`用のデフォルト値 `1000000` と `base64` に基づく`max_length_for_to_base64`を追加し、クラッシュを防ぐためにデフォルト値を ` 200000` に設定しました。[#10851] (https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

リリース日: 2022年8月22日

### 改善

- Broker Loadは、Parquetファイル内の`List`タイプをネストされていない`ARRAY`データ型に変換する機能をサポートします。[#9150](https://github.com/StarRocks/starrocks/pull/9150)
- JSON関連の関数（json_query、get_json_string、およびget_json_int）のパフォーマンスを最適化しました。[#9623](https://github.com/StarRocks/starrocks/pull/9623)
- エラーメッセージを最適化しました。Hive、Iceberg、またはHudiのクエリ中に、列のデータ型がStarRocksでサポートされていない場合、システムはその列に例外をスローします。[#10139](https://github.com/StarRocks/starrocks/pull/10139)
- リソースグループのスケジューリング遅延を削減し、リソースの分離のパフォーマンスを最適化しました。[#10122](https://github.com/StarRocks/starrocks/pull/10122)

### バグ修正

以下の問題が修正されました。

- Elasticsearch外部テーブルのクエリが間違った結果を返す場合があります。これは、`limit`演算子の間違ったプッシュダウンによるものです。[#9952](https://github.com/StarRocks/starrocks/pull/9952)
- Oracle外部テーブルのクエリが`limit`演算子を使用すると失敗する場合があります。[#9542](https://github.com/StarRocks/starrocks/pull/9542)
- すべてのKafka Brokerがルーチンロード中に停止すると、BEがブロックされます。[#9935](https://github.com/StarRocks/starrocks/pull/9935)
- 対応する外部テーブルのデータ型と一致しないParquetファイルのクエリ中に、BEがクラッシュする場合があります。[#10107](https://github.com/StarRocks/starrocks/pull/10107)
- 外部テーブルのスキャン範囲が空のため、クエリがタイムアウトします。[#10091](https://github.com/StarRocks/starrocks/pull/10091)
- サブクエリにORDER BY句が含まれると、システムは例外をスローします。[#10180](https://github.com/StarRocks/starrocks/pull/10180)
- Hive Metastoreが非同期でHiveメタデータが再読み込みされると、ハングします。[#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

リリース日: 2022年7月29日

### 新機能

- Primary Keyテーブルは完全なDELETE WHERE構文をサポートします。詳細については[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)を参照してください。
- Primary Keyテーブルは永続的な主キーインデックスをサポートします。主キーインデックスをディスクに永続化することができ、メモリ使用量を大幅に削減することができます。詳細については[Primary Key table](../table_design/table_types/primary_key_table.md)を参照してください。
- グローバル辞書は、リアルタイムのデータ投入中に更新できます。これにより、クエリのパフォーマンスが最適化され、文字データのクエリパフォーマンスが2倍に向上します。
- CREATE TABLE AS SELECT文を非同期で実行できます。詳細については、[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を参照してください。
- 次のリソースグループ関連の機能をサポートします。
  - リソースグループの監視：監査ログでクエリのリソースグループを表示し、APIを呼び出してリソースグループのメトリックスを取得できます。詳細については[監視およびアラート](../administration/Monitor_and_Alert.md#monitor-and-alerting)を参照してください。
  - CPU、メモリ、およびI/Oリソースの大規模クエリの消費を制限：クエリを特定のリソースグループにルーティングすることができます。クラシファイアに基づいてまたはセッション変数を設定することで、リソースグループにクエリをルーティングできます。詳細については[リソースグループ](../administration/resource_group.md)を参照してください。
- JDBC外部テーブルを使用して、Oracle、PostgreSQL、MySQL、SQLServer、ClickHouseなどのデータベースのデータを簡単にクエリできます。StarRocksは、述語のプッシュダウンもサポートし、クエリのパフォーマンスを向上させます。詳細については[外部テーブル用のJDBC互換データベース](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)を参照してください。
- [プレビュー] 新しいデータソースコネクタフレームワークをリリースし、外部カタログをサポートします。外部テーブルを作成せずにHiveデータに直接アクセスしてクエリできる外部カタログを使用できます。詳細については[内部と外部のデータを管理するためのカタログの使用](../data_source/catalog/query_external_data.md)を参照してください。
- 次の関数を追加しました。
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)、[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)、[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改善

- コンパクションメカニズムは、大量のメタデータをより迅速にマージできるようになりました。これにより、頻繁なデータ更新後のメタデータの圧迫や過剰なディスク使用が防止されます。
- Parquetファイルおよび圧縮ファイルの読み込みのパフォーマンスを最適化しました。
- マテリアライズドビューの作成メカニズムを最適化しました。最適化後、マテリアライズドビューは従来の10倍の速さで作成できます。
- 次の演算子のパフォーマンスを最適化しました。
  - TopNおよびソート演算子
  - 関数を含む等価比較演算子は、これらの演算子がスキャン演算子にプッシュダウンされたときにZone Mapインデックスを使用できるようになりました。
- Apache Hive™外部テーブルのパフォーマンスを最適化しました。
  - Apache Hive™テーブルがParquet、ORC、またはCSV形式で保存されている場合、HiveでのADD COLUMNやREPLACE COLUMNによるスキーマ変更は、対応するHive外部テーブルにREFRESHステートメントを実行するとStarRocksに同期されます。詳細については[Hive外部テーブル](../data_source/External_table.md#hive-external-table)を参照してください。
  - `hive.metastore.uris`はHiveリソースに対して変更できるようになりました。詳細については[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。
- Apache Iceberg外部テーブルのパフォーマンスを最適化しました。カスタムカタログを使用してIcebergリソースを作成できます。詳細については[Apache Iceberg外部テーブル](../data_source/External_table.md#apache-iceberg-external-table)を参照してください。
- Elasticsearch外部テーブルのパフォーマンスを最適化しました。Elasticsearchクラスターのデータノードのアドレスのスニッフィングを無効にすることができます。詳細については[Elasticsearch外部テーブル](../data_source/External_table.md#elasticsearch-external-table)を参照してください。
- sum()関数が数値文字列を受け入れると、数値文字列が暗黙的に変換されるようになりました。
- year()、month()、day()関数がDATEデータ型をサポートするようになりました。

### バグ修正

以下の問題が修正されました。

- 大量のタブレットが原因でCPU使用率が急上昇することがあります。
- "fail to prepare tablet reader"が発生する原因についての問題
- FEsが再起動に失敗する。[#5642](https://github.com/StarRocks/starrocks/issues/5642 )  [#4969](https://github.com/StarRocks/starrocks/issues/4969 )  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- JSON関数を含むステートメントを含む場合、CTASステートメントを正常に実行できません。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### その他

- StarGoは、クラスタ管理ツールであり、複数のクラスタを展開、起動、アップグレード、およびロールバックし、複数のクラスタを管理できます。詳細については、[StarGoを使用したStarRocksのデプロイ](../administration/stargo.md)を参照してください。
- StarRocksをバージョン2.3にアップグレードするかStarRocksを展開する場合、パイプラインエンジンはデフォルトで有効になります。パイプラインエンジンを使用すると、高い同時実行シナリオおよび複雑なクエリのパフォーマンスが向上します。StarRocks 2.3を使用して、大幅なパフォーマンスの低下が検出された場合、`SET GLOBAL`ステートメントを実行して`enable_pipeline_engine`を`false`に設定することで、パイプラインエンジンを無効にできます。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)ステートメントは、MySQLの構文と互換性があり、GRANTステートメントの形式でユーザに割り当てられた権限を表示します。
- memory_limitation_per_thread_for_schema_change（BEの構成項目）は、デフォルト値2 GBを使用することが推奨されており、データボリュームがこの制限を超えるとデータがディスクに書き込まれます。したがって、以前にこのパラメータをより大きな値に設定している場合は、2 GBに設定することをお勧めします。それ以外の場合、スキーマ変更タスクに多量のメモリが必要になる可能性があります。

### アップグレードの注意事項

アップグレード前に使用されていた以前のバージョンにロールバックするには、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを追加し、パラメータを`true`に設定します。このパラメータは、StarRocks v2.2.0で新しい種類のログが追加されたために必要です。パラメータを追加しないと、以前のバージョンにロールバックできません。チェックポイントが作成された後は、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを`false`に設定し、各FEを再起動して、以前の構成に戻します。