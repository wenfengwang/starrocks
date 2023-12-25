---
displayed_sidebar: English
---

# StarRocks バージョン 2.2

リリース日: 2023年4月6日

## 改善点

- `bitmap_contains()` 関数を最適化して、メモリ消費量を削減し、一部のシナリオでパフォーマンスを向上させました。 [#20616](https://github.com/StarRocks/starrocks/issues/20616)
- Compactionフレームワークを最適化して、CPUリソース消費を削減しました。 [#11746](https://github.com/StarRocks/starrocks/issues/11746)

## バグ修正

以下のバグが修正されました:

- Stream Loadジョブで要求されたURLが正しくない場合、該当するFEがハングし、HTTPリクエストを処理できなくなります。 [#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 統計情報を収集する際に、該当するFEが異常に大量のメモリを消費し、OOMを引き起こす可能性があります。 [#16331](https://github.com/StarRocks/starrocks/issues/16331)
- 一部のクエリでメモリ解放が適切に処理されない場合、BEがクラッシュすることがあります。 [#11395](https://github.com/StarRocks/starrocks/issues/11395)
- `TRUNCATE TABLE` コマンド実行後にNullPointerExceptionが発生し、該当するFEが再起動できなくなることがあります。 [#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

リリース日: 2022年12月2日

### 改善点

- Routine Loadジョブのエラーメッセージを最適化しました。 [#12203](https://github.com/StarRocks/starrocks/pull/12203)

- 論理演算子 `&&` をサポートしました。 [#11819](https://github.com/StarRocks/starrocks/issues/11819)

- BEがクラッシュした際、クエリは即座にキャンセルされ、期限切れのクエリによるシステムの停止問題を防ぎます。 [#12954](https://github.com/StarRocks/starrocks/pull/12954)

- FEの起動スクリプトを最適化しました。FE起動時にJavaバージョンがチェックされるようになりました。 [#14094](https://github.com/StarRocks/starrocks/pull/14094)

- Primary Keyテーブルから大量のデータを削除する機能をサポートしました。 [#4772](https://github.com/StarRocks/starrocks/issues/4772)

### バグ修正

以下のバグが修正されました:

- ユーザーが複数のテーブル (UNION) からビューを作成する際、UNION演算の最左子がNULL定数を使用している場合、BEがクラッシュすることがあります。 ([#13792](https://github.com/StarRocks/starrocks/pull/13792))

- クエリ対象のParquetファイルの列の型がHiveテーブルスキーマと矛盾している場合、BEがクラッシュすることがあります。 [#8848](https://github.com/StarRocks/starrocks/issues/8848)

- クエリに多数のOR演算子が含まれる場合、プランナーは過剰な再帰計算を行う必要があり、クエリがタイムアウトすることがあります。 [#12788](https://github.com/StarRocks/starrocks/pull/12788)

- サブクエリにLIMIT句が含まれている場合、クエリ結果が不正確になることがあります。 [#12466](https://github.com/StarRocks/starrocks/pull/12466)

- SELECT句内で二重引用符と単一引用符が混在している場合、`CREATE VIEW` ステートメントが失敗することがあります。 [#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

リリース日: 2022年11月15日

### 改善点

- Hiveパーティションの統計情報を収集する際のサンプルサイズを制御するセッション変数 `hive_partition_stats_sample_size` を追加しました。パーティションの数が多すぎると、Hiveメタデータの取得時にエラーが発生する可能性があります。 [#12700](https://github.com/StarRocks/starrocks/pull/12700)

- Elasticsearch外部テーブルがカスタムタイムゾーンをサポートしました。 [#12662](https://github.com/StarRocks/starrocks/pull/12662)

### バグ修正

以下のバグが修正されました:

- 外部テーブルのメタデータ同期中にエラーが発生した場合、DECOMMISSION操作が停止することがあります。 [#12369](https://github.com/StarRocks/starrocks/pull/12368)

- 新しく追加された列が削除された場合、Compactionがクラッシュすることがあります。 [#12907](https://github.com/StarRocks/starrocks/pull/12907)

- `SHOW CREATE VIEW` では、ビュー作成時に追加されたコメントが表示されないことがあります。 [#4163](https://github.com/StarRocks/starrocks/issues/4163)

- Java UDFのメモリリークがOOMを引き起こす可能性があります。 [#12418](https://github.com/StarRocks/starrocks/pull/12418)

- Follower FEに格納されているノードの生存状態が、`heartbeatRetryTimes` に依存しているため、一部のシナリオでは正確ではない可能性があります。この問題を解決するために、ノードの生存状態を示すプロパティ `aliveStatus` が `HeartbeatResponse` に追加されました。 [#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 挙動の変更

StarRocksでクエリ可能なHive STRING列の長さを64KBから1MBに拡張しました。STRING列が1MBを超える場合、クエリ中にnull列として処理されます。 [#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

リリース日: 2022年10月17日

### バグ修正

以下のバグが修正されました:

- 初期化段階でエラーに遭遇した式が原因でBEがクラッシュする可能性があります。 [#11395](https://github.com/StarRocks/starrocks/pull/11395)

- 無効なJSONデータがロードされた場合、BEがクラッシュすることがあります。 [#10804](https://github.com/StarRocks/starrocks/issues/10804)

- パイプラインエンジンが有効な場合、並列書き込みでエラーが発生することがあります。 [#11451](https://github.com/StarRocks/starrocks/issues/11451)

- `ORDER BY NULL LIMIT` 句を使用するとBEがクラッシュすることがあります。 [#11648](https://github.com/StarRocks/starrocks/issues/11648)

- クエリ対象のParquetファイルの列の型がHiveテーブルスキーマと一致しない場合、BEがクラッシュすることがあります。 [#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

リリース日: 2022年9月23日

### バグ修正

以下のバグが修正されました:

- ユーザーがJSONデータをStarRocksにロードする際、データが失われる可能性があります。 [#11054](https://github.com/StarRocks/starrocks/issues/11054)

- `SHOW FULL TABLES` の出力が正しくないことがあります。 [#11126](https://github.com/StarRocks/starrocks/issues/11126)

- 以前のバージョンでは、ビュー内のデータにアクセスするためには、ユーザーはベーステーブルとビューの両方に対する権限が必要でした。現在のバージョンでは、ビューに対する権限のみが必要です。 [#11290](https://github.com/StarRocks/starrocks/pull/11290)

- EXISTS または IN でネストされた複雑なクエリの結果が正しくありません。 [#11415](https://github.com/StarRocks/starrocks/pull/11415)

- 対応する Hive テーブルのスキーマが変更された場合、REFRESH EXTERNAL TABLE が失敗します。 [#11406](https://github.com/StarRocks/starrocks/pull/11406)

- 非リーダー FE がビットマップインデックス作成操作をリプレイする際にエラーが発生することがあります。 [#11261](

## 2.2.6

リリース日: 2022年9月14日

### バグ修正

以下のバグが修正されました:

- サブクエリに LIMIT が含まれている場合の `order by... limit...offset` の結果が正しくありません。 [#9698](https://github.com/StarRocks/starrocks/issues/9698)

- 大量のデータを持つテーブルに対して部分更新を実行すると BE がクラッシュすることがあります。 [#9809](https://github.com/StarRocks/starrocks/issues/9809)

- BITMAP データの圧縮サイズが 2 GB を超えると、コンパクションにより BE がクラッシュすることがあります。 [#11159](https://github.com/StarRocks/starrocks/pull/11159)

- パターンの長さが 16 KB を超える場合、like() 関数と regexp() 関数が機能しなくなります。 [#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 動作変更

配列の出力における JSON 値を表す形式が変更されました。返される JSON 値にエスケープ文字は使用されなくなりました。 [#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

リリース日: 2022年8月18日

### 改善点

- パイプラインエンジンが有効化された場合のシステムパフォーマンスが向上しました。 [#9580](https://github.com/StarRocks/starrocks/pull/9580)

- インデックスメタデータのメモリ統計の精度が向上しました。 [#9837](https://github.com/StarRocks/starrocks/pull/9837)

### バグ修正

以下のバグが修正されました:

- BE が Routine Load 中に Kafka パーティションオフセット (`get_partition_offset`) のクエリでスタックする可能性があります。 [#9937](https://github.com/StarRocks/starrocks/pull/9937)

- 複数の Broker Load スレッドが同じ HDFS ファイルをロードしようとするとエラーが発生します。 [#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

リリース日: 2022年8月3日

### 改善点

- Hive テーブルのスキーマ変更を対応する外部テーブルに同期する機能をサポートしました。 [#9010](https://github.com/StarRocks/starrocks/pull/9010)

- Broker Load を使用して Parquet ファイルから ARRAY データをロードする機能をサポートしました。 [#9131](https://github.com/StarRocks/starrocks/pull/9131)

### バグ修正

以下のバグが修正されました:

- Broker Load が複数の keytab ファイルを持つ Kerberos ログインを処理できません。 [#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- **stop_be.sh** を実行直後に終了すると、Supervisor がサービスの再起動に失敗することがあります。 [#9175](https://github.com/StarRocks/starrocks/pull/9175)

- Join Reorder の優先順位が不適切で "Column cannot be resolved" のエラーが発生することがあります。 [#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

リリース日: 2022年7月24日

### バグ修正

以下のバグが修正されました:

- ユーザーがリソースグループを削除する際にエラーが発生することがあります。 [#8036](https://github.com/StarRocks/starrocks/pull/8036)

- スレッド数が不足している場合に Thrift サーバーが終了することがあります。 [#7974](https://github.com/StarRocks/starrocks/pull/7974)

- 一部のシナリオで CBO の join reorder が結果を返さないことがあります。 [#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

リリース日: 2022年6月29日

### 改善点

- UDF を複数のデータベース間で使用できるようにしました。 [#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- スキーマ変更などの内部処理の同時実行制御を最適化して、FE のメタデータ管理の負荷を軽減しました。これにより、大量のデータを高い並行性でロードするシナリオでのロードジョブの蓄積や遅延の可能性が低減されます。 [#6838](https://github.com/StarRocks/starrocks/pull/6838)

### バグ修正

以下のバグが修正されました:

- CTAS を使用して作成されたレプリカの数 (`replication_num`) が正しくありません。 [#7036](https://github.com/StarRocks/starrocks/pull/7036)

- ALTER ROUTINE LOAD を実行した後にメタデータが失われることがあります。 [#7068](https://github.com/StarRocks/starrocks/pull/7068)

- ランタイムフィルターがプッシュダウンされないことがあります。 [#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- メモリリークを引き起こす可能性のあるパイプラインの問題。 [#7295](https://github.com/StarRocks/starrocks/pull/7295)

- Routine Load ジョブが中止された際にデッドロックが発生することがあります。 [#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 一部のプロファイル統計情報が不正確です。 [#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string 関数が JSON 配列を正しく処理しないことがあります。 [#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

リリース日: 2022年6月2日

### 改善点

- ホットスポットコードの一部を再構築し、ロックの粒度を下げることでデータロードのパフォーマンスを最適化し、ロングテールレイテンシーを短縮しました。 [#6641](https://github.com/StarRocks/starrocks/pull/6641)

- 各クエリにおける BE の配置されているマシンの CPU とメモリ使用情報を FE 監査ログに追加しました。 [#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- 主キーテーブルとユニークキーテーブルでサポートされるJSONデータ型。 [#6544](https://github.com/StarRocks/starrocks/pull/6544)

- ロックの粒度を小さくし、BEレポートリクエストの重複を排除することでFEの負荷を軽減しました。多数のBEがデプロイされている場合のレポート性能を最適化し、大規模クラスタでのルーチンロードタスクの停滞問題を解決しました。 [#6293](https://github.com/StarRocks/starrocks/pull/6293)

### バグ修正

以下のバグが修正されました：

- StarRocksが`SHOW FULL TABLES FROM DatabaseName`ステートメントで指定されたエスケープ文字を解析する際にエラーが発生します。 [#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FEのディスクスペース使用量が急激に増加します（このバグはBDBJEバージョンをロールバックすることで修正されます）。 [#6708](https://github.com/StarRocks/starrocks/pull/6708)

- `enable_docvalue_scan=true`が有効になっていると、BEがカラムスキャン後に返されるデータ内で関連フィールドを見つけられずに障害が発生します。 [#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

リリース日：2022年5月22日

### 新機能

- [プレビュー] リソースグループ管理機能がリリースされました。この機能により、StarRocksは同じクラスタ内の異なるテナントからの複雑なクエリと単純なクエリを処理する際に、CPUとメモリリソースを分離して効率的に使用することができます。

- [プレビュー] Javaベースのユーザー定義関数（UDF）フレームワークが実装されました。このフレームワークは、Javaの構文に準拠してコンパイルされたUDFをサポートし、StarRocksの機能を拡張します。

- [プレビュー] 主キーテーブルは、注文更新やマルチストリームジョインなどのリアルタイムデータ更新シナリオにおいて、データが主キーテーブルにロードされる際に特定の列のみを更新することをサポートします。

- [プレビュー] JSONデータ型とJSON関数がサポートされています。

- 外部テーブルを使用してApache Hudiからデータをクエリすることができます。これにより、StarRocksを使用したデータレイク分析のユーザーエクスペリエンスがさらに向上します。詳細は[外部テーブル](../data_source/External_table.md)を参照してください。

- 追加された関数は以下の通りです：
  - ARRAY関数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、array_sort、array_distinct、array_join、reverse、array_slice、array_concat、array_difference、arrays_overlap、array_intersect
  - BITMAP関数：bitmap_max、bitmap_min
  - その他の関数：retention、square

### 改善点

- コストベースのオプティマイザ（CBO）のパーサーとアナライザーが再構築され、コード構造が最適化され、共通テーブル式（CTE）を使用したINSERTなどの構文がサポートされました。これらの改善により、CTEの再利用を伴うクエリなど、複雑なクエリのパフォーマンスが向上します。

- AWS Simple Storage Service（S3）、Alibaba Cloud Object Storage Service（OSS）、Tencent Cloud Object Storage（COS）などのクラウドオブジェクトストレージサービスに保存されているApache Hive™外部テーブルに対するクエリのパフォーマンスが最適化されました。最適化後、オブジェクトストレージベースのクエリのパフォーマンスはHDFSベースのクエリと同等です。さらに、ORCファイルの遅延マテリアライゼーションがサポートされ、小ファイルに対するクエリが加速されます。詳細は[Apache Hive™外部テーブル](../data_source/External_table.md)を参照してください。

- Apache Hive™からのクエリが外部テーブルを使用して実行される場合、StarRocksはデータ変更やパーティション変更などのHiveメタストアイベントを消費することで、キャッシュされたメタデータのインクリメンタル更新を自動的に実行します。StarRocksはApache Hive™のDECIMAL型とARRAY型のデータに対するクエリもサポートします。詳細は[Apache Hive™外部テーブル](../data_source/External_table.md)を参照してください。

- UNION ALL演算子が最適化され、以前より2倍から25倍高速に実行されるようになりました。

- 適応的な並列性をサポートし、最適化されたプロファイルを提供するパイプラインエンジンがリリースされ、高い同時実行シナリオでの単純なクエリのパフォーマンスが向上しました。

- 複数の文字を組み合わせて、インポートされるCSVファイルの単一行区切り文字として使用できます。

### バグ修正

- データがロードされたり、主キーテーブルに基づくテーブルに変更がコミットされたりするとデッドロックが発生することがあります。 [#4998](https://github.com/StarRocks/starrocks/pull/4998)

- Oracle Berkeley DB Java Edition（BDB JE）を実行するFEを含むフロントエンド（FE）が不安定です。 [#4428](https://github.com/StarRocks/starrocks/pull/4428)、[#4666](https://github.com/StarRocks/starrocks/pull/4666)、[#2](https://github.com/StarRocks/bdb-je/pull/2)

- 大量のデータに対してSUM関数が呼び出された場合、関数によって返される結果が算術オーバーフローを起こすことがあります。 [#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUND関数とTRUNCATE関数によって返される結果の精度が不十分です。 [#4256](https://github.com/StarRocks/starrocks/pull/4256)

- いくつかのバグがSynthesized Query Lancer（SQLancer）によって検出されました。詳細は[SQLancer関連の問題](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2)を参照してください。

### その他

Flink-connector-starrocksはApache Flink® v1.14をサポートしています。

### アップグレードノート

- 2.0.4以降のStarRocksバージョンまたは2.1.6以降のStarRocksバージョン2.1.xを使用している場合、アップグレード前にタブレットクローン機能を無効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");`および`ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。アップグレード後、この機能を有効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");`および`ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。

- アップグレード前に使用されていた以前のバージョンにロールバックするには、各FEの**fe.conf**ファイルに`ignore_unknown_log_id`パラメータを追加し、そのパラメータを`true`に設定します。このパラメータは必要です。なぜならStarRocks v2.2.0では新しいタイプのログが追加されているからです。このパラメータを追加しない場合、以前のバージョンにロールバックすることはできません。チェックポイントが作成された後、各FEの**fe.conf**ファイルで`ignore_unknown_log_id`パラメータを`false`に設定することを推奨します。その後、FEを再起動して、以前の設定に復元します。
