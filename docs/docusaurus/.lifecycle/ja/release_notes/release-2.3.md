```yaml
---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン2.3

## 2.3.18

リリース日: 2023年10月11日

### バグ修正

以下の問題を修正しました:

- サードパーティ製ライブラリlibrdkafkaのバグにより、Routine Loadジョブのロードタスクがデータロード中にスタックし、新規に作成されたロードタスクも実行に失敗する場合があります。 [#28301](https://github.com/StarRocks/starrocks/pull/28301)
- SparkまたはFlinkコネクタがメモリ統計の不正確さによりデータのエクスポートに失敗することがあります。 [#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- Stream Loadジョブが`if`というキーワードを使用するとBEがクラッシュすることがあります。 [#31926](https://github.com/StarRocks/starrocks/pull/31926)
- パーティション化されたStarRocks外部テーブルにデータをロードする際に`"get TableMeta failed from TNetworkAddress"`というエラーが報告されることがあります。 [#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

リリース日: 2023年9月4日

### バグ修正

以下の問題を修正しました:
- Routine Loadジョブがデータを消費できない問題を修正しました。 [#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

リリース日: 2023年8月4日

### バグ修正

以下の問題を修正しました:
ブロックされたLabelCleanerスレッドによるFEメモリリークを修正しました。 [#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

リリース日: 2023年7月31日

## 改善

- タブレットスケジューリングロジックを最適化し、特定の状況下でタブレットが長時間ペンディングのままになるか、FEがクラッシュするのを防ぎました。 [#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- TabletCheckerのスケジューリングロジックを最適化し、修復されていないタブレットを繰り返しスケジューリングすることを防ぎました。 [#27648](https://github.com/StarRocks/starrocks/pull/27648)
- パーティションメタデータにはvisibleTxnIdが記録され、それはタブレット複製の表示バージョンに対応します。複製のバージョンが他と一貫していない場合、このバージョンを作成したトランザクションを追跡しやすくなります。 [#27924](https://github.com/StarRocks/starrocks/pull/27924)

### バグ修正

以下の問題を修正しました:

- FEsでの不正確なテーブルレベルのスキャン統計が、テーブルのクエリおよびロードに関連するメトリクスに不正確さをもたらすことがあります。 [#28022](https://github.com/StarRocks/starrocks/pull/28022)
- Joinキーが大きなBINARY列である場合、BEがクラッシュする可能性があります。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 特定のシナリオで、集約演算子がスレッド安全性の問題を引き起こし、BEがクラッシュする可能性があります。 [#26092](https://github.com/StarRocks/starrocks/pull/26092)
- データが[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)を使用して復元された後、タブレットのバージョン番号がBEとFEの間で一貫していない場合があります。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md)を使用してテーブルが復旧された後、パーティションが自動的に作成されない場合があります。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)
- INSERT INTOを使用してデータをロードしようとする際に、データが厳格な要件を満たさず、データロードに対して厳格モードが有効になっている場合、ローディングトランザクションがペンディング状態になり、DDLステートメントがハングします。 [#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 低カーディナリティ最適化が有効になっている場合、一部のINSERTジョブが`[42000][1064] Dict Decode failed, Dict can't take cover all key :0`を返すことがあります。 [#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 特定のケースでは、Pipelineが有効になっていない場合に、INSERT INTO SELECT操作がタイムアウトする場合があります。 [#26594](https://github.com/StarRocks/starrocks/pull/26594)
- クエリの条件が`WHERE partition_column < xxx`で、`xxx`の値が時刻に正確ですが分や秒には正確でない場合（例: `2023-7-21 22`）、クエリはデータを返さないことがあります。 [#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

リリース日: 2023年6月28日

### 改善

- CREATE TABLEのタイムアウト時に返されるエラーメッセージを最適化し、パラメータの調整のヒントを追加しました。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 大量の蓄積されたタブレットバージョンを持つプライマリキー表のメモリ使用量を最適化しました。 [#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks外部テーブルメタデータの同期をデータのロード時に発生するように変更しました。 [#24739](https://github.com/StarRocks/starrocks/pull/24739)
- システムクロックに依存するNetworkTimeの依存関係を削除し、サーバ間で一貫していないシステムクロックによる不正確なNetworkTimeを修正しました。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題を修正しました:

- 頻繁にTRUNCATE操作を受ける小さなテーブルに低カーディナリティ辞書最適化が適用される場合、クエリでエラーが発生することがあります。 [#23185](https://github.com/StarRocks/starrocks/pull/23185)
- NULL定数を含むUNIONを持つビューがクエリされるとBEがクラッシュすることがあります。 [#13792](https://github.com/StarRocks/starrocks/pull/13792)
- Bitmap Indexに基づくクエリがエラーを返すことがあります。 [#23484](https://github.com/StarRocks/starrocks/pull/23484)
- DOUBLEまたはFLOAT値をDECIMAL値に丸める結果がBEとFEで一貫していない場合があります。 [#23152](https://github.com/StarRocks/starrocks/pull/23152)
- データロードとスキーマ変更が同時に発生した場合、スキーマ変更が一時停止することがあります。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- Broker Load、Sparkコネクタ、またはFlinkコネクタを使用してParquetファイルをStarRocksにロードすると、BEがOOMの問題が発生することがあります。 [#25254](https://github.com/StarRocks/starrocks/pull/25254)
- クエリにORDER BY句で定数が指定され、クエリにLIMIT句がある場合、`unknown error`というエラーメッセージが返されることがあります。 [#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

リリース日: 2023年6月1日

### 改善

- INSERT INTO ... SELECTのタイムアウト時に報告されるエラーメッセージを最適化し、マルチテーブルのJOINに`bitmap_contains`関数を使用する際のメモリ消費を削減し、パフォーマンスを最適化しました。 [#21964](https://github.com/StarRocks/starrocks/pull/21964) [#20617](https://github.com/StarRocks/starrocks/pull/20617) [#20653](https://github.com/StarRocks/starrocks/pull/20653)

### バグ修正

以下のバグが修正されました:

- パーティション名に大文字と小文字が区別されるため、パーティションの切り捨てに失敗することがあります。 [#21809](https://github.com/StarRocks/starrocks/pull/21809)
- Parquetファイルからのint96タイムスタンプデータのロードがデータのオーバーフローを引き起こすことがあります。 [#22355](https://github.com/StarRocks/starrocks/issues/22355)
- マテリアライズドビューが削除された後、BEの運用を停止することがあります。 [#22743](https://github.com/StarRocks/starrocks/issues/22743)
- クエリの実行計画にブロードキャスト結合の後にバケットシャッフル結合が続く場合に、左側のテーブルの等しい結合キーのデータがバケットシャッフル結合に送信される前に削除されるとBEがクラッシュすることがあります。 [#23227](https://github.com/StarRocks/starrocks/pull/23227)
```
- クエリの実行プランにクロス結合の後にハッシュ結合が含まれ、かつフラグメントインスタンス内のハッシュ結合の右側テーブルが空の場合、返される結果が正しくない場合があります。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- マテリアライズドビューの一時的なパーティションの作成に失敗することでBEの運用停止に失敗します。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- SQL文に複数のエスケープ文字を含むSTRING値が含まれる場合、SQL文を解析できません。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- パーティション列の最大値のデータをクエリする際に失敗します。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocksがv2.4からv2.3にロールバックした後、ロードジョブが失敗します。[#23642](https://github.com/StarRocks/starrocks/pull/23642)
- 列の削減と再利用に関連する問題。[#16624](https://github.com/StarRocks/starrocks/issues/16624)

## 2.3.12

リリース日: 2023年4月25日

### 改善

以下をサポートします:
- フラグメントが有効な真偽値に変換できる場合、暗黙の変換をサポートします。[# 21792](https://github.com/StarRocks/starrocks/pull/21792)

### バグ修正

以下のバグが修正されました:

- ユーザーのLOAD_PRIVがテーブルレベルで付与されている場合、ロードジョブの失敗時にトランザクションがロールバックされ、エラーメッセージ `Access denied; you need (at least one of) the LOAD privilege(s) for this operation` が返されます。[# 21129](https://github.com/StarRocks/starrocks/issues/21129)
- ALTER SYSTEM DROP BACKENDを実行してBEを削除した後、そのBEでレプリケーション数が2に設定されたテーブルのレプリカを修復できません。この状況では、これらのテーブルへのデータロードが失敗します。[# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- CREATE TABLEでサポートされていないデータ型が使用されている場合、NPEが返されます。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- ブロードキャスト結合のショートサーキットロジックが異常であり、クエリ結果が正しくありません。[# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- マテリアライズドビューを使用するとディスク使用量が大幅に増加する場合があります。[# 20590](https://github.com/StarRocks/starrocks/pull/20590)
- オーディットローダープラグインを完全にアンインストールできません。[# 20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT`の結果に表示される行数が、`SELECT COUNT(*) FROM XXX`の結果と一致しない場合があります。[# 20084](https://github.com/StarRocks/starrocks/issues/20084)
- サブクエリがウィンドウ関数を使用し、その親クエリがGROUP BY句を使用する場合、クエリ結果を集計できません。[# 19725](https://github.com/StarRocks/starrocks/issues/19725)
- BEが起動されると、BEプロセスは存在しますが、すべてのBEポートが開かれません。[# 19347](https://github.com/StarRocks/starrocks/pull/19347)
- ディスクI/Oが過度に高い場合、プライマリキーのテーブルでトランザクションが遅くコミットされ、結果としてこれらのテーブルのクエリでエラー "backend not found" が返される場合があります。[# 18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

リリース日: 2023年3月28日

### 改善

- 多くの式を含む複雑なクエリを実行すると、通常は多くの `ColumnRefOperators` が生成されます。元々、StarRocksは `ColumnRefOperator::id` を保存するために `BitSet` を使用していましたが、これには多くのメモリが消費されます。メモリ使用量を減らすために、StarRocksは今、`ColumnRefOperator::id` を保存するために `RoaringBitMap` を使用します。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 大規模なクエリが小規模なクエリに与える性能への影響を軽減するために、新しいI/Oスケジューリング戦略が導入されました。新しいI/Oスケジューリング戦略を有効にするには、BEの静的パラメータ `pipeline_scan_queue_mode=1` を **be.conf** に設定し、その後BEを再起動してください。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### バグ修正

以下のバグが修正されました:

- 期限切れのデータが適切にリサイクルされていないテーブルは、比較的大きなディスク領域を占有します。[#19796](https://github.com/StarRocks/starrocks/pull/19796)
- 次のシナリオで表示されるエラーメッセージは情報がありません: ブローカーロードジョブがParquetファイルをStarRocksにロードし、`NULL`値がNOT NULL列にロードされます。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- 既存のパーティションを置き換えるために大量の一時的なパーティションを頻繁に作成すると、FEノードでメモリリークとFull GCが発生します。[#19283](https://github.com/StarRocks/starrocks/pull/19283)
- 置き換えることができないレプリカを `ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");` のようなステートメントを使用して手動で指定できます、BEの数がレプリカの数以下の場合、破損したレプリカを修復できません。[#19443](https://github.com/StarRocks/starrocks/pull/19443)
- Follower FEに `INSERT INTO SELECT` リクエストが送信された場合、パラメータ `parallel_fragment_exec_instance_num` が効果を持ちません。[#18841](https://github.com/StarRocks/starrocks/pull/18841)
- `<=>` 演算子を使用して値を `NULL` 値と比較した場合、比較結果が正しくありません。[#19210](https://github.com/StarRocks/starrocks/pull/19210)
- リソースグループの同時実行制限が継続的に達成された場合、クエリ並行性メトリックが徐々に減少します。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- 高い並行データロードジョブはエラー `"get database read lock timeout, database=xxx"` を発生させる場合があります。[#16748](https://github.com/StarRocks/starrocks/pull/16748) [#18992](https://github.com/StarRocks/starrocks/pull/18992)

## 2.3.10

リリース日: 2023年3月9日

### 改善

- `storage_medium` の推論を最適化しました。BEがストレージデバイスとしてSSDとHDDの両方を使用する場合、`storage_cooldown_time` プロパティが指定されている場合、StarRocksは `storage_medium` を `SSD` に設定します。それ以外の場合、StarRocksは `storage_medium` を `HDD` に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)

### バグ修正

以下のバグが修正されました:

- データレイクのParquetファイルからARRAYデータをクエリすると、クエリが失敗する場合があります。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- プログラムによって開始されたStream Loadジョブがハングし、FEがプログラムによって送信されたHTTPリクエストを受信しません。[#18559](https://github.com/StarRocks/starrocks/pull/18559)
- Elasticsearch外部テーブルがクエリされた場合、エラーが発生する場合があります。[#13727](https://github.com/StarRocks/starrocks/pull/13727)
- 式が初期化中にエラーに遭遇した場合、BEがクラッシュする可能性があります。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- SQL文が空の配列リテラル `[]` を使用する場合、クエリが失敗する場合があります。[#18550](https://github.com/StarRocks/starrocks/pull/18550)
- StarRocksがバージョン2.2から2.3.9およびそれ以降にアップグレードされた後、`Rou tine Load` ジョブが `COLUMN` パラメータで指定された計算式で作成されると、エラー `No match for <expr> with operand types xxx and xxx` が発生します。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BE再起動後、ロードジョブがハングします。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- SELECT文がWHERE句でOR演算子を使用する場合、余分なパーティションがスキャンされます。[#18610](https://github.com/StarRocks/starrocks/pull/18610)

## 2.3.9

リリース日: 2023年2月20日

### バグ修正
- スキーマの変更中に、タブレットのクローンがトリガーされ、そのタブレットレプリカが存在するBEノードが変更されると、スキーマの変更に失敗します。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- group_concat() 関数によって返される文字列が切り捨てられます。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- Tencent Big Data Suite（TBDS）を介してHDFSからデータをロードするために Broker Load を使用する場合、エラー `invalid hadoop.security.authentication.tbds.securekey` が発生し、StarRocks が TBDS が提供した認証情報を使用して HDFS へアクセスできないことを示します。 [#14125](https://github.com/StarRocks/starrocks/pull/14125) [#15693](https://github.com/StarRocks/starrocks/pull/15693)
- 一部のケースでは、CBO が誤ったロジックを使用して、2つのオペレータが等価かどうかを比較する可能性があります。[#17227](https://github.com/StarRocks/starrocks/pull/17227) [#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 非リーダーのFEノードに接続し、SQL 文 `USE <catalog_name>.<database_name>` を送信した場合、非リーダーの FE ノードはSQL 文を `<catalog_name>` を除外してリーダーの FE ノードに転送します。結果として、リーダーの FE ノードは `default_catalog` を使用しようとし、指定されたデータベースを見つけられず、最終的に失敗します。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

リリース日: 2023年2月2日

### バグ修正

次のバグが修正されています:

- 大規模なクエリの完了後にリソースが解放されると、他のクエリが低速化する可能性が低いです。この問題は、リソースグループが有効になっている場合や大規模なクエリが予期せず終了した場合により頻繁に発生する可能性があります。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- プライマリキーのテーブルにおいて、レプリカのメタデータバージョンが遅れると、StarRocks はこのレプリカに他のレプリカから不足しているメタデータを増分クローンします。このプロセスで、StarRocks は大量のメタデータのバージョンを取得し、タイムリーなGCなしで過剰なメモリを消費し、その結果、BEがOOM例外に遭遇する可能性があります。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- FEがBEに時々ハートビートを送信し、ハートビート接続がタイムアウトすると、FEはBEを利用できないと見なし、BE上でのトランザクションの失敗につながります。[# 16386](https://github.com/StarRocks/starrocks/pull/16386)
- StarRocks 外部テーブルを使用して、StarRocks クラスタ間でデータをロードする場合、ソースの StarRocks クラスタがより古いバージョンであり、ターゲットの StarRocks クラスタがより新しいバージョンである場合（2.2.8 〜 2.2.11、2.3.4 〜 2.3.7、2.4.1 または 2.4.2）、データのロードに失敗します。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- 複数のクエリが同時に実行され、メモリ使用量が比較的高い場合に BEがクラッシュすることがあります。[#16047](https://github.com/StarRocks/starrocks/pull/16047)
- テーブルの動的パーティショニングが有効になっており、一部のパーティションが動的に削除された場合、TRUNCATE TABLE を実行すると、`NullPointerException` エラーが返されます。また、テーブルにデータをロードした場合、FEがクラッシュして再起動できなくなります。[#16822](https://github.com/StarRocks/starrocks/pull/16822)

## 2.3.7

リリース日: 2022年12月30日

### バグ修正

次のバグが修正されています:

- StarRocks テーブルで NULL を許可する列が、そのテーブルから作成されたビューで誤って NOT NULL に設定されています。[#15749](https://github.com/StarRocks/starrocks/pull/15749)
- StarRocks にデータがロードされると新しいタブレットバージョンが生成されます。ただし、FEがまだ新しいタブレットバージョンを検出しておらず、引き続きBEに対してタブレットの過去のバージョンを読み取る必要があります。ガベージコレクションメカニズムが過去のバージョンを削除すると、クエリは過去のバージョンを見つけられずにエラーが返されます。"Not found: get_applied_rowsets(version xxxx) failed tablet:xxx #version:x [xxxxxxx]"。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- データを頻繁にロードすると、FEがあまりにも多くのメモリを使用することがあります。[#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 集計クエリおよびマルチテーブル JOIN クエリに対して、統計情報が正確に収集されず、実行計画で CROSS JOIN が発生し、クエリの待ち時間が長くなることがあります。[#12067](https://github.com/StarRocks/starrocks/pull/12067)  [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

リリース日: 2022年12月22日

### 改善点

- パイプライン実行エンジンは、INSERT INTO 文をサポートします。有効にするには、FE構成項目 `enable_pipeline_load_for_insert` を `true` に設定します。[#14723](https://github.com/StarRocks/starrocks/pull/14723)
- プライマリキーのテーブルにおけるコンパクションの使用メモリが削減されました。[#13861](https://github.com/StarRocks/starrocks/pull/13861)  [#13862](https://github.com/StarRocks/starrocks/pull/13862)

### 動作の変更

- FEパラメータ `default_storage_medium` を非推奨としました。テーブルの保存媒体は自動的にシステムによって推論されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグ修正

次のバグが修正されています:

- リソースグループ機能が有効になっていて複数のリソースグループで同時にクエリが実行されている場合に BEがハングアップすることがあります。[#14905](https://github.com/StarRocks/starrocks/pull/14905)
- `CREATE MATERIALIZED VIEW AS SELECT` を使用してマテリアライズドビューを作成する場合、SELECT句が集約関数を使用せず、GROUP BY を使用する場合、たとえば`CREATE MATERIALIZED VIEW test_view AS SELECT a,b from test group by b,a order by a;`、BEノードがすべてクラッシュします。[#13743](https://github.com/StarRocks/starrocks/pull/13743)
- プライマリキーのテーブルに頻繁にデータをロードすると、BEの再起動が非常に遅くなることがあります。[#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 環境にJREのみがインストールされており、JDKがインストールされていない場合、FEの再起動後にクエリが失敗します。バグが修正された後、その環境でFEは再起動できず、`JAVA_HOME can not be jre` エラーが返されます。FEを正常に再起動するには、その環境にJDKをインストールする必要があります。[#14332](https://github.com/StarRocks/starrocks/pull/14332)
- クエリによってBEがクラッシュします。[#14221](https://github.com/StarRocks/starrocks/pull/14221)
- `exec_mem_limit` を式に設定できません。[#13647](https://github.com/StarRocks/starrocks/pull/13647)
- サブクエリの結果に基づいて同期更新されたマテリアライズドビューを作成できません。[#13507](https://github.com/StarRocks/starrocks/pull/13507)
- Hive外部テーブルをリフレッシュすると、列のコメントが削除されます。[#13742](https://github.com/StarRocks/starrocks/pull/13742)
- 相関JOINの際、右側のテーブルが左側のテーブルよりも先に処理され、右側のテーブルが非常に大きい場合、左側のテーブルにコンパクションが実行されると、BEノードがクラッシュします。[#14070](https://github.com/StarRocks/starrocks/pull/14070)
- Parquetファイルの列名が大文字と小文字を区別する場合、クエリ条件でParquetファイルから大文字の列名を使用すると、クエリは結果を返しません。[#13860](https://github.com/StarRocks/starrocks/pull/13860) [#14773](https://github.com/StarRocks/starrocks/pull/14773)
- バルクロード中に、Brokerへの接続数がデフォルトの最大接続数を超えると、Brokerが切断され、ロードジョブが `list path error` エラーメッセージとともに失敗します。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- BEが高負荷の場合、リソースグループのメトリック `starrocks_be_resource_group_running_queries` が不正確になることがあります。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- クエリ文でOUTER JOINを使用すると、BEノードがクラッシュする可能性があります。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- StarRocks 2.4で非同期マテリアライズドビューを作成した後、バージョン2.3にロールバックすると、FEが起動しないことがあります。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- プライマリキーテーブルがdelete_rangeを使用し、パフォーマンスが良くない場合、RocksDBからのデータ読み取りが遅くなり、高いCPU使用率を引き起こすことがあります。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.5

リリース日: 2022年11月30日

### 改良

- Colocate JoinがEqui Joinをサポートするようになりました。[#13546](https://github.com/StarRocks/starrocks/pull/13546)
- データが頻繁にロードされる場合に、連続してWALレコードが追加されることでプライマリキーインデックスファイルが大きすぎる問題を修正しました。[#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FEはバッチですべてのタブレットをスキャンするため、db.readLockを長時間保持することを防ぐためにスキャン間隔でdb.readLockを解放します。[#13070](https://github.com/StarRocks/starrocks/pull/13070)

### バグ修正

以下のバグが修正されました:

- UNION ALLの結果を直接もとにビューを作成し、UNION ALL演算子の入力列にNULL値が含まれる場合、ビューのスキーマが不正です。NULL_TYPEのデータ型ではなく、UNION ALLの入力列です。[#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...`および`SELECT * FROM ... LIMIT ...`のクエリ結果が一貫していません。[#13585](https://github.com/StarRocks/starrocks/pull/13585)
- FEに同期された外部タブレットメタデータがローカルタブレットメタデータを上書きし、Flinkからのデータ読み込みが失敗する原因となる問題を修正しました。[#12579](https://github.com/StarRocks/starrocks/pull/12579)
- Runtime FilterでNULLフィルタがリテラル定数を処理するとBEノードがクラッシュします。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- CTASを実行するとエラーが返されます。[#12388](https://github.com/StarRocks/starrocks/pull/12388)
- オーディットログでパイプラインエンジンによって収集された`ScanRows`メトリクスが間違っている可能性があります。[#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 圧縮されたHIVEデータをクエリすると、クエリ結果が正しくありません。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- BEノードがクラッシュし、その後StarRocksが遅く応答するため、クエリがタイムアウトします。[#12955](https://github.com/StarRocks/starrocks/pull/12955)
- Broker Loadを使用してデータをロードする際にKerberos認証エラーが発生します。[#13355](https://github.com/StarRocks/starrocks/pull/13355)
- OR述語が多すぎると、統計推定に時間がかかります。[#13086](https://github.com/StarRocks/starrocks/pull/13086)
- Broker Loadが大文字の列名を含むORCファイル（Snappy圧縮）をロードするとBEノードがクラッシュします。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- プライマリキーテーブルのアンロードまたはクエリで30分以上かかるとエラーが返されます。[#13403](https://github.com/StarRocks/starrocks/pull/13403)
- ブローカーを使用して大量のデータをHDFSにバックアップすると、バックアップタスクが失敗します。[#12836](https://github.com/StarRocks/starrocks/pull/12836)
- Icebergから読み取ったデータが正しくないことがあり、`parquet_late_materialization_enable`パラメータに起因するものです。[#13132](https://github.com/StarRocks/starrocks/pull/13132)
- ビューを作成するとエラー`failed to init view stmt`が返されます。[#13102](https://github.com/StarRocks/starrocks/pull/13102)
- JDBCを使用してStarRockに接続しSQLステートメントを実行するとエラーが返されます。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- クエリに多数のバケットやタブレットヒントが含まれると、クエリがタイムアウトする可能性があります。[#13272](https://github.com/StarRocks/starrocks/pull/13272)
- BEノードがクラッシュして再起動できず、その間に新しく構築したテーブルへのデータのロードジョブがエラーを報告します。[#13701](https://github.com/StarRocks/starrocks/pull/13701)
- マテリアライズドビューが作成されるとすべてのBEノードがクラッシュします。[#13184](https://github.com/StarRocks/starrocks/pull/13184)
- ALTER ROUTINE LOADを実行して消費されたパーティションのオフセットを更新すると、指定されたパーティション1が消費されたパーティションに含まれていないというエラーが返され、フォロワーが最終的にクラッシュします。[#12227](https://github.com/StarRocks/starrocks/pull/12227)

## 2.3.4

リリース日: 2022年11月10日

### 改良

- StarRocksがCSVファイルを解析できない場合に、ルーチンロードジョブの作成に失敗したときにエラーメッセージが解決策を提供します。[#12204]( https://github.com/StarRocks/starrocks/pull/12204)
- StarRocksがHiveからデータをクエリし、CSVファイルを解析できない場合にクエリが失敗します。[#13013](https://github.com/StarRocks/starrocks/pull/13013)

### バグ修正

以下のバグが修正されました:

- HDFSファイルパスに`()`が含まれる場合、クエリが失敗することがあります。[#12660](https://github.com/StarRocks/starrocks/pull/12660)
- サブクエリにLIMITが含まれる場合、ORDER BY ... LIMIT ... OFFSETの結果が正しくありません。[#9698](https://github.com/StarRocks/starrocks/issues/9698)
- StarRocksはORCファイルをクエリする際に大文字小文字を区別しません。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- RuntimeFilterがprepareメソッドを呼び出さずに閉じられるとBEがクラッシュする可能性があります。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- メモリリークによりBEがクラッシュすることがあります。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- 新しい列を追加してすぐにデータを削除すると、クエリ結果が正しくない場合があります。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- データのソートによりBEがクラッシュすることがあります。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- StarRocksとMySQLクライアントが同じLAN上にない場合、INSERT INTO SELECTを使用して作成されたロードジョブをKILLを1回実行するだけでは正常に終了させることができません。[#11879](https://github.com/StarRocks/starrocks/pull/11897)
- パイプラインエンジンによって収集された`ScanRows`メトリクスが間違っている可能性があります。[#12185](https://github.com/StarRocks/starrocks/pull/12185)

## 2.3.3

リリース日: 2022年9月27日

### バグ修正

以下のバグが修正されました:

- Hiveの外部テーブルがテキストファイルとして格納されている場合にクエリ結果が正確でない場合があります。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- Parquetファイルをクエリする際にネストされた配列がサポートされていません。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- StarRocksおよび外部データソースからデータを読み取る並列クエリが同じリソースグループにルーティングされると、クエリがタイムアウトすることがあります。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- パイプライン実行エンジンがデフォルトで有効になっている場合、パラレルフラグメントの実行インスタンス数のパラメータが1に変更されます。これにより、INSERT INTOを使用したデータのロードが遅くなる可能性があります。[#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 式の初期化に誤りがある場合、BEがクラッシュする可能性があります。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- ORDER BY LIMITを実行すると`heap-buffer-overflow`エラーが発生する可能性があります。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- Leader FEを再起動するとスキーマ変更が失敗することがあります。[#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

リリース日: 2022年9月7日

### 新機能

- パーケット形式の外部テーブルでの範囲フィルタベースのクエリを高速化するためにLate materializationがサポートされました。[#9738](https://github.com/StarRocks/starrocks/pull/9738)
- SHOW AUTHENTICATIONステートメントが追加され、ユーザー認証関連情報を表示します。[#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改良
- 構成アイテムは提供されるが、StarRocksがデータをクエリする場合に、バケット分割されたHiveテーブルのすべてのデータファイルを再帰的に走査するかどうかを制御するために使用されています。[#10239](https://github.com/StarRocks/starrocks/pull/10239)
- `realtime`リソースグループのタイプは`short_query`に名前が変更されています。[#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocksは、Hive外部テーブルで大文字と小文字をデフォルトで区別しなくなりました。[#10187](https://github.com/StarRocks/starrocks/pull/10187)

### バグ修正

以下のバグが修正されました：

- Elasticsearch外部テーブルのクエリは、テーブルが複数のシャードに分割されると予期しない終了する場合があります。[#10369](https://github.com/StarRocks/starrocks/pull/10369)
- サブクエリが共通テーブル式（CTE）として書き直されると、StarRocksがエラーをスローします。[#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 大量のデータがロードされると、StarRocksがエラーをスローします。[#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 同じThriftサービスIPアドレスが複数のカタログに構成されている場合、1つのカタログを削除すると、他のカタログの増分メタデータの更新が無効になります。[#10511](https://github.com/StarRocks/starrocks/pull/10511)
- BEからのメモリ消費の統計情報が正確ではありません。[#9837](https://github.com/StarRocks/starrocks/pull/9837)
- 主キー付きテーブルのクエリでStarRocksがエラーをスローします。[#10811](https://github.com/StarRocks/starrocks/pull/10811)
- 論理ビューのクエリが許可されず、これらのビューに対するSELECT権限がある場合でもエラーが発生します。[#10563](https://github.com/StarRocks/starrocks/pull/10563)
- StarRocksは、論理ビューの命名に制限を課しません。現在、論理ビューはテーブルと同じ命名規則に従う必要があります。[#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 動作変更

- ビットマップ機能のデフォルト値1000000に対するBE構成`max_length_for_bitmap_function`を追加し、クラッシュを防ぐためにbase64向けのデフォルト値200000に対する`max_length_for_to_base64`を追加しました。[#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

リリース日：2022年8月22日

### 改善

- ブローカーロードは、Parquetファイルのリスト型をネストされていないARRAYデータ型に変換することをサポートします。[#9150](https://github.com/StarRocks/starrocks/pull/9150)
- JSON関連の機能（json_query、get_json_string、get_json_int）のパフォーマンスが最適化されました。[#9623](https://github.com/StarRocks/starrocks/pull/9623)
- エラーメッセージが最適化されました：Hive、Iceberg、またはHudiのクエリ中に、StarRocksがサポートしていない列のデータ型をクエリしようとすると、システムがその列で例外をスローします。[#10139](https://github.com/StarRocks/starrocks/pull/10139)
- リソースグループのスケジューリングレイテンシーが低減され、リソースの分離パフォーマンスが最適化されました。[#10122](https://github.com/StarRocks/starrocks/pull/10122)

### バグ修正

以下のバグが修正されました：

- Elasticsearch外部テーブルのクエリで誤った結果が返されることがあります。これは、`limit`演算子の不正確なプッシュダウンによるものです。[#9952](https://github.com/StarRocks/starrocks/pull/9952)
- Oracle外部テーブルのクエリで`limit`演算子が使用されると失敗することがあります。[#9542](https://github.com/StarRocks/starrocks/pull/9542)
- ルーチンロード中にKafka Brokersがすべて停止すると、BEがブロックされます。[#9935](https://github.com/StarRocks/starrocks/pull/9935)
- 対応する外部テーブルのParquetファイルのデータ型が一致しない場合、クエリ中にBEがクラッシュします。[#10107](https://github.com/StarRocks/starrocks/pull/10107)
- エクスターナルテーブルのスキャン範囲が空のため、クエリでタイムアウトが発生します。[#10091](https://github.com/StarRocks/starrocks/pull/10091)
- サブクエリにORDER BY句が含まれている場合、システムが例外をスローします。[#10180](https://github.com/StarRocks/starrocks/pull/10180)
- Hiveメタストアが非同期で再読み込みされる場合、ハングが発生します。[#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

リリース日：2022年7月29日

### 新機能

- Primary Keyテーブルは完全なDELETE WHERE構文をサポートしています。詳細については、[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)を参照してください。
- Primary Keyテーブルは永続的なプライマリキーインデックスをサポートしています。メモリではなくディスクにプライマリキーインデックスを永続化できます。これによりメモリ使用量を大幅に削減できます。詳細については、[Primary Keyテーブル](../table_design/table_types/primary_key_table.md)を参照してください。
- グローバル辞書はリアルタイムデータ取り込み中に更新できます。これによりクエリパフォーマンスが最適化され、文字列データに対する2倍のクエリパフォーマンスが提供されます。
- CREATE TABLE AS SELECTステートメントを非同期で実行できるようになりました。詳細については、[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を参照してください。
- 次のリソースグループ関連の機能をサポートします：
  - モニタリングリソースグループ：監査ログでクエリのリソースグループを表示し、APIを呼び出すことでリソースグループのメトリクスを取得できます。詳細については、[モニタリングとアラート](../administration/Monitor_and_Alert.md#monitor-and-alerting)を参照してください。
  - 大規模なクエリのCPU、メモリ、I/Oリソースの消費を制限します：クラシファイアに基づいてクエリを特定のリソースグループにルーティングしたり、セッション変数を構成したりして、クエリを特定のリソースグループにルーティングできます。詳細については、[リソースグループ](../administration/resource_group.md)を参照してください。
- JDBC外部テーブルを使用して、Oracle、PostgreSQL、MySQL、SQLServer、ClickHouse、およびその他のデータベースのデータを便利にクエリできます。StarRocksは述部プッシュダウンもサポートしており、クエリパフォーマンスが向上します。詳細については、[JDBC互換データベース用の外部テーブル](../data_source/External_table.md#external-table-for-a-JDBC-compatible-database)を参照してください。
- [プレビュー] 新しいデータソースコネクタフレームワークがリリースされ、外部カタログをサポートします。外部テーブルを作成せずにHiveデータに直接アクセスおよびクエリできる外部カタログを使用できます。詳細については、[内部および外部データの管理にカタログを使用](../data_source/catalog/query_external_data.md)を参照してください。
- 次の関数が追加されました：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md), [base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md), [array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md), [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改善

- コンパクションメカニズムは大量のメタデータを高速にマージできるようになりました。これにより、頻繁なデータ更新後に発生するメタデータの圧縮と過剰なディスク使用を防ぎます。
- Parquetファイルと圧縮ファイルの読み込みのパフォーマンスが最適化されました。
- マテリアライズドビューの作成メカニズムが最適化されました。最適化により、マテリアライズドビューの作成速度が従来の10倍速くなりました。
- 次の演算子のパフォーマンスが最適化されました：
  - TopNおよびソート演算子
  - 関数を含む等価比較演算子は、これらの演算子がスキャン演算子にプッシュダウンされるときにZone Mapインデックスを使用できます。
- Apache Hive™外部テーブルが最適化されました。
  - Apache Hive™テーブルがParquet、ORC、またはCSV形式で保存されている場合、HiveでADD COLUMNまたはREPLACE COLUMNによるスキーマ変更があった場合、それに対応するHive外部テーブルでREFRESHステートメントが実行されると、StarRocksに同期されます。詳細については、[Hive外部テーブル](../data_source/External_table.md#hive-external-table)を参照してください。
  - `hive.metastore.uris`をHiveリソース用に変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。
- Apache Iceberg外部テーブルのパフォーマンスが最適化されました。カスタムカタログを使用してIcebergリソースを作成できるようになりました。詳細については、[Apache Iceberg外部テーブル](../data_source/External_table.md#apache-iceberg-external-table)を参照してください。
- Elasticsearch外部テーブルのパフォーマンスが最適化されました。Elasticsearchクラスターのデータノードのアドレスをスニッフィングすることが無効になります。詳細については、[Elasticsearch外部テーブル](../data_source/External_table.md#elasticsearch-external-table)を参照してください。
- sum()関数が数値文字列を暗黙的に変換できるようになりました。
- year()、month()、day()関数がDATEデータ型をサポートします。

### バグ修正

以下のバグが修正されました：

- 過剰なタブレットのためにCPU利用率が急上昇します。
- "fail to prepare tablet reader"が発生する原因に関する問題
- FEsが再起動に失敗する。[#5642](https://github.com/StarRocks/starrocks/issues/5642 )  [#4969](https://github.com/StarRocks/starrocks/issues/4969 )  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- CTASステートメントにJSON関数が含まれている場合、ステートメントを正常に実行できない。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### その他

- StarGoはクラスタ管理ツールで、複数のクラスタを展開、起動、アップグレード、ロールバックし、複数のクラスタを管理できる。詳細については、[StarGoを使用したStarRocksの展開](../administration/stargo.md)を参照してください。
- パイプラインエンジンは、StarRocksをバージョン2.3にアップグレードするか、StarRocksを展開するとデフォルトで有効になります。パイプラインエンジンを使用すると、高い同時実行シナリオおよび複雑なクエリにおけるシンプルなクエリのパフォーマンスが向上します。StarRocks 2.3を使用している際にパフォーマンスの大幅な低下が検出された場合、`SET GLOBAL`ステートメントを実行して`enable_pipeline_engine`を`false`に設定することでパイプラインエンジンを無効にできます。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)ステートメントは、MySQLの構文と互換性があり、GRANTステートメントの形式でユーザに割り当てられた権限を表示します。
- schema_change（BE設定項目）の`memory_limitation_per_thread_for_schema_change`は、デフォルト値の2 GBを使用し、データのボリュームがこの制限を超えるとデータがディスクに書き込まれます。したがって、以前にこのパラメータをより大きな値に設定していた場合は、2 GBに設定することをおすすめします。そうしないと、スキーマ変更タスクが大量のメモリを消費する可能性があります。

### アップグレードの注意事項

アップグレード前に使用していた以前のバージョンにロールバックするには、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを追加し、パラメータを`true`に設定してください。このパラメータは、StarRocks v2.2.0で新しいタイプのログが追加されたため必要です。パラメータを追加しないと、以前のバージョンにロールバックできません。チェックポイントが作成された後に、**fe.conf**ファイルの`ignore_unknown_log_id`パラメータをそれぞれのFEに`false`に設定することをお勧めします。その後、再起動して以前の構成にFEを復元してください。