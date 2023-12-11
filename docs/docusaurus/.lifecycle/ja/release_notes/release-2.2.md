---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン2.2

リリース日: 2023年4月6日

## 改善点

- bitmap_contains() 関数を最適化し、メモリの消費を削減し、一部のシナリオでのパフォーマンスを向上させました。 [#20616](https://github.com/StarRocks/starrocks/issues/20616)
- Compactフレームワークを最適化し、CPUリソースの消費を削減しました。[#11746](https://github.com/StarRocks/starrocks/issues/11746)

## バグ修正

以下のバグが修正されました:

- Stream LoadのジョブでリクエストされたURLが正しくない場合、責任を持つFEが停止し、HTTPリクエストを処理できなくなります。[#18468](https://github.com/StarRocks/starrocks/issues/18468)
- 責任を持つFEが統計情報を収集する際、異常に多くのメモリを消費することがあり、OOMを引き起こします。[#16331](https://github.com/StarRocks/starrocks/issues/16331)
- クエリ内でメモリの解放が適切に処理されない場合、BEがクラッシュします。[#11395](https://github.com/StarRocks/starrocks/issues/11395)
- TRUNCATE TABLEコマンドを実行した後、NullPointerExceptionが発生し、責任を持つFEが再起動できなくなる場合があります。[#16773](https://github.com/StarRocks/starrocks/issues/16773)

## 2.2.10

リリース日: 2022年12月2日

### 改善点

- Routine Loadジョブのエラーメッセージの返り値を最適化しました。[#12203](https://github.com/StarRocks/starrocks/pull/12203)

- 論理演算子 `&&` をサポートしました。[#11819](https://github.com/StarRocks/starrocks/issues/11819)

- BEがクラッシュした際、クエリが即座にキャンセルされるようになり、期限切れのクエリによるシステムのスタック問題を防ぎます。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

- FE起動スクリプトを最適化しました。FE起動時にJavaバージョンが確認されるようになりました。[#14094](https://github.com/StarRocks/starrocks/pull/14094)

- プライマリキーのテーブルから大量のデータを削除することをサポートしました。[#4772](https://github.com/StarRocks/starrocks/issues/4772)

### バグ修正

以下のバグが修正されました:

- ユーザーが複数のテーブル(UNION)からビューを作成する場合、UNION操作の最左子がNULL定数を使用するとBEがクラッシュします。([#13792](https://github.com/StarRocks/starrocks/pull/13792))

- クエリに使用されるParquetファイルの列の型がHiveテーブルのスキーマと整合していない場合、BEがクラッシュします。[#8848](https://github.com/StarRocks/starrocks/issues/8848)

- クエリに大量のOR演算子が含まれる場合、プランナーが過剰な再帰計算を実行する必要があり、クエリのタイムアウトが発生します。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- サブクエリにLIMIT句が含まれる場合、クエリ結果が正しくありません。[#12466](https://github.com/StarRocks/starrocks/pull/12466)

- SELECT句で二重引用符と単一引用符が混在する場合、CREATE VIEW文が失敗します。[#13102](https://github.com/StarRocks/starrocks/pull/13102)

## 2.2.9

リリース日: 2022年11月15日

### 改善点

- `hive_partition_stats_sample_size` セッション変数を追加し、収集するHiveパーティションの数を制御できるようにしました。過剰なパーティションの数はHiveメタデータの取得にエラーを引き起こします。[#12700](https://github.com/StarRocks/starrocks/pull/12700)

- Elasticsearch外部テーブルがカスタムタイムゾーンをサポートするようにしました。[#12662](https://github.com/StarRocks/starrocks/pull/12662)

### バグ修正

以下のバグが修正されました:

- 外部テーブルのメタデータ同期中にエラーが発生した場合、DECOMMISSION操作がスタックします。[#12369](https://github.com/StarRocks/starrocks/pull/12368)

- 追加された列が削除された場合、Compactionがクラッシュします。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- SHOW CREATE VIEWで作成時に追加されたコメントが表示されません。[#4163](https://github.com/StarRocks/starrocks/issues/4163)

- Java UDFでメモリリークが発生し、OOMが発生する可能性があります。[#12418](https://github.com/StarRocks/starrocks/pull/12418)

- Follower FEに保存されているノードの生存状態が、`heartbeatRetryTimes`に依存しているため、一部のシナリオで正確ではない場合があります。この問題を解決するために、`aliveStatus`プロパティを`HeartbeatResponse`に追加し、ノードの生存状態を示すようにしました。[#12481](https://github.com/StarRocks/starrocks/pull/12481)

### 動作の変更

StarRocksからクエリされることができるHive STRING列の長さを64 KBから1 MBに拡張しました。STRING列が1 MBを超える場合、クエリ中にNULL列として処理されます。[#12986](https://github.com/StarRocks/starrocks/pull/12986)

## 2.2.8

リリース日: 2022年10月17日

### バグ修正

以下のバグが修正されました:

- 初期化段階で式にエラーが発生した場合、BEがクラッシュする可能性があります。[#11395](https://github.com/StarRocks/starrocks/pull/11395)

- 無効なJSONデータがロードされた場合、BEがクラッシュする可能性があります。[#10804](https://github.com/StarRocks/starrocks/issues/10804)

- パイプラインエンジンが有効になっている場合、並列書き込みでエラーが発生します。[#11451](https://github.com/StarRocks/starrocks/issues/11451)

- ORDER BY NULL LIMIT句が使用された場合、BEがクラッシュします。[#11648](https://github.com/StarRocks/starrocks/issues/11648)

- クエリに使用されるParquetファイルの列の型がHiveテーブルのスキーマと整合していない場合、BEがクラッシュします。[#11839](https://github.com/StarRocks/starrocks/issues/11839)

## 2.2.7

リリース日: 2022年9月23日

### バグ修正

以下のバグが修正されました:

- ユーザーがJSONデータをStarRocksにロードする際、データが失われる可能性があります。[#11054](https://github.com/StarRocks/starrocks/issues/11054)

- SHOW FULL TABLESの出力が正しくありません。[#11126](https://github.com/StarRocks/starrocks/issues/11126)

- 以前のバージョンでは、ビューのデータにアクセスするには、ユーザーは基本テーブルとビューの両方にアクセス権を持っている必要がありました。しかし、現在のバージョンでは、ユーザーはビューにのみアクセス権を持つ必要があります。[#11290](https://github.com/StarRocks/starrocks/pull/11290)

- EXISTSまたはINとネストされた複雑なクエリの結果が正しくありません。[#11415](https://github.com/StarRocks/starrocks/pull/11415)

- 対応するHiveテーブルのスキーマが変更された場合、REFRESH EXTERNAL TABLEは失敗します。[#11406](https://github.com/StarRocks/starrocks/pull/11406)

- リーダーでないFEがビットマップインデックス作成操作を再生する際、エラーが発生する可能性があります。[#11261](
## 2.2.6

リリース日: 2022年9月14日

### バグ修正

以下のバグが修正されました:

- LIMITを含むサブクエリがある場合、「order by... limit...offset」の結果が正しくありません。[#9698](https://github.com/StarRocks/starrocks/issues/9698)

- 大容量のテーブルで部分更新が実行される場合、BEがクラッシュする可能性があります。[#9809](https://github.com/StarRocks/starrocks/issues/9809)

- Compactionが、コンパクトするBITMAPデータのサイズが2 GBを超える場合、BEがクラッシュします。[#11159](https://github.com/StarRocks/starrocks/pull/11159)

- like() 関数とregexp() 関数は、パターンの長さが16 KBを超えると機能しません。[#10364](https://github.com/StarRocks/starrocks/issues/10364)

### 動作の変更

出力で配列内のJSON値を表現するために使用される形式が変更されました。エスケープ文字はもはや返されたJSON値には使用されません。[#10790](https://github.com/StarRocks/starrocks/issues/10790)

## 2.2.5

リリース日: 2022年8月18日

### 改善点

- パイプラインエンジンが有効になっている場合のシステムパフォーマンスを改善しました。[#9580](https://github.com/StarRocks/starrocks/pull/9580)

- インデックスメタデータのメモリ統計の精度を向上させました。[#9837](https://github.com/StarRocks/starrocks/pull/9837)

### バグ修正

以下のバグが修正されました:

- Routine Load時にBEがKafkaパーティションオフセットをクエリする中でスタックする可能性があります。[#9937](https://github.com/StarRocks/starrocks/pull/9937)
- 複数のブローカーロードスレッドが同じHDFSファイルをロードしようとすると、エラーが発生します。 [#9507](https://github.com/StarRocks/starrocks/pull/9507)

## 2.2.4

リリース日: 2022年8月3日

### 改善

- Hiveテーブルでのスキーマ変更を対応する外部テーブルに同期するようになりました。 [#9010](https://github.com/StarRocks/starrocks/pull/9010)

- ブローカーロードを介してParquetファイルでのARRAYデータの読み込みをサポートしました。 [#9131](https://github.com/StarRocks/starrocks/pull/9131)

### バグ修正

以下のバグが修正されました。

- ブローカーロードが複数のキータブファイルを使用するKerberosログインを処理できない問題を修正しました。 [#8820](https://github.com/StarRocks/starrocks/pull/8820) [#8837](https://github.com/StarRocks/starrocks/pull/8837)

- **stop_be.sh** が実行された直後にすぐに終了すると、スーパーバイザーがサービスの再起動に失敗する可能性があります。 [#9175](https://github.com/StarRocks/starrocks/pull/9175)

- 不正な結合再配置の優先順位がエラー "Column cannot be resolved" を引き起こす問題を修正しました。 [#9063](https://github.com/StarRocks/starrocks/pull/9063) [#9487](https://github.com/StarRocks/starrocks/pull/9487)

## 2.2.3

リリース日: 2022年7月24日

### バグ修正

以下のバグが修正されました。

- ユーザーがリソースグループを削除するとエラーが発生する問題を修正しました。 [#8036](https://github.com/StarRocks/starrocks/pull/8036)

- スレッド数が十分でない場合にThriftサーバーが終了する問題を修正しました。 [#7974](https://github.com/StarRocks/starrocks/pull/7974)

- 特定のシナリオでCBO内の結合再配置が結果を返さない問題を修正しました。 [#7099](https://github.com/StarRocks/starrocks/pull/7099) [#7831](https://github.com/StarRocks/starrocks/pull/7831) [#6866](https://github.com/StarRocks/starrocks/pull/6866)

## 2.2.2

リリース日: 2022年6月29日

### 改善

- UDFを異なるデータベース間で使用できるようになりました。 [#6865](https://github.com/StarRocks/starrocks/pull/6865) [#7211](https://github.com/StarRocks/starrocks/pull/7211)

- スキーマ変更などの内部処理の並行制御が最適化されました。これにより、FEメタデータ管理への負荷が軽減されます。また、高い並行性で大量のデータをロードするシナリオにおいて、ロードジョブが詰まる可能性や遅延する可能性が軽減されます。 [#6838](https://github.com/StarRocks/starrocks/pull/6838)

### バグ修正

以下のバグが修正されました:

- CTASを使用して作成されたレプリカ（`replication_num`）の数が間違っている問題を修正しました。 [#7036](https://github.com/StarRocks/starrocks/pull/7036)

- ALTER ROUTINE LOADを実行した後にメタデータが失われる可能性がある問題を修正しました。 [#7068](https://github.com/StarRocks/starrocks/pull/7068)

- ランタイムフィルターがプッシュダウンに失敗する問題を修正しました。 [#7206](https://github.com/StarRocks/starrocks/pull/7206) [#7258](https://github.com/StarRocks/starrocks/pull/7258)

- メモリリークを引き起こす可能性があるパイプラインの問題を修正しました。  [#7295](https://github.com/StarRocks/starrocks/pull/7295)

- Routine Loadジョブが中止された際にデッドロックが発生する可能性があります。 [#6849](https://github.com/StarRocks/starrocks/pull/6849)

- 一部のプロファイル統計情報が不正確である問題を修正しました。 [#7074](https://github.com/StarRocks/starrocks/pull/7074) [#6789](https://github.com/StarRocks/starrocks/pull/6789)

- get_json_string関数がJSON配列を誤って処理する問題を修正しました。 [#7671](https://github.com/StarRocks/starrocks/pull/7671)

## 2.2.1

リリース日: 2022年6月2日

### 改善

- コードの一部を再構築し、ロックの粒度を減らすことでデータ読み込みのパフォーマンスを最適化し、長尾のレイテンシを削減しました。 [#6641](https://github.com/StarRocks/starrocks/pull/6641)

- BEが展開されているマシンのCPUおよびメモリ使用状況情報を各クエリのFE監査ログに追加しました。 [#6208](https://github.com/StarRocks/starrocks/pull/6208) [#6209](https://github.com/StarRocks/starrocks/pull/6209)

- PRIMARY KEYテーブルおよびUNIQUE KEYテーブルでのJSONデータ型のサポートを追加しました。 [#6544](https://github.com/StarRocks/starrocks/pull/6544)

- ロックの粒度を減らし、BEレポートリクエストの重複を削減してFEの負荷を軽減し、大規模クラスターでのRoutine Loadタスクが詰まる問題を解決しました。 [#6293](https://github.com/StarRocks/starrocks/pull/6293)

### バグ修正

以下のバグが修正されました:

- `SHOW FULL TABLES FROM DatabaseName` ステートメントで指定されたエスケープ文字を解析する際にエラーが発生する問題を修正しました。 [#6559](https://github.com/StarRocks/starrocks/issues/6559)

- FEディスクスペースの使用率が急激に上昇する問題を修正しました（BDBJEバージョンをロールバックすることで修正）。 [#6708](https://github.com/StarRocks/starrocks/pull/6708)

- カラムスキャンが有効になっている場合（`enable_docvalue_scan=true`）、データから関連するフィールドが見つからないためにBEが故障する問題を修正しました。 [#6600](https://github.com/StarRocks/starrocks/pull/6600)

## 2.2.0

リリース日: 2022年5月22日

### 新機能

- [プレビュー] リソースグループ管理機能がリリースされました。この機能により、StarRocksは同じクラスター内で異なるテナントからの複雑なクエリとシンプルなクエリの両方を効率的に分離して利用することができます。

- [プレビュー] Javaベースのユーザー定義関数（UDF）フレームワークが実装されました。このフレームワークは、Javaの構文に準拠してコンパイルされたUDFの使用をサポートし、StarRocksの機能を拡張します。

- [プレビュー] PRIMARY KEYテーブルがリアルタイムデータ更新シナリオ（注文の更新、複数ストリームの結合など）で特定の列のみを更新することをサポートしました。

- [プレビュー] JSONデータ型およびJSON関数がサポートされています。

- Apache Hudiからデータをクエリするために外部テーブルを使用できるようになりました。これにより、StarRocksを使用したユーザーのデータレイク分析体験がさらに向上します。詳細については、[External tables](../data_source/External_table.md)を参照してください。

- 次の関数が追加されました:
  - ARRAY関数: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md), array_sort, array_distinct, array_join, reverse, array_slice, array_concat, array_difference, arrays_overlap, array_intersect
  - BITMAP関数: bitmap_maxおよびbitmap_min
  - その他の関数: retentionおよびsquare

### 改善

- コストベースオプティマイザ（CBO）のパーサーおよびアナライザーが再構築され、コード構造が最適化され、CTEを再利用するクエリなどの複雑なクエリのパフォーマンスを向上させるために対応しています。

- Apache Hive™の外部テーブルに対するクエリのパフォーマンスが最適化されます。この最適化により、オブジェクトストレージベースのクエリのパフォーマンスがHDFSベースのクエリと同等になります。さらに、ORCファイルの遅延マテリアライゼーションがサポートされ、小さなファイルのクエリが加速されます。詳細については、[Apache Hive™ external table](../data_source/External_table.md)を参照してください。

- Apache Hive™からのクエリが外部テーブルを使用して実行される場合、StarRocksはHiveメタストアイベントの消費によりキャッシュメタデータの増分更新を自動的に行います。また、StarRocksはApache Hive™からDECIMALおよびARRAY型のデータのクエリをサポートしています。詳細については、[Apache Hive™ external table](../data_source/External_table.md)を参照してください。

- UNION ALL演算子は、以前よりも2〜25倍高速化されました。

- 適応型並列性をサポートし、最適化されたプロファイルを提供するパイプラインエンジンがリリースされ、高い並行性のシンプルなクエリのパフォーマンスを向上させました。

- インポート対象のCSVファイルの1行デリミタとして複数の文字を結合して使用できるようになりました。

### バグ修正

- PRIMARY KEYテーブルに基づくテーブルにデータがロードまたはコミットされるとデッドロックが発生する問題を修正しました。 [#4998](https://github.com/StarRocks/starrocks/pull/4998)
- フロントエンド（FE）は、Oracle Berkeley DB Java Edition（BDB JE）を実行するFEを含め、不安定です。[#4428](https://github.com/StarRocks/starrocks/pull/4428)、[#4666](https://github.com/StarRocks/starrocks/pull/4666)、[#2](https://github.com/StarRocks/bdb-je/pull/2)

- SUM 関数によって返される結果は、大量のデータに対して関数が呼び出されると算術オーバーフローが発生します。[#3944](https://github.com/StarRocks/starrocks/pull/3944)

- ROUND および TRUNCATE 関数によって返される結果の精度が不満です。[#4256](https://github.com/StarRocks/starrocks/pull/4256)

- Synthesized Query Lancer（SQLancer）によっていくつかのバグが検出されています。詳細は[SQLancer 関連の問題](https://github.com/StarRocks/starrocks/issues?q=is:issue++label:sqlancer++milestone:2.2)を参照してください。

### その他

Flink-connector-starrocks は Apache Flink® v1.14 をサポートしています。

### アップグレードの注意事項

- StarRocks バージョン 2.0.4 より新しいバージョンまたは StarRocks バージョン 2.1.6 より新しいバージョンを使用している場合、アップグレード前にタブレットクローン機能を無効にできます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "0");` および `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "0");`）。アップグレード後、この機能を有効にすることができます（`ADMIN SET FRONTEND CONFIG ("max_scheduling_tablets" = "2000");` および `ADMIN SET FRONTEND CONFIG ("max_balancing_tablets" = "100");`）。

- アップグレード前に使用していた前のバージョンにロールバックする場合は、各 FE の **fe.conf** ファイルに `ignore_unknown_log_id` パラメータを追加し、パラメータを `true` に設定してください。このパラメータは、StarRocks v2.2.0 で新しいタイプのログが追加されたために必要です。パラメータを追加しないと、以前のバージョンにロールバックできません。チェックポイントが作成された後に、各 FE の **fe.conf** ファイルに `ignore_unknown_log_id` パラメータを `false` に設定することをお勧めします。その後、FE を再起動して、以前の構成に戻します。