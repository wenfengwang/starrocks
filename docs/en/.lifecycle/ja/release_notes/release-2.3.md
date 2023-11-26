---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.3

## 2.3.18

リリース日: 2023年10月11日

### バグ修正

以下の問題が修正されました:

- サードパーティライブラリ librdkafka のバグにより、Routine Load ジョブのロードタスクがデータロード中にスタックし、新しく作成されたロードタスクも実行に失敗します。[#28301](https://github.com/StarRocks/starrocks/pull/28301)
- Spark や Flink のコネクタが正確なメモリ統計情報を持たないため、データのエクスポートに失敗する場合があります。[#31200](https://github.com/StarRocks/starrocks/pull/31200) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- Stream Load ジョブが `if` キーワードを使用する場合、BE がクラッシュすることがあります。[#31926](https://github.com/StarRocks/starrocks/pull/31926)
- パーティション化された StarRocks 外部テーブルにデータをロードすると、エラー `"get TableMeta failed from TNetworkAddress"` が報告されます。[#30466](https://github.com/StarRocks/starrocks/pull/30466)

## 2.3.17

リリース日: 2023年9月4日

### バグ修正

以下の問題が修正されました:

- Routine Load ジョブがデータを消費できない場合があります。[#29883](https://github.com/StarRocks/starrocks/issues/29883) [#18550](https://github.com/StarRocks/starrocks/pull/18550)

## 2.3.16

リリース日: 2023年8月4日

### バグ修正

以下の問題が修正されました:

ブロックされた LabelCleaner スレッドによる FE メモリリーク。[#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 2.3.15

リリース日: 2023年7月31日

## 改善点

- タブレットのスケジューリングロジックを最適化し、特定の状況下でタブレットが長時間保留状態になるか、FE がクラッシュすることを防ぎます。[#21647](https://github.com/StarRocks/starrocks/pull/21647) [#23062](https://github.com/StarRocks/starrocks/pull/23062) [#25785](https://github.com/StarRocks/starrocks/pull/25785)
- TabletChecker のスケジューリングロジックを最適化し、修復されていないタブレットを繰り返しスケジュールすることを防ぎます。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- パーティションメタデータには visibleTxnId が記録され、これはタブレットレプリカの可視バージョンに対応します。レプリカのバージョンが他のバージョンと一致しない場合、このバージョンを作成したトランザクションを追跡するのが容易になります。[#27924](https://github.com/StarRocks/starrocks/pull/27924)

### バグ修正

以下の問題が修正されました:

- FEs におけるテーブルレベルのスキャン統計が正しくないため、テーブルクエリやロードに関連するメトリクスが正確ではありません。[#28022](https://github.com/StarRocks/starrocks/pull/28022)
- Join キーが大きな BINARY 列の場合、BE がクラッシュする場合があります。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 集計演算子が特定のシナリオでスレッドセーフの問題を引き起こし、BE がクラッシュする場合があります。[#26092](https://github.com/StarRocks/starrocks/pull/26092)
- データが [RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) を使用して復元された後、BE と FE の間でタブレットのバージョン番号が一致しない場合があります。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- テーブルの復旧後にパーティションが自動的に作成されない場合があります。[RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) を使用してテーブルを復旧した後。[#26813](https://github.com/StarRocks/starrocks/pull/26813)
- INSERT INTO を使用してロードするデータが品質要件を満たさず、データロードのために厳密モードが有効になっている場合、ロードトランザクションは保留状態になり、DDL ステートメントがハングします。[#27140](https://github.com/StarRocks/starrocks/pull/27140)
- 低カーディナリティ最適化が有効になっている場合、一部の INSERT ジョブは `[42000][1064] Dict Decode failed, Dict can't take cover all key :0` を返す場合があります。[#27395](https://github.com/StarRocks/starrocks/pull/27395)
- 特定のケースでは、パイプラインが有効でない場合、INSERT INTO SELECT 操作がタイムアウトすることがあります。[#26594](https://github.com/StarRocks/starrocks/pull/26594)
- クエリ条件が `WHERE partition_column < xxx` であり、`xxx` の値が時間に対して分と秒ではなく時まで正確な場合、クエリはデータを返しません。例えば、`2023-7-21 22`。[#27780](https://github.com/StarRocks/starrocks/pull/27780)

## 2.3.14

リリース日: 2023年6月28日

### 改善点

- CREATE TABLE がタイムアウトした場合に返されるエラーメッセージを最適化し、パラメータのチューニングのヒントを追加しました。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- 大量のタブレットバージョンを持つ主キーテーブルのメモリ使用量を最適化しました。[#20760](https://github.com/StarRocks/starrocks/pull/20760)
- StarRocks 外部テーブルメタデータの同期をデータロード中に行うように変更しました。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- ネットワーク時刻の依存関係をシステムクロックから削除し、一貫性のないシステムクロックによる不正なネットワーク時刻を修正しました。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題が修正されました:

- 小さなテーブルに対して低カーディナリティ辞書最適化が適用される場合、TRUNCATE 操作が頻繁に行われると、クエリでエラーが発生する可能性があります。[#23185](https://github.com/StarRocks/starrocks/pull/23185)
- UNION の最初の子が定数 NULL の UNION を含むビューがクエリされると、BE がクラッシュする場合があります。[#13792](https://github.com/StarRocks/starrocks/pull/13792)  
- 一部の場合、ビットマップインデックスに基づくクエリがエラーを返す場合があります。[#23484](https://github.com/StarRocks/starrocks/pull/23484)
- BE で DOUBLE または FLOAT 値を DECIMAL 値に丸めると、FE での結果と一致しない結果が返されます。[#23152](https://github.com/StarRocks/starrocks/pull/23152)
- スキーマ変更が同時にデータロードが発生する場合、スキーマ変更がハングする場合があります。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- Broker Load、Spark コネクタ、Flink コネクタを使用して Parquet ファイルを StarRocks にロードする場合、BE OOM の問題が発生する場合があります。[#25254](https://github.com/StarRocks/starrocks/pull/25254)
- クエリの ORDER BY 句に定数が指定され、クエリに LIMIT 句がある場合、エラーメッセージ `unknown error` が返されます。[#25538](https://github.com/StarRocks/starrocks/pull/25538)

## 2.3.13

リリース日: 2023年6月1日

### 改善点

- INSERT INTO ... SELECT がタイムアウトすると報告されるエラーメッセージを最適化し、パラメータのチューニングのヒントを追加しました。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- `bitmap_contains` 関数を使用する複数テーブルの結合におけるメモリ消費を削減し、パフォーマンスを最適化しました。[#20617](https://github.com/StarRocks/starrocks/pull/20617) [#20653](https://github.com/StarRocks/starrocks/pull/20653)

### バグ修正

以下のバグが修正されました:

- パーティションのトランケート操作が失敗する場合があります。トランケート操作はパーティション名の大文字と小文字を区別します。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- Parquet ファイルから int96 タイムスタンプデータをロードすると、データオーバーフローが発生します。[#22355](https://github.com/StarRocks/starrocks/issues/22355)
- マテリアライズドビューが削除された後に BE のデコミッションが失敗します。[#22743](https://github.com/StarRocks/starrocks/issues/22743)
- クエリの実行計画に Broadcast Join の後に Bucket Shuffle Join が続く場合（例：`SELECT * FROM t1 JOIN [Broadcast] t2 ON t1.a = t2.b JOIN [Bucket] t3 ON t2.b = t3.c`）、Broadcast Join の左側のテーブルの等値結合キーのデータが Bucket Shuffle Join に送信される前に削除されると、BE がクラッシュする場合があります。[#23227](https://github.com/StarRocks/starrocks/pull/23227)
- クエリの実行計画に Cross Join の後に Hash Join が続く場合、フラグメントインスタンス内の Hash Join の右側のテーブルが空の場合、返される結果が正しくありません。[#23877](https://github.com/StarRocks/starrocks/pull/23877)
- マテリアライズドビューの一時パーティション作成に失敗することで、BE のデコミッションが失敗します。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- SQL ステートメントに複数のエスケープ文字を含む STRING 値が指定されている場合、SQL ステートメントを解析できません。[#23119](https://github.com/StarRocks/starrocks/issues/23119)
- パーティション列の値が最大値であるデータをクエリすると失敗します。[#23153](https://github.com/StarRocks/starrocks/issues/23153)
- StarRocks を v2.4 から v2.3 にロールバックした後、計算式が `COLUMN` パラメータで指定された計算式を持つ Routine Load ジョブが作成されると、エラー `No match for <expr> with operand types xxx and xxx` が発生する場合があります。[#17856](https://github.com/StarRocks/starrocks/pull/17856)
- BE の再起動後にロードジョブがハングアップします。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- OUTER JOIN を使用するクエリが BE ノードをクラッシュさせる場合があります。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- StarRocks をバージョン 2.4 で非同期マテリアライズドビューを作成し、それを 2.3 にロールバックすると、FE の起動に失敗する場合があります。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- 主キーテーブルで delete_range を使用し、パフォーマンスが良くない場合、RocksDB からのデータ読み取りが遅くなり、CPU 使用率が高くなる場合があります。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.12

リリース日: 2023年4月25日

### 改善点

返された式の値が有効なブール値に変換できる場合、暗黙の型変換をサポートします。[# 21792](https://github.com/StarRocks/starrocks/pull/21792)

### バグ修正

以下の問題が修正されました:

- ユーザの LOAD_PRIV がテーブルレベルで許可されている場合、ロードジョブの失敗時にトランザクションロールバックでエラーメッセージ `Access denied; you need (at least one of) the LOAD privilege(s) for this operation` が返されます。[# 21129](https://github.com/StarRocks/starrocks/issues/21129)
- ALTER SYSTEM DROP BACKEND を実行して BE を削除した後、その BE にレプリケーション数が 2 に設定されているテーブルのレプリカを修復できません。この状況では、これらのテーブルへのデータロードが失敗します。[# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- CREATE TABLE でサポートされていないデータ型を使用すると、NPE が返されます。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- ブロードキャスト結合のショートサーキットロジックが異常であり、クエリ結果が正しくありません。[# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- マテリアライズドビューが使用されるとディスク使用量が大幅に増加します。[# 20590](https://github.com/StarRocks/starrocks/pull/20590)
- オーディットローダープラグインを完全にアンインストールできません。[# 20468](https://github.com/StarRocks/starrocks/issues/20468)
- `INSERT INTO XXX SELECT` の結果の行数が `SELECT COUNT(*) FROM XXX` の結果と一致しない場合があります。[# 20084](https://github.com/StarRocks/starrocks/issues/20084)
- サブクエリがウィンドウ関数を使用し、親クエリが GROUP BY 句を使用する場合、クエリ結果を集計できません。[# 19725](https://github.com/StarRocks/starrocks/issues/19725)
- BE が起動されると、BE プロセスは存在しますが、すべての BE のポートが開かれません。[# 19347](https://github.com/StarRocks/starrocks/pull/19347)
- ディスク I/O が非常に高い場合、主キーテーブルのトランザクションが遅くコミットされ、結果としてこれらのテーブルのクエリがエラー "backend not found" を返す場合があります。[# 18835](https://github.com/StarRocks/starrocks/issues/18835)

## 2.3.11

リリース日: 2023年3月28日

### 改善点

- 複数の式を含む複雑なクエリを実行すると、多数の `ColumnRefOperators` が生成されます。元々、StarRocks は `BitSet` を使用して `ColumnRefOperator::id` を格納していましたが、これにより大量のメモリが消費されました。メモリ使用量を削減するために、StarRocks は `RoaringBitMap` を使用して `ColumnRefOperator::id` を格納するようになりました。[#16499](https://github.com/StarRocks/starrocks/pull/16499)
- 大規模なクエリが小規模なクエリに対して与えるパフォーマンスへの影響を軽減するために、新しい I/O スケジューリング戦略を導入しました。新しい I/O スケジューリング戦略を有効にするには、**be.conf** で BE の静的パラメータ `pipeline_scan_queue_mode=1` を設定して BE を再起動します。[#19009](https://github.com/StarRocks/starrocks/pull/19009)

### バグ修正

以下の問題が修正されました:

- 大規模なクエリが終了した後にリソースが解放されると、他のクエリが遅くなる可能性があります。この問題は、リソースグループが有効になっている場合や、大規模なクエリが予期せず終了した場合により頻繁に発生する可能性があります。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- 主キーテーブルの場合、レプリカのメタデータバージョンが遅れている場合、StarRocks は他のレプリカからこのレプリカに欠落しているメタデータを増分的にクローンします。このプロセスでは、StarRocks はメタデータの多数のバージョンをプルし、メモリが適切に GC されないまま多数のメタデータのバージョンが蓄積されると、過剰なメモリが消費され、BE が OOM 例外に遭遇する可能性があります。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- FE が BE に定期的なハートビートを送信し、ハートビート接続がタイムアウトすると、FE は BE が利用できないと見なし、BE 上のトランザクションが失敗する場合があります。[# 16386](https://github.com/StarRocks/starrocks/pull/16386)
- StarRocks 外部テーブルを使用して StarRocks クラスタ間でデータをロードする場合、ソース StarRocks クラスタがより古いバージョンであり、ターゲット StarRocks クラスタがより新しいバージョン（2.2.8 〜 2.2.11、2.3.4 〜 2.3.7、2.4.1 または 2.4.2）である場合、データロードが失敗します。[#16173](https://github.com/StarRocks/starrocks/pull/16173)
- BE が高負荷の場合、リソースグループのメトリクス `starrocks_be_resource_group_running_queries` が正しくない場合があります。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- クエリステートメントが OUTER JOIN を使用する場合、BE ノードがクラッシュする可能性があります。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- StarRocks をバージョン 2.4 で非同期マテリアライズドビューを作成し、それを 2.3 にロールバックすると、FE の起動に失敗する場合があります。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- 主キーテーブルが delete_range を使用し、パフォーマンスが良くない場合、RocksDB からのデータ読み取りが遅くなり、CPU 使用率が高くなる場合があります。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.10

リリース日: 2023年2月9日

### バグ修正

以下の問題が修正されました:

- スキーマ変更中にタブレットクローンがトリガされ、タブレットレプリカが存在する BE ノードが変更されると、スキーマ変更が失敗します。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- group_concat() 関数が返す文字列が切り捨てられます。[#16948](https://github.com/StarRocks/starrocks/pull/16948)
- HDFS を介して Tencent Big Data Suite (TBDS) を使用してデータをロードする場合、`invalid hadoop.security.authentication.tbds.securekey` エラーが発生し、TBDS が提供する認証情報を使用して StarRocks が HDFS にアクセスできないことを示します。[#14125](https://github.com/StarRocks/starrocks/pull/14125) [#15693](https://github.com/StarRocks/starrocks/pull/15693)
- 一部の場合、CBO が2つの演算子が等価であるかどうかを比較するためのロジックを誤って使用する場合があります。[#17227](https://github.com/StarRocks/starrocks/pull/17227) [#17199](https://github.com/StarRocks/starrocks/pull/17199)
- 非リーダー FE ノードに接続し、SQL ステートメント `USE <catalog_name>.<database_name>` を送信すると、非リーダー FE ノードは SQL ステートメントをリーダー FE ノードに転送します（`<catalog_name>` は除外）。その結果、リーダー FE ノードは `default_catalog` を使用するように選択し、指定したデータベースを見つけることができません。[#17302](https://github.com/StarRocks/starrocks/pull/17302)

## 2.3.8

リリース日: 2023年2月2日

### バグ修正

以下の問題が修正されました:

- 大規模なクエリが終了した後、他のクエリが遅くなる場合があります。この問題は、リソースグループが有効になっている場合や、大規模なクエリが予期せず終了した場合により頻繁に発生する可能性があります。[#16454](https://github.com/StarRocks/starrocks/pull/16454) [#16602](https://github.com/StarRocks/starrocks/pull/16602)
- 主キーテーブルの場合、レプリカのメタデータバージョンが遅れている場合、StarRocks は他のレプリカからこのレプリカに欠落しているメタデータを増分的にクローンします。このプロセスでは、StarRocks はメタデータの多数のバージョンをプルし、メモリが適切に GC されないまま多数のメタデータのバージョンが蓄積されると、過剰なメモリが消費され、BE が OOM 例外に遭遇する可能性があります。[#15935](https://github.com/StarRocks/starrocks/pull/15935)
- INSERT INTO を使用して頻繁にデータを主キーテーブルにロードし、データの変更を行った後すぐに BE を再起動すると、BE の再起動が非常に遅くなる場合があります。[#15128](https://github.com/StarRocks/starrocks/pull/15128)
- 環境に JRE のみがインストールされ、JDK がインストールされていない場合、FE の再起動後にクエリが失敗します。バグが修正された後、FE はその環境で再起動できず、エラー `JAVA_HOME can not be jre` を返します。FE を正常に再起動するには、環境に JDK をインストールする必要があります。[#14332](https://github.com/StarRocks/starrocks/pull/14332)
- クエリが BE のクラッシュを引き起こす場合があります。[#14221](https://github.com/StarRocks/starrocks/pull/14221)
- `exec_mem_limit` を式に設定できません。[#13647](https://github.com/StarRocks/starrocks/pull/13647)
- サブクエリの結果に基づいて同期リフレッシュされたマテリアライズドビューを作成することはできません。[#13507](https://github.com/StarRocks/starrocks/pull/13507)
- リフレッシュした Hive 外部テーブルのコメントが削除されます。[#13742](https://github.com/StarRocks/starrocks/pull/13742)
- 相関 JOIN 中、右のテーブルが左のテーブルよりも先に処理され、右のテーブルが非常に大きい場合、左のテーブルでコンパクションが実行されると、BE ノードがクラッシュする場合があります。[#14070](https://github.com/StarRocks/starrocks/pull/14070)
- Parquet ファイルの列名が大文字と小文字を区別する場合、クエリ条件が Parquet ファイルの大文字の列名を使用し、クエリが結果を返さない場合があります。[#13860](https://github.com/StarRocks/starrocks/pull/13860) [#14773](https://github.com/StarRocks/starrocks/pull/14773)
- ブローカーロード時に接続数がデフォルトの最大接続数を超えると、ブローカーが切断され、ロードジョブが `list path error` のエラーメッセージで失敗します。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- BE が高負荷の場合、リソースグループのメトリクス `starrocks_be_resource_group_running_queries` が正しくない場合があります。[#14043](https://github.com/StarRocks/starrocks/pull/14043)
- クエリステートメントが OUTER JOIN を使用する場合、BE ノードがクラッシュする可能性があります。[#14840](https://github.com/StarRocks/starrocks/pull/14840)
- StarRocks をバージョン 2.4 で非同期マテリアライズドビューを作成し、それを 2.3 にロールバックすると、FE の起動に失敗する場合があります。[#14400](https://github.com/StarRocks/starrocks/pull/14400)
- 主キーテーブルが delete_range を使用し、パフォーマンスが良くない場合、RocksDB からのデータ読み取りが遅くなり、CPU 使用率が高くなる場合があります。[#15130](https://github.com/StarRocks/starrocks/pull/15130)

## 2.3.7

リリース日: 2022年12月30日

### バグ修正

以下の問題が修正されました:

- StarRocks テーブルで NULL が許可されている列は、ビューで作成された場合には誤って NOT NULL に設定されます。[#15749](https://github.com/StarRocks/starrocks/pull/15749)
- StarRocks にデータをロードすると、新しいタブレットバージョンが生成されます。しかし、FE はまだ新しいタブレットバージョンを検出せず、BE に対してタブレットの過去のバージョンを読み取るように要求します。ガベージコレクションメカニズムが過去のバージョンを削除すると、クエリは過去のバージョンを見つけることができず、エラー "Not found: get_applied_rowsets(version xxxx) failed tablet:xxx #version:x [xxxxxxx]" が返されます。[#15726](https://github.com/StarRocks/starrocks/pull/15726)
- データの頻繁なロード時に FE が過剰にメモリを使用します。[#15377](https://github.com/StarRocks/starrocks/pull/15377)
- 集計クエリや複数テーブルの JOIN クエリの統計情報が正確に収集されず、実行計画に CROSS JOIN が発生し、クエリの待ち時間が長くなります。[#12067](https://github.com/StarRocks/starrocks/pull/12067)  [#14780](https://github.com/StarRocks/starrocks/pull/14780)

## 2.3.6

リリース日: 2022年12月22日

### 改善点

- Colocate Join が Equi Join をサポートします。[#13546](https://github.com/StarRocks/starrocks/pull/13546)
- データの頻繁なロード時に主キーテーブルの主キーインデックスファイルが非常に大きくなる問題を修正しました。[#12862](https://github.com/StarRocks/starrocks/pull/12862)
- FE はタブレットをバッチでスキャンし、長時間 db.readLock を保持することを防ぐために、スキャン間隔で db.readLock を解放します。[#13070](https://github.com/StarRocks/starrocks/pull/13070)

### バグ修正

以下の問題が修正されました:

- UNION ALL の結果を直接ベースにしたビューが作成され、UNION ALL 演算子の入力列に NULL 値が含まれる場合、ビューのスキーマが正しくありません。列のデータ型は NULL_TYPE ではなく、UNION ALL の入力列です。[#13917](https://github.com/StarRocks/starrocks/pull/13917)
- `SELECT * FROM ...` と `SELECT * FROM ... LIMIT ...` のクエリ結果が一貫していません。[#13585](https://github.com/StarRocks/starrocks/pull/13585)
- FE に同期された外部タブレットメタデータがローカルのタブレットメタデータを上書きするため、Flink からのデータロードが失敗する場合があります。[#12579](https://github.com/StarRocks/starrocks/pull/12579)
- BEノードは、ランタイムフィルタでリテラル定数を処理する際に、nullフィルタがクラッシュします。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- CTASを実行するとエラーが返されます。[#12388](https://github.com/StarRocks/starrocks/pull/12388)
- オーディットログでパイプラインエンジンによって収集されるメトリック「ScanRows」が間違っている場合があります。[#12185](https://github.com/StarRocks/starrocks/pull/12185)
- 圧縮されたHIVEデータをクエリすると、クエリ結果が正しくありません。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- BEノードがクラッシュすると、クエリがタイムアウトし、StarRocksの応答が遅くなります。[#12955](https://github.com/StarRocks/starrocks/pull/12955)
- ブローカーロードを使用してデータをロードする際に、Kerberos認証の失敗エラーが発生します。[#13355](https://github.com/StarRocks/starrocks/pull/13355)
- OR述語が多すぎると、統計推定に時間がかかりすぎる場合があります。[#13086](https://github.com/StarRocks/starrocks/pull/13086)
- ブローカーロードが大文字の列名を含むORCファイル（Snappy圧縮）をロードすると、BEノードがクラッシュします。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- プライマリキーテーブルのアンロードまたはクエリに30分以上かかると、エラーが返されます。[#13403](https://github.com/StarRocks/starrocks/pull/13403)
- ブローカーを使用して大量のデータをHDFSにバックアップすると、バックアップタスクが失敗します。[#12836](https://github.com/StarRocks/starrocks/pull/12836)
- StarRocksがIcebergから読み取ったデータが正しくない場合があります。これは、`parquet_late_materialization_enable`パラメータによるものです。[#13132](https://github.com/StarRocks/starrocks/pull/13132)
- ビューが作成されるとエラー「failed to init view stmt」が返されます。[#13102](https://github.com/StarRocks/starrocks/pull/13102)
- JDBCを使用してStarRockに接続し、SQLステートメントを実行するとエラーが返されます。[#13526](https://github.com/StarRocks/starrocks/pull/13526)
- クエリがタイムアウトするのは、クエリが多くのバケットを含み、タブレットヒントを使用しているためです。[#13272](https://github.com/StarRocks/starrocks/pull/13272)
- BEノードがクラッシュし、再起動できない場合、新しく構築されたテーブルへのローディングジョブがエラーを報告します。[#13701](https://github.com/StarRocks/starrocks/pull/13701)
- マテリアライズドビューが作成されると、すべてのBEノードがクラッシュします。[#13184](https://github.com/StarRocks/starrocks/pull/13184)
- ALTER ROUTINE LOADを実行して消費パーティションのオフセットを更新すると、「指定されたパーティション1は消費パーティションにありません」というエラーが返され、フォロワーが最終的にクラッシュします。[#12227](https://github.com/StarRocks/starrocks/pull/12227)

## 2.3.4

リリース日：2022年11月10日

### 改善点

- StarRocksが実行中のRoutine Loadジョブの数が制限を超えるためにRoutine Loadジョブを作成できない場合、エラーメッセージに解決策が表示されます。[#12204](https://github.com/StarRocks/starrocks/pull/12204)
- StarRocksがHiveからデータをクエリし、CSVファイルを解析できない場合、エラーが発生します。[#13013](https://github.com/StarRocks/starrocks/pull/13013)

### バグ修正

以下のバグが修正されています：

- HDFSファイルパスに`()`が含まれる場合、クエリが失敗する場合があります。[#12660](https://github.com/StarRocks/starrocks/pull/12660)
- サブクエリにLIMITが含まれる場合、ORDER BY ... LIMIT ... OFFSETの結果が正しくありません。[#9698](https://github.com/StarRocks/starrocks/issues/9698)
- StarRocksは、ORCファイルをクエリする際に大文字と小文字を区別しません。[#12724](https://github.com/StarRocks/starrocks/pull/12724)
- RuntimeFilterが準備メソッドを呼び出さずに閉じられると、BEがクラッシュする場合があります。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- メモリリークにより、BEがクラッシュする場合があります。[#12906](https://github.com/StarRocks/starrocks/issues/12906)
- 新しい列を追加してすぐにデータを削除すると、クエリ結果が正しくない場合があります。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- データのソートにより、BEがクラッシュする場合があります。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- StarRocksとMySQLクライアントが同じLANにない場合、INSERT INTO SELECTを使用して作成されたローディングジョブは、KILLを1回だけ実行しても正常に終了できません。[#11879](https://github.com/StarRocks/starrocks/pull/11897)
- オーディットログでパイプラインエンジンによって収集されるメトリック「ScanRows」が間違っている場合があります。[#12185](https://github.com/StarRocks/starrocks/pull/12185)

## 2.3.3

リリース日：2022年9月27日

### バグ修正

以下のバグが修正されています：

- Hive外部テーブルをクエリする際、Hive外部テーブルがテキストファイルとして保存されている場合、クエリ結果が正確でない場合があります。[#11546](https://github.com/StarRocks/starrocks/pull/11546)
- Parquetファイルをクエリする際、ネストされた配列はサポートされていません。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- StarRocksからデータを読み取るクエリまたはクエリがタイムアウトする場合、StarRocksと外部データソースからデータを読み取る並行クエリが同じリソースグループにルーティングされるか、クエリがStarRocksと外部データソースからデータを読み取る場合、クエリがタイムアウトする場合があります。[#10983](https://github.com/StarRocks/starrocks/pull/10983)
- パイプライン実行エンジンがデフォルトで有効になっている場合、パラメータparallel_fragment_exec_instance_numが1に変更されます。これにより、INSERT INTOを使用してデータをロードする際にデータのローディングが遅くなります。[#11462](https://github.com/StarRocks/starrocks/pull/11462)
- 式の初期化時にエラーが発生すると、BEがクラッシュする場合があります。[#11396](https://github.com/StarRocks/starrocks/pull/11396)
- ORDER BY LIMITを実行すると、エラー「heap-buffer-overflow」が発生する場合があります。[#11185](https://github.com/StarRocks/starrocks/pull/11185)
- Leader FEを再起動するとスキーマ変更が失敗する場合があります。[#11561](https://github.com/StarRocks/starrocks/pull/11561)

## 2.3.2

リリース日：2022年9月7日

### 新機能

- レンジフィルタベースのクエリを高速化するために、遅延マテリアライゼーションがサポートされています。外部テーブル（Parquet形式）のクエリパフォーマンスが2倍に向上します。[#9738](https://github.com/StarRocks/starrocks/pull/9738)
- `SHOW AUTHENTICATION`ステートメントが追加され、ユーザーの認証関連情報を表示できます。[#9996](https://github.com/StarRocks/starrocks/pull/9996)

### 改善点

- StarRocksがStarRocksからクエリデータを取得する際に、バケット化されたHiveテーブルのすべてのデータファイルを再帰的にトラバースするかどうかを制御する設定項目が提供されます。[#10239](https://github.com/StarRocks/starrocks/pull/10239)
- リソースグループタイプ`realtime`が`short_query`として名前が変更されました。[#10247](https://github.com/StarRocks/starrocks/pull/10247)
- StarRocksは、デフォルトでHive外部テーブルの大文字と小文字を区別しません。[#10187](https://github.com/StarRocks/starrocks/pull/10187)

### バグ修正

以下のバグが修正されています：

- Elasticsearch外部テーブルのクエリが予期せず終了する場合があります。これは、テーブルが複数のシャードに分割されている場合に発生します。[#10369](https://github.com/StarRocks/starrocks/pull/10369)
- サブクエリが共通テーブル式（CTE）として書き換えられた場合、StarRocksはエラーをスローします。[#10397](https://github.com/StarRocks/starrocks/pull/10397)
- 大量のデータをロードすると、StarRocksはエラーをスローします。[#10370](https://github.com/StarRocks/starrocks/issues/10370) [#10380](https://github.com/StarRocks/starrocks/issues/10380)
- 複数のカタログに同じThriftサービスIPアドレスが設定されている場合、1つのカタログを削除すると、他のカタログの増分メタデータの更新が無効になります。[#10511](https://github.com/StarRocks/starrocks/pull/10511)
- BEからのメモリ消費の統計情報が正確ではありません。[#9837](https://github.com/StarRocks/starrocks/pull/9837)
- Primary Keyテーブルのクエリでエラーが発生します。[#10811](https://github.com/StarRocks/starrocks/pull/10811)
- ロジカルビューのクエリは許可されません。これにもかかわらず、これらのビューに対してSELECT権限を持っている場合でもです。[#10563](https://github.com/StarRocks/starrocks/pull/10563)
- StarRocksは、ロジカルビューの命名に制限を課しません。現在、ロジカルビューはテーブルと同じ命名規則に従う必要があります。[#10558](https://github.com/StarRocks/starrocks/pull/10558)

### 動作の変更

- BEの設定項目`max_length_for_bitmap_function`が追加されました。デフォルト値は1000000で、ビットマップ関数のためのものです。また、`max_length_for_to_base64`も追加されました。デフォルト値は200000で、base64のためのものです。これにより、クラッシュが防止されます。[#10851](https://github.com/StarRocks/starrocks/pull/10851)

## 2.3.1

リリース日：2022年8月22日

### 改善点

- Broker Loadは、Parquetファイル内のList型をネストされていないARRAYデータ型に変換することができます。[#9150](https://github.com/StarRocks/starrocks/pull/9150)
- JSON関連の関数（json_query、get_json_string、get_json_int）のパフォーマンスが最適化されました。[#9623](https://github.com/StarRocks/starrocks/pull/9623)
- エラーメッセージが最適化されました。Hive、Iceberg、またはHudiのクエリ中に、StarRocksがサポートしていない列のデータ型をクエリしようとすると、システムは列に対して例外をスローします。[#10139](https://github.com/StarRocks/starrocks/pull/10139)
- リソースグループのスケジューリングレイテンシを削減し、リソースの分離パフォーマンスを最適化しました。[#10122](https://github.com/StarRocks/starrocks/pull/10122)

### バグ修正

以下のバグが修正されています：

- Elasticsearch外部テーブルのクエリが`limit`演算子のプッシュダウンが正しく行われないため、誤った結果が返される場合があります。[#9952](https://github.com/StarRocks/starrocks/pull/9952)
- Oracle外部テーブルのクエリで`limit`演算子が使用されると失敗します。[#9542](https://github.com/StarRocks/starrocks/pull/9542)
- BEがブロックされる場合があります。すべてのKafkaブローカーがRoutine Load中に停止された場合。[#9935](https://github.com/StarRocks/starrocks/pull/9935)
- BEがクラッシュする場合があります。Parquetファイルのデータ型が対応する外部テーブルのデータ型と一致しない場合。[#10107](https://github.com/StarRocks/starrocks/pull/10107)
- クエリがタイムアウトする場合があります。外部テーブルのスキャン範囲が空の場合。[#10091](https://github.com/StarRocks/starrocks/pull/10091)
- サブクエリにORDER BY句が含まれる場合、システムは例外をスローします。[#10180](https://github.com/StarRocks/starrocks/pull/10180)
- Hiveメタストアがハングする場合があります。Hiveメタデータが非同期で再ロードされる場合。[#10132](https://github.com/StarRocks/starrocks/pull/10132)

## 2.3.0

リリース日：2022年7月29日

### 新機能

- プライマリキーテーブルは、完全なDELETE WHERE構文をサポートしています。詳細については、[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#delete-data-by-primary-key)を参照してください。
- プライマリキーテーブルは、永続的なプライマリキーインデックスをサポートしています。メモリではなくディスクにプライマリキーインデックスを永続化することができます。これにより、メモリ使用量を大幅に削減できます。詳細については、[プライマリキーテーブル](../table_design/table_types/primary_key_table.md)を参照してください。
- グローバル辞書は、リアルタイムデータの取り込み中に更新することができます。これにより、クエリパフォーマンスが最適化され、文字列データのクエリパフォーマンスが2倍に向上します。
- CREATE TABLE AS SELECTステートメントは非同期で実行できます。詳細については、[CREATE TABLE AS SELECT](../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)を参照してください。
- リソースグループ関連の以下の機能をサポートします：
  - モニタリングリソースグループ：オーディットログでクエリのリソースグループを表示し、APIを呼び出してリソースグループのメトリックを取得できます。詳細については、[モニタリングとアラート](../administration/Monitor_and_Alert.md#monitor-and-alerting)を参照してください。
  - CPU、メモリ、およびI/Oリソースの大規模なクエリの消費を制限します：クエリを特定のリソースグループにルーティングするために、分類子を基にリソースグループを設定するか、セッション変数を設定することができます。詳細については、[リソースグループ](../administration/resource_group.md)を参照してください。
- JDBC外部テーブルを使用して、Oracle、PostgreSQL、MySQL、SQLServer、ClickHouseなどのデータベースのデータを簡単にクエリできます。StarRocksはプレディケートプッシュダウンもサポートしており、クエリパフォーマンスが向上します。詳細については、[JDBC互換データベースの外部テーブル](../data_source/External_table.md#external-table-for-a-jdbc-compatible-database)を参照してください。
- [プレビュー] 新しいデータソースコネクタフレームワークがリリースされ、外部カタログをサポートします。外部カタログを使用すると、外部テーブルを作成せずにHiveデータに直接アクセスしてクエリを実行できます。詳細については、[内部および外部データの管理にカタログを使用する](../data_source/catalog/query_external_data.md)を参照してください。
- 次の関数が追加されました：
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)
  - [ntile](../sql-reference/sql-functions/Window_function.md)
  - [bitmap_union_count](../sql-reference/sql-functions/bitmap-functions/bitmap_union_count.md)、[base64_to_bitmap](../sql-reference/sql-functions/bitmap-functions/base64_to_bitmap.md)、[array_to_bitmap](../sql-reference/sql-functions/array-functions/array_to_bitmap.md)
  - [week](../sql-reference/sql-functions/date-time-functions/week.md)、[time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)

### 改善点

- 大量のメタデータをより迅速にマージするために、コンパクションメカニズムを最適化しました。これにより、頻繁なデータ更新後のメタデータの圧縮や過剰なディスク使用量が防止されます。
- Parquetファイルと圧縮ファイルのロードパフォーマンスが最適化されました。
- マテリアライズドビューの作成メカニズムが最適化されました。最適化後、マテリアライズドビューの作成速度は以前の10倍速くなります。
- 次の演算子のパフォーマンスが最適化されました：
  - TopNおよびソート演算子
  - 関数を含む等価比較演算子は、これらの演算子がスキャン演算子にプッシュダウンされる場合、Zone Mapインデックスを使用できます。
- Apache Hive™外部テーブルが最適化されました。
  - Apache Hive™テーブルがParquet、ORC、またはCSV形式で保存されている場合、Hiveの対応する外部テーブルの対応するHive外部テーブルのREFRESHステートメントを実行すると、HiveでのADD COLUMNまたはREPLACE COLUMNによるスキーマ変更がStarRocksに同期されます。詳細については、[Hive外部テーブル](../data_source/External_table.md#hive-external-table)を参照してください。
  - `hive.metastore.uris`をHiveリソースに変更できます。詳細については、[ALTER RESOURCE](../sql-reference/sql-statements/data-definition/ALTER_RESOURCE.md)を参照してください。
- Apache Iceberg外部テーブルのパフォーマンスが最適化されました。カスタムカタログを使用してIcebergリソースを作成できます。詳細については、[Apache Iceberg外部テーブル](../data_source/External_table.md#apache-iceberg-external-table)を参照してください。
- Elasticsearch外部テーブルのパフォーマンスが最適化されました。Elasticsearchクラスタのデータノードのアドレスのスニッフィングを無効にすることができます。詳細については、[Elasticsearch外部テーブル](../data_source/External_table.md#elasticsearch-external-table)を参照してください。
- sum()関数が数値文字列を受け入れる場合、数値文字列を暗黙的に変換します。
- year()、month()、day()関数はDATEデータ型をサポートします。

### バグ修正

以下のバグが修正されています：

- 大量のタブレットが原因でCPU使用率が急上昇します。
- 「タブレットリーダーの準備に失敗」という問題が発生する場合があります。
- FEが再起動に失敗します。[#5642](https://github.com/StarRocks/starrocks/issues/5642 )  [#4969](https://github.com/StarRocks/starrocks/issues/4969 )  [#5580](https://github.com/StarRocks/starrocks/issues/5580)
- ステートメントにJSON関数が含まれる場合、CTASステートメントを正常に実行できません。[#6498](https://github.com/StarRocks/starrocks/issues/6498)

### その他

- クラスタ管理ツールであるStarGoを使用すると、クラスタをデプロイ、起動、アップグレード、ロールバックし、複数のクラスタを管理できます。詳細については、[StarGoでStarRocksをデプロイする](../administration/stargo.md)を参照してください。
- StarRocksをバージョン2.3にアップグレードするか、StarRocksをデプロイすると、パイプラインエンジンがデフォルトで有効になります。パイプラインエンジンは、高並行性のシンプルなクエリと複雑なクエリのパフォーマンスを向上させることができます。StarRocks 2.3を使用して重大なパフォーマンスの低下を検出した場合は、`SET GLOBAL`ステートメントを実行して`enable_pipeline_engine`を`false`に設定してパイプラインエンジンを無効にすることができます。
- [SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)ステートメントはMySQLの構文と互換性があり、GRANTステートメントの形式でユーザーに割り当てられた権限を表示します。
- アップグレード後に以前のバージョンにロールバックする場合は、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを追加し、パラメータを`true`に設定してください。このパラメータは、StarRocks v2.2.0で新しいタイプのログが追加されたため、必要です。パラメータを追加しない場合、以前のバージョンにロールバックできません。チェックポイントが作成された後、`ignore_unknown_log_id`パラメータを各FEの**fe.conf**ファイルに`false`に設定することをお勧めします。その後、FEを再起動して、FEを以前の設定に復元します。
