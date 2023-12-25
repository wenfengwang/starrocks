---
displayed_sidebar: Chinese
---

# StarRocks バージョン 2.0

## 2.0.9

リリース日：2022年8月6日

### 問題修正

以下の問題を修正しました：

- Broker Loadを使用してデータをインポートする際、Brokerプロセスの負荷が大きい場合、内部のハートビート処理に問題が発生し、データのインポートが失敗する可能性があります。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- Broker Loadを使用してデータをインポートする際、`COLUMNS FROM PATH AS` パラメータで指定された列がStarRocksのテーブルに存在しない場合、BEが停止する問題がありました。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 一部のクエリがLeader FEノードに転送されると、`/api/query_detail` インターフェースを通じて取得されるSQLステートメントの実行情報が正しくない可能性があります。例えば、SHOW FRONTENDSステートメントなどです。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 複数のBroker Loadジョブを同時に提出し、同じHDFSファイルからデータをインポートする際、一つのジョブに異常が発生すると、他のジョブもデータを正常に読み取れず、最終的に失敗する可能性があります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

リリース日：2022年7月15日

### 問題修正

以下の問題を修正しました：

- Leader FEノードを繰り返し切り替えると、すべてのインポートジョブが中断し、インポートが行えなくなる可能性があります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- インポートされたデータに偏りがある場合、一部の列が多くのメモリを消費し、MemTableのメモリ推定が4GBを超えることで、BEが停止する可能性があります。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- 物化ビューが大文字と小文字を処理する際に問題があり、FEを再起動するとそのスキーマが変更される問題がありました。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- Routine Loadを使用してKafkaからJSONデータをインポートする際、JSONデータに空行が含まれていると、その後のデータが失われる可能性があります。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

リリース日：2022年6月13日

### 問題修正

以下の問題を修正しました：

- テーブルのコンパクション(Compaction)を行う際、ある列の任意の値が0x40000000回以上繰り返されると、コンパクションが停止する問題がありました。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- BDB JE v7.3.8のバージョンに問題があり、FEの起動後にディスクI/Oが高くなり、ディスク使用率が異常に増加し、回復の兆しがない問題がありました。BDB JE v7.3.7に戻すとFEが正常に戻ります。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

リリース日：2022年5月25日

### 問題修正

以下の問題を修正しました：

- 一部のグラフィカルインターフェースツールが自動的に`set_sql_limit`変数を設定し、SQLステートメントのORDER BY LIMITが無視され、結果として返されるデータ行数が正しくない問題がありました。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- Colocation Groupに含まれるテーブルが多く、インポート頻度が高い場合、そのColocation Groupが`stable`状態を維持できず、JOINステートメントでColocate Joinを使用できない問題がありました。インポート時に少し待つように最適化し、インポートされたTabletのレプリカの完全性をできるだけ保証します。
- 負荷が高い、ネットワークの遅延などの理由で少数のレプリカがインポートに失敗すると、システムがレプリカのクローンをトリガーします。この状況では、デッドロックが発生する可能性があり、プロセスの負荷が非常に低く、多くのリクエストがタイムアウトする現象が発生する可能性があります。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- 主キーモデルのテーブルがテーブル構造の変更を経た後、データのインポート時に"duplicate key xxx"エラーが発生する可能性があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- DROP SCHEMAステートメントを実行すると、データベースが強制的に削除され、削除されたデータベースは復元できない問題がありました。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

リリース日：2022年5月13日

アップグレード推奨：今回はデータストレージやデータクエリの正確性に関連する重要なバグをいくつか修正しましたので、タイムリーにアップグレードすることをお勧めします。

### 問題修正

以下の問題を修正しました：

- 【Critical Bug】バッチでpublish versionを改善することにより、BEがダウンした場合にデータが失われる問題を解決しました。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- 【Critical Bug】データ書き込みの特定の段階で、Tabletが移動して完了した場合、データは元のTabletに対応するディスクに引き続き書き込まれ、データが失われ、クエリエラーが発生する問題がありました。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- 【Critical Bug】複数のDELETE操作を行った後、システムが内部で低カーディナリティ最適化を使用している場合、クエリ結果が間違っている可能性があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- 【Critical Bug】JOINクエリの2つのフィールドのタイプがそれぞれDOUBLEとVARCHARの場合、JOINクエリの結果が間違っている可能性があります。[#5809](https://github.com/StarRocks/starrocks/pull/5809)
- データインポートの特定の状況で、いくつかのレプリカのバージョンがまだ有効になっていないにもかかわらず、FEによって有効とマークされ、クエリ時に対応するバージョンのデータが見つからないエラーが発生する問題がありました。[#5153](https://github.com/StarRocks/starrocks/issues/5153)
- `SPLIT`関数に`NULL`パラメータを使用すると、BEが停止する問題がありました。[#4092](https://github.com/StarRocks/starrocks/issues/4092)
- Apache Doris 0.13からStarRocks 1.19.xにアップグレードし、しばらく運用した後、StarRocks 2.0.1にアップグレードすると、アップグレードに失敗する可能性があります。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

リリース日： 2022年4月18日

### 問題修正

以下の問題を修正しました：


- 列の削除、パーティションの追加、およびTabletのクローニング後、新旧のTabletの列Unique IDが一致しない可能性があり、システムが共有Tablet Schemaを使用しているため、BEが停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocksの外部テーブルにデータをインポートする際、ターゲットとなるStarRocksクラスタのFEがLeaderでない場合、FEが停止する可能性があります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- 明細モデルのテーブルでテーブル構造の変更とマテリアライズドビューの作成を同時に実行すると、データクエリに誤りが生じる可能性があります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- バッチpublish versionの改善により、BEがダウンしたことによるデータ損失の問題を解決しました。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

リリース日： 2022年3月14日

### 問題修正

- BEのフリーズが原因でクエリエラーが発生する問題を修正しました。
- 単一のtabletに対する集約操作を行う際に適切な実行計画が得られず、クエリが失敗する問題を修正しました。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FEが低カーディナリティのグローバル辞書を構築するための情報を収集する際にデッドロックが発生する可能性がある問題を修正しました。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

リリース日： 2022年3月2日

### 機能最適化

- FEのメモリ使用量を最適化しました。パラメータ`label_keep_max_num`を設定することで、一定期間内に保持されるインポートタスクの最大数を制御し、高頻度のジョブインポート時にFEのメモリ使用量が過多になりFull GCが発生するのを防ぎます。

### 問題修正

- ColumnDecoderの異常によりBEノードが応答しなくなる問題を修正しました。
- JSON形式のデータをインポートする際にjsonpathsを設定した後、__opフィールドを自動的に認識できない問題を修正しました。
- Broker Loadによるデータインポート中にソースデータが変更され、BEノードが応答しなくなる問題を修正しました。
- マテリアライズドビューを作成した後、一部のSQLステートメントでエラーが発生する問題を修正しました。
- クエリステートメントに低カーディナリティのグローバル辞書がサポートしていない述語が同時に存在すると、クエリが失敗する問題を修正しました。

## 2.0.1

リリース日： 2022年1月21日

### 機能最適化

- StarRocksがHive外部テーブルを読み取る際のHive外部テーブルの暗黙的なデータ変換機能を最適化しました。[#2829](https://github.com/StarRocks/starrocks/pull/2829)
- 高並行クエリシナリオで、StarRocksのCBOオプティマイザが統計情報を収集する際のロック競合を最適化しました。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- CBOの統計情報処理、UNIONオペレータなどを最適化しました。

### 問題修正

- レプリカのグローバル辞書の不一致がクエリに影響を与える問題を修正しました。[#2700](https://github.com/StarRocks/starrocks/pull/2700)[#2765](https://github.com/StarRocks/starrocks/pull/2765)
- StarRocksにデータをインポートする前に設定したパラメータ`exec_mem_limit`が効かない問題を修正しました。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > パラメータ`exec_mem_limit`は、データインポート時に各BEノードの計算層で使用するメモリの上限を指定するために使用されます。
- StarRocksのプライマリキーモデルにデータをインポートする際にOOMが発生する問題を修正しました。[#2743](https://github.com/StarRocks/starrocks/pull/2743)[#2777](https://github.com/StarRocks/starrocks/pull/2777)
- StarRocksが大量のMySQL外部テーブルをクエリする際にクエリが停止する問題を修正しました。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 挙動変更

- StarRocksはHive外部テーブルを使用して、Hive外部テーブル上に作成されたAmazon S3外部テーブルにアクセスすることをサポートしています。Amazon S3外部テーブルにアクセスするためのjarファイルが大きいため、現在のところStarRocksのバイナリ製品パッケージにはこのjarファイルは含まれていません。必要な場合は、[Hive_s3_lib](https://releases.mirrorship.cn/resources/hive_s3_jar.tar.gz)をクリックしてダウンロードしてください。

## 2.0.0

リリース日：2022年1月5日

### 新機能

- 外部テーブル
  - [実験機能]S3上のHive外部テーブル機能をサポート [参考文書](../data_source/External_table.md#deprecated-hive-外部表)
  - DecimalV3が外部テーブルクエリをサポート [#425](https://github.com/StarRocks/starrocks/pull/425)
- ストレージ層での複雑な式のプッシュダウン計算を実装し、パフォーマンスを向上させました
- Broker LoadがHuawei OBSをサポート [#1182](https://github.com/StarRocks/starrocks/pull/1182)
- 国家暗号アルゴリズムsm3をサポート
- ARM系国産CPUに対応：鯤鵬アーキテクチャで検証済み
- プライマリキーモデル（Primary Key）が正式にリリースされ、Stream Load、Broker Load、Routine Loadをサポートし、Flink-cdcをベースにしたMySQLデータの秒単位の同期ツールも提供しています。[参考文書](../table_design/table_types/primary_key_table.md)

### 機能最適化

- オペレータのパフォーマンスを最適化
  - 低カーディナリティ辞書のパフォーマンス最適化[#791](https://github.com/StarRocks/starrocks/pull/791)
  - 単一テーブルのint scanのパフォーマンス最適化 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 高カーディナリティ下での`count(distinct int)`のパフォーマンス最適化 [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250) [#544](https://github.com/StarRocks/starrocks/pull/544) [#570](https://github.com/StarRocks/starrocks/pull/570)
  - 実行層の最適化と改善 `Group by 2 int` / `limit` / `case when` / `not equal`
- メモリ管理の最適化
  - メモリ統計/制御フレームワークを再構築し、メモリ使用量を正確に統計し、OOMを完全に解決
  - メタデータのメモリ使用を最適化
  - 大容量メモリの解放が実行スレッドを長時間ブロックする問題を解決
  - プロセスの優雅な終了メカニズムをサポートし、メモリリークのチェックをサポート[#1093](https://github.com/StarRocks/starrocks/pull/1093)

### 問題修正

- Hive外部テーブルからの大量のメタデータ取得がタイムアウトする問題を修正しました
- マテリアライズドビューの作成時のエラーメッセージが不明確な問題を修正しました
- 修正ベクトル化エンジンにおける`like`の実装 [#722](https://github.com/StarRocks/starrocks/pull/722)
- `alter table`における述語`is`の解析エラーを修正 [#725](https://github.com/StarRocks/starrocks/pull/725)
- `curdate`関数が日付をフォーマットできない問題を修正
