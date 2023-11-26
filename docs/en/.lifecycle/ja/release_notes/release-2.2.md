---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.2

リリース日: 2023年4月6日

## 改善点

- bitmap_contains() 関数を最適化し、メモリ消費を削減し、一部のシナリオでのパフォーマンスを向上させました。[#20616](https://github.com/StarRocks/starrocks/issues/20616)
- Compaction フレームワークを最適化し、CPUリソースの消費を削減しました。[#11746](https://github.com/StarRocks/starrocks/issues/11746)

## バグ修正

以下のバグが修正されました:

- Stream Load ジョブのリクエストURLが正しくない場合、責任を持つFEがハングし、HTTPリクエストを処理できなくなる問題を修正しました。[#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 責任を持つFEが統計情報を収集する際に、異常に大量のメモリを消費し、OOMが発生する問題を修正しました。[#16331](https://github.com/StarRocks/starrocks/issues/16331)
- クエリの一部でメモリ解放が正しく処理されない場合、BEがクラッシュする問題を修正しました。[#11395](https://github.com/StarRocks/starrocks/issues/11395)
- TRUNCATE TABLE コマンドが実行された後、NullPointerExceptionが発生し、責任を持つFEが再起動に失敗する場合がある問題を修正しました。[#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

リリース日: 2022年12月2日

### 改善点

- Routine Load ジョブのエラーメッセージの返却を最適化しました。[#12203](https://github.com/StarRocks/starrocks/pull/12203)

- 論理演算子 `&&` をサポートしました。[#11819](https://github.com/StarRocks/starrocks/issues/11819)

- BEがクラッシュした場合、クエリの有効期限が切れてシステムが停止する問題を防ぐため、クエリを即座にキャンセルするようにしました。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

- FEの起動スクリプトを最適化しました。FEの起動時にJavaのバージョンをチェックするようになりました。[#14094](https://github.com/StarRocks/starrocks/pull/14094)

- プライマリキーテーブルから大量のデータを削除することをサポートしました。[#4772](https://github.com/StarRocks/starrocks/issues/4772)

### バグ修正

以下のバグが修正されました:

- 複数のテーブルからビューを作成する場合（UNION）、UNION操作の左側の子がNULL定数を使用すると、BEがクラッシュする問題を修正しました。([#13792](https://github.com/StarRocks/starrocks/pull/13792))

- クエリするParquetファイルの列タイプがHiveテーブルスキーマと一貫していない場合、BEがクラッシュする問題を修正しました。[#8848](https://github.com/StarRocks/starrocks/issues/8848)

- クエリに大量のOR演算子が含まれる場合、プランナーは過剰な再帰計算を実行する必要があり、クエリがタイムアウトする問題を修正しました。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- サブクエリにLIMIT句が含まれる場合、クエリ結果が正しくありません。[#12466](https://github.com/StarRocks/starrocks/pull/12466)

- SELECT句で二重引用符と単一引用符が混在している場合、CREATE VIEW文が失敗する問題を修正しました。[#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

リリース日: 2022年11月15日

### 改善点

- Hiveパーティションの統計情報を収集するためのセッション変数 `hive_partition_stats_sample_size` を追加しました。過剰なパーティションの数はHiveメタデータの取得エラーを引き起こす可能性があります。[#12700](https://github.com/StarRocks/starrocks/pull/12700)

- Elasticsearch外部テーブルはカスタムタイムゾーンをサポートします。[#12662](https://github.com/StarRocks/starrocks/pull/12662)

### バグ修正

以下のバグが修正されました:

- 外部テーブルのメタデータ同期中にメタデータ同期中にエラーが発生した場合、DECOMMISSION操作がスタックする問題を修正しました。[#12369](https://github.com/StarRocks/starrocks/pull/12368)

- 新しく追加された列が削除された場合、Compactionがクラッシュする問題を修正しました。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- SHOW CREATE VIEWは、ビューの作成時に追加されたコメントを表示しません。[#4163](https://github.com/StarRocks/starrocks/issues/4163)

- Java UDFのメモリリークがOOMを引き起こす場合があります。[#12418](https://github.com/StarRocks/starrocks/pull/12418)

- Follower FEに格納されるノードの生存状態は、`heartbeatRetryTimes`に依存するため、一部のシナリオでは正確ではありません。この問題を修正するために、`HeartbeatResponse`に`aliveStatus`プロパティを追加し、ノードの生存状態を示すようにしました。[#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 動作の変更

StarRocksによってクエリで問い合わせることができるHive STRING列の長さが64 KBから1 MBに拡張されました。STRING列が1 MBを超える場合、クエリ中にnull列として処理されます。[#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

リリース日: 2022年10月17日

### バグ修正

以下のバグが修正されました:

- 初期化ステージで式がエラーに遭遇した場合、BEがクラッシュする問題を修正しました。[#11395](https://github.com/StarRocks/starrocks/pull/11395)

- 無効なJSONデータがロードされた場合、BEがクラッシュする問題を修正しました。[#10804](https://github.com/StarRocks/starrocks/issues/10804)

- パイプラインエンジンが有効になっている場合、並列書き込みがエラーになる問題を修正しました。[#11451](https://github.com/StarRocks/starrocks/issues/11451)

- ORDER BY NULL LIMIT句が使用された場合、BEがクラッシュする問題を修正しました。[#11648](https://github.com/StarRocks/starrocks/issues/11648)

- クエリするParquetファイルの列タイプがHiveテーブルスキーマと一貫していない場合、BEがクラッシュする問題を修正しました。[#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

リリース日: 2022年9月23日

### バグ修正

以下のバグが修正されました:

- ユーザーがJSONデータをStarRocksにロードするとデータが失われる場合があります。[#11054](https://github.com/StarRocks/starrocks/issues/11054)

- SHOW FULL TABLESの出力結果が正しくありません。[#11126](https://github.com/StarRocks/starrocks/issues/11126)

- 以前のバージョンでは、ビューのデータにアクセスするためには、ベーステーブルとビューの両方に対する権限が必要でした。現在のバージョンでは、ビューに対する権限のみが必要です。[#11290](https://github.com/StarRocks/starrocks/pull/11290)

- EXISTSまたはINでネストされた複雑なクエリの結果が正しくありません。[#11415](https://github.com/StarRocks/starrocks/pull/11415)

- 対応するHiveテーブルのスキーマが変更された場合、REFRESH EXTERNAL TABLEが失敗する問題を修正しました。[#11406](https://github.com/StarRocks/starrocks/pull/11406)

- リーダーではないFEがビットマップインデックスの作成操作を再生するとエラーが発生する場合があります。[#11261](

## 2.2.6

リリース日: 2022年9月14日

### バグ修正

以下のバグが修正されました:

- サブクエリにLIMITが含まれる場合、`order by... limit...offset` の結果が正しくありません。[#9698](https://github.com/StarRocks/starrocks/issues/9698)

- 大量のデータを持つテーブルで部分更新を実行すると、BEがクラッシュする問題を修正しました。[#9809](https://github.com/StarRocks/starrocks/issues/9809)

- BITMAPデータのサイズが2 GBを超える場合、CompactionがBEをクラッシュさせる問題を修正しました。[#11159](https://github.com/StarRocks/starrocks/pull/11159)

- パターンの長さが16 KBを超える場合、like()およびregexp()関数が機能しない問題を修正しました。[#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 動作の変更

出力で配列内のJSON値を表すために使用される形式が変更されました。エスケープ文字は返されたJSON値では使用されなくなりました。[#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

リリース日: 2022年8月18日

### 改善点

- パイプラインエンジンが有効になっている場合のシステムのパフォーマンスを向上させました。[#9580](https://github.com/StarRocks/starrocks/pull/9580)

- インデックスメタデータのメモリ統計の精度を向上させました。[#9837](https://github.com/StarRocks/starrocks/pull/9837)

### バグ修正

以下のバグが修正されました:

- Routine Load中にKafkaパーティションオフセット(`get_partition_offset`)のクエリがスタックする場合があります。[#9937](https://github.com/StarRocks/starrocks/pull/9937)

- 複数のBroker Loadスレッドが同じHDFSファイルをロードしようとするとエラーが発生する問題を修正しました。[#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

リリース日: 2022年8月3日

### 改善点

- Hiveテーブルのスキーマ変更を対応する外部テーブルに同期することをサポートしました。[#9010](https://github.com/StarRocks/starrocks/pull/9010)

- Broker Loadを介してParquetファイルのARRAYデータをロードすることをサポートしました。[#9131](https://github.com/StarRocks/starrocks/pull/9131)

### バグ修正

以下のバグが修正されました:

- Broker Loadは複数のキーバスファイルでKerberosログインを処理できません。[#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- Supervisorがサービスを再起動できない場合があります。[#9175](https://github.com/StarRocks/starrocks/pull/9175)

- 不正なJoin Reorderの優先順位により、「列を解決できません」というエラーが発生する問題を修正しました。[#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

リリース日: 2022年7月24日

### バグ修正

以下のバグが修正されました:

- リソースグループを削除するとエラーが発生する問題を修正しました。[#8036](https://github.com/StarRocks/starrocks/pull/8036)

- スレッド数が不十分な場合、Thriftサーバーが終了する問題を修正しました。[#7974](https://github.com/StarRocks/starrocks/pull/7974)

- CBOのジョインリオーダーにより、一部のシナリオで結果が返されない問題を修正しました。[#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

リリース日: 2022年6月29日

### 改善点

- UDFはデータベース間で使用できるようになりました。[#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- スキーマ変更などの内部処理の並行制御を最適化しました。これにより、FEメタデータ管理の負荷が軽減されます。また、大量のデータを高い並行性でロードするシナリオでは、ロードジョブが積み上がるか遅くなる可能性が低くなります。[#6838](https://github.com/StarRocks/starrocks/pull/6838)

### バグ修正

以下のバグが修正されました:

- CTASを使用して作成されたレプリカ（`replication_num`）の数が正しくありません。[#7036](https://github.com/StarRocks/starrocks/pull/7036)

- ALTER ROUTINE LOADを実行した後、メタデータが失われる場合があります。[#7068](https://github.com/StarRocks/starrocks/pull/7068)

- ランタイムフィルタがプッシュダウンされない場合があります。[#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- メモリリークが発生する可能性があるパイプラインの問題。[#7295](https://github.com/StarRocks/starrocks/pull/7295)

- Routine Loadジョブが中断された場合にデッドロックが発生する場合があります。[#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 一部のプロファイル統計情報が不正確です。[#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string関数がJSON配列を誤って処理します。[#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

リリース日: 2022年6月2日

### 改善点

- パイプラインエンジンが有効になっている場合のデータロードのパフォーマンスを向上させ、ロングテールのレイテンシを削減しました。[#6641](https://github.com/StarRocks/starrocks/pull/6641)

- BEがクエリを処理するマシンのCPUとメモリ使用状況をFEの監査ログに追加しました。[#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- プライマリキーテーブルにJSONデータ型とJSON関数をサポートしました。[#6544](https://github.com/StarRocks/starrocks/pull/6544)

- ロックの粒度を最適化し、BEレポートリクエストの重複を削除することで、FEの負荷を軽減しました。大量のBEが展開されているシナリオでレポートのパフォーマンスを最適化し、大規模クラスタでRoutine Loadタスクがスタックする問題を解決しました。[#6293](https://github.com/StarRocks/starrocks/pull/6293)

### バグ修正

以下のバグが修正されました:

- `SHOW FULL TABLES FROM DatabaseName` ステートメントで指定されたエスケープ文字をStarRocksが解析するとエラーが発生する問題を修正しました。[#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FEのディスクスペース使用量が急増する問題を修正しました（このバグはBDBJEバージョンをロールバックすることで修正されます）。[#6708](https://github.com/StarRocks/starrocks/pull/6708)

- BEが列をスキャンした後のデータで関連するフィールドを見つけられないため、BEが正常に動作しなくなる問題を修正しました（`enable_docvalue_scan=true`）。[#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

リリース日: 2022年5月22日

### 新機能

- [プレビュー] リソースグループ管理機能がリリースされました。この機能により、StarRocksは同じクラスタ内の異なるテナントからの複雑なクエリと単純なクエリの両方を処理する際に、CPUとメモリリソースを分離して効率的に使用することができます。

- [プレビュー] Javaベースのユーザー定義関数（UDF）フレームワークが実装されました。このフレームワークは、Javaの構文に準拠してコンパイルされたUDFをサポートし、StarRocksの機能を拡張することができます。

- [プレビュー] プライマリキーテーブルは、リアルタイムデータの更新（注文の更新、マルチストリームの結合など）シナリオで、特定の列のみを更新することをサポートします。

- [プレビュー] JSONデータ型とJSON関数をサポートしました。

- Apache Hudiからデータをクエリするために外部テーブルを使用できるようになりました。これにより、StarRocksを使用したデータレイク分析のエクスペリエンスがさらに向上します。詳細については、[外部テーブル](../data_source/External_table.md)を参照してください。

- 次の関数が追加されました:
  - ARRAY関数: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、array_sort、array_distinct、array_join、reverse、array_slice、array_concat、array_difference、arrays_overlap、array_intersect
  - BITMAP関数: bitmap_max、bitmap_min
  - その他の関数: retention、square

### 改善点

- コストベースの最適化（CBO）のパーサとアナライザを再構築し、コード構造を最適化し、CTEを再利用するなどの複雑なクエリのパフォーマンスを向上させるための構文をサポートしました。

- AWS Simple Storage Service（S3）、Alibaba Cloud Object Storage Service（OSS）、Tencent Cloud Object Storage（COS）などのクラウドオブジェクトストレージサービスに格納されたApache Hive™外部テーブルのクエリのパフォーマンスを最適化しました。最適化後、オブジェクトストレージベースのクエリのパフォーマンスはHDFSベースのクエリと同等になります。また、ORCファイルの遅延マテリアライズがサポートされ、小さなファイルのクエリが高速化されます。詳細については、[Apache Hive™外部テーブル](../data_source/External_table.md)を参照してください。

- Apache Hive™からのクエリを使用する場合、StarRocksはHiveメタストアのイベント（データの変更やパーティションの変更など）を消費してキャッシュされたメタデータの増分更新を自動的に実行します。また、Apache Hive™からのDECIMALおよびARRAY型のデータのクエリもサポートします。詳細については、[Apache Hive™外部テーブル](../data_source/External_table.md)を参照してください。

- UNION ALL演算子を最適化し、以前のバージョンよりも2倍から25倍高速化しました。

- 配列をインポートするCSVファイルのために複数の文字を組み合わせて1つの行区切り文字として使用することができるようになりました。

### バグ修正

- プライマリキーテーブルに基づくテーブルにデータをロードしたり変更をコミットしたりするとデッドロックが発生します。[#4998](https://github.com/StarRocks/starrocks/pull/4998)

- Oracle Berkeley DB Java Edition（BDB JE）を含むFEが不安定です。[#4428](https://github.com/StarRocks/starrocks/pull/4428), [#4666](https://github.com/StarRocks/starrocks/pull/4666), [#2](https://github.com/StarRocks/bdb-je/pull/2)

- SUM関数が大量のデータに対して呼び出された場合、算術オーバーフローが発生します。[#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUNDおよびTRUNCATE関数の結果の精度が不十分です。[#4256](https://github.com/StarRocks/starrocks/pull/4256)

- Synthesized Query Lancer（SQLancer）によっていくつかのバグが検出されました。詳細については、[SQLancer関連の問題](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2)を参照してください。

### その他

Flink-connector-starrocksはApache Flink® v1.14をサポートしています。

### アップグレードの注意事項

- StarRocksのバージョンが2.0.4より後の場合、または2.1.6より後のバージョンのStarRocksを使用している場合、アップグレード前にタブレットのクローン機能を無効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");` および `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。アップグレード後、この機能を有効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");` および `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。

- アップグレード前に使用していた以前のバージョンにロールバックする場合は、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを追加し、パラメータを`true`に設定します。このパラメータは、StarRocks v2.2.0で新しいタイプのログが追加されたため必要です。パラメータを追加しない場合、以前のバージョンにロールバックすることはできません。チェックポイントが作成された後、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを`false`に設定することをおすすめします。その後、FEを再起動して、FEを以前の設定に復元します。
