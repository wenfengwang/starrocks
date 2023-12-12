---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.0

## 2.0.9

リリース日: 2022年8月6日

### バグ修正

以下のバグが修正されています:

- Broker Load ジョブの場合、Broker が過負荷になると内部ハートビートがタイムアウトし、データが失われる可能性があります。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- Broker Load ジョブの場合、`COLUMNS FROM PATH AS` パラメータで指定された列を含まない StarRocks テーブルの場合、BE が実行を停止します。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 一部のクエリがリーダ FE に転送され、`/api/query_detail` アクションが SHOW FRONTENDS などの SQL ステートメントに関する正確な実行情報を返さないことがあります。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 同じ HDFS データファイルを読み込むために複数の Broker Load ジョブが作成された場合、1つのジョブで例外が発生すると、他のジョブもデータを正しく読み込めない可能性があり、その結果、失敗します。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

リリース日: 2022年7月15日

### バグ修正

以下のバグが修正されています:

- リーダ FE ノードの切り替えを繰り返すと、すべてのロードジョブがハングおよび失敗することがあります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- メモリ使用量の推定が MemTable の 4GB を超えると BE がクラッシュする可能性があります。データスキューが発生した場合、一部のフィールドが大量のメモリリソースを占有するためです。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- フィルタリングされたビューのスキーマが大文字と小文字の解析エラーにより変更されたため、FE の再起動後にマテリアライズド ビューのスキーマが変更されます。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- Routine Load を使用して JSON データを Kafka から StarRocks にロードする場合、JSON データに空行があると、空行以降のデータが失われます。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

リリース日: 2022年6月13日

### バグ修正

以下のバグが修正されています:

- コンパクション中のテーブルの列の重複値の数が 0x40000000 を超えると、コンパクションが一時停止します。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE の再起動後、BDB JE v7.3.8 におけるいくつかの問題により I/O が高くなり、異常にディスク使用量が増加し、正常に復元する兆候が示されません。BDB JE v7.3.7 にロールバックすることで、FE は正常に復元されます。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

リリース日: 2022年5月25日

### バグ修正

以下のバグが修正されています:

- 一部のグラフィカルユーザインターフェース（GUI）ツールが `set_sql_limit` 変数を自動的に構成します。その結果、SQL ステートメント ORDER BY LIMIT が無視され、クエリに対して不正な行数が返されることがあります。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- Co-location グループ（CG）に大量のテーブルが含まれ、頻繁にデータがロードされる場合、CG は`stable` 状態になることができません。この場合、JOIN ステートメントでは Co-locate Join 操作がサポートされません。StarRocks はデータロード中に少し長く待機するように最適化されています。これにより、データがロードされるタブレットレプリカの整合性が最大化されます。
- レプリカのいくつかが重い負荷や高いネットワーク遅延などの理由でロードに失敗すると、これらのレプリカでクローニングがトリガーされます。この場合、デッドロックが発生し、プロセスの負荷は低いままで、大量のリクエストがタイムアウトする状況が発生する可能性があります。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- Primary Key テーブルを使用するテーブルのスキーマが変更された後、そのテーブルにデータをロードする際に "duplicate key xxx" エラーが発生する可能性があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- データベースに `DROP SCHEMA` ステートメントが実行されると、データベースが強制的に削除され、復元できなくなります。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

リリース日: 2022年5月13日

アップグレード推奨: このバージョンでは、ストアされたデータの正確性に関連するいくつかの重大なバグが修正されています。StarRocks クラスタをできるだけ早くアップグレードすることをお勧めします。

### バグ修正

以下のバグが修正されています:

- [重大なバグ] BE の障害によりデータが失われることがあります。特定のバージョンを複数の BE に同時に公開するメカニズムを導入することで、このバグが修正されました。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- [重大なバグ] 特定のデータ取込みフェーズでテーブレットが移行される場合、元のディスクに引き続きデータが書き込まれ続けます。その結果、データが失われ、クエリを正常に実行できなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- [重大なバグ] 複数の DELETE 操作を実行した後にクエリを実行すると、クエリの低カーディナリティ列に対する最適化が行われる場合、不正確なクエリ結果を取得する可能性があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- [重大なバグ] DOUBLE 値の列と VARCHAR 値の列を結合するために JOIN 句を含むクエリを実行した場合、クエリ結果が不正確になる可能性があります。[#5809](https://github.com/StarRocks/starrocks/pull/5809)
- 特定の状況で、StarRocks クラスタにデータをロードする際、特定のバージョンのレプリカが FEs によって有効とマークされるが、その後レプリカが有効にならない可能性があります。この場合、特定のバージョンのデータが見つからずエラーが報告されます。[#5153](https://github.com/StarRocks/starrocks/issues/5153)
- `SPLIT` 関数のパラメータが `NULL` に設定されている場合、StarRocks クラスタの BE が停止する可能性があります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)  
- Apache Doris 0.13 から StarRocks 1.19.x にクラスタをアップグレードし、一定期間実行した後、StarRocks 2.0.1 にさらにアップグレードしようとすると失敗する可能性があります。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

リリース日: 2022年4月18日

### バグ修正

以下のバグが修正されています:

- 列を削除し、新しいパーティションを追加し、テーブレットをクローニングした後、新旧テーブレットの列のユニーク ID が異なる場合、共有されたテーブットスキーマを使用するため、BE が動作を停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks 外部テーブルにデータをロードする際、対象の StarRocks クラスタの構成された FE がリーダでない場合、FE が停止する可能性があります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- Duplicate Key テーブルがスキーマ変更と同時にマテリアライズド ビューを作成する場合、クエリの結果が正しくないことがあります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- BE の障害によるデータの可能性がある問題（Batch publish version を使用することで解決）[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

リリース日: 2022年3月14日

### バグ修正

以下のバグが修正されています:

- BE ノードが一時停止状態になっているときにクエリが失敗することがあります。
- 一つのテーブレットのテーブル結合に適切な実行計画がない場合にクエリが失敗することがあります。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FE ノードが低カーディナリティ最適化のためにグローバル辞書を構築するための情報を収集する際にデッドロックの問題が発生する可能性があります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

リリース日: 2022年3月2日

### 改善

- メモリ使用量が最適化されました。ユーザーは `label_keep_max_num` パラメータを指定して、一定期間内に保持するロードジョブの最大数を制御できます。これにより、頻繁なデータロードによる FE の高いメモリ使用量によるフル GC を防ぐことができます。

### バグ修正

以下のバグが修正されています:

- 列デコーダが例外に遭遇したときに BE ノードが失敗する可能性があります。
- Auto __op mapping does not take effect when jsonpaths is specified in the command used for loading JSON data.
- BE nodes fail because the source data changes during data loading using Broker Load.
- Some SQL statements report errors after materialized views are created.
- Query may fail if an SQL clause contains a predicate that supports global dictionary for low-cardinality optimization and a predicate that does not.

## 2.0.1

リリース日：2022年1月21日

### 改善

- Hiveのimplicit_cast操作は、StarRocksが外部テーブルを使用してHiveデータをクエリする際に読み取ることができます。[#2829](https://github.com/StarRocks/starrocks/pull/2829)
- StarRocks CBOが統計情報を収集して高同時実行クエリをサポートするために高CPU使用率を修正するために、読み書きロックが使用されます。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- CBO統計情報の収集とUNION演算子が最適化されました。

### バグ修正

- レプリカの一貫性のないグローバル辞書によって引き起こされるクエリエラーが修正されました。[#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- データの読み込み中に`exec_mem_limit`パラメータが効果を発揮しないエラーが修正されました。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > `exec_mem_limit`パラメータは、データの読み込み中に各BEノードのメモリ制限を指定します。
- プライマリキー テーブルにデータをインポートする際に発生するOOMエラーが修正されました。[#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- StarRocksが外部テーブルを使用して大規模なMySQLテーブルをクエリする際に、BEノードが応答を停止するエラーが修正されました。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 動作の変更

StarRocksは外部テーブルを使用してHiveおよびそのAWS S3ベースの外部テーブルにアクセスできます。ただし、S3データにアクセスするために使用されるjarファイルが大きすぎるため、StarRocksのバイナリパッケージにはこのjarファイルが含まれていません。このjarファイルを使用する場合は、[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz)からダウンロードできます。

## 2.0.0

リリース日：2022年1月5日

### 新機能

- 外部テーブル
  - [実験的な機能]S3上のHive外部テーブルをサポート
  - DecimalV3は外部テーブルをサポート [#425](https://github.com/StarRocks/starrocks/pull/425)
- 複雑な式をストレージ層に押し込んで計算を実行し、パフォーマンスを向上させる
- Primary Keyが公式にリリースされ、Stream Load、Broker Load、Routine Loadをサポートし、Flink-cdcベースのMySQLデータの二次同期ツールも提供します

### 改善

- 算術演算子の最適化
  - 低基数辞書のパフォーマンスを最適化 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - 単一テーブルのintのスキャンパフォーマンスを最適化 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 高基数の`count(distinct int)`のパフォーマンスを最適化  [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250)  [#544](https://github.com/StarRocks/starrocks/pull/544)[#570](https://github.com/StarRocks/starrocks/pull/570)
  - `Group by int` / `limit` / `case when` / `not equal`の実装レベルの最適化
- メモリ管理の最適化
  - メモリ統計および制御フレームワークをリファクタリングして、メモリ使用量を正確にカウントし、OOMを完全に解決します
  - メタデータのメモリ使用量を最適化
  - 大規模メモリの解放が実行スレッドで長時間スタックする問題を解決
  - プロセスの優雅な終了メカニズムを追加し、メモリリークのチェックをサポート [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### バグ修正

- Hive外部テーブルが大量のメタデータを取得する際のタイムアウトの問題が修正されました。
- マテリアライズドビューの作成時のエラーメッセージが不明瞭な問題が修正されました。
- ベクトル化エンジンでlikeの実装が修正されました [#722](https://github.com/StarRocks/starrocks/pull/722)
- `alter table`で述語の解析エラーが修正されました[#725](https://github.com/StarRocks/starrocks/pull/725)
- `curdate`関数が日付をフォーマットできないエラーが修正されました