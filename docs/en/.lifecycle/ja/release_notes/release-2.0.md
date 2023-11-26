---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.0

## 2.0.9

リリース日: 2022年8月6日

### バグ修正

以下のバグが修正されています:

- Broker Load ジョブの場合、ブローカーが重い負荷をかけられていると、内部のハートビートがタイムアウトしてデータの損失が発生することがあります。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- Broker Load ジョブの場合、`COLUMNS FROM PATH AS` パラメータで指定されたカラムが StarRocks テーブルに存在しない場合、BE が停止します。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 特定のクエリがリーダー FE に転送され、SQL ステートメント（SHOW FRONTENDS など）に関する実行情報が `/api/query_detail` アクションで正しく返されないことがあります。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 同じ HDFS データファイルを読み込むために複数の Broker Load ジョブが作成された場合、1つのジョブで例外が発生すると、他のジョブもデータを正しく読み取ることができず、結果的に失敗することがあります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

リリース日: 2022年7月15日

### バグ修正

以下のバグが修正されています:

- リーダー FE ノードの切り替えを繰り返すと、すべてのロードジョブが停止して失敗することがあります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- MemTable のメモリ使用量の推定が 4GB を超えると BE がクラッシュすることがあります。これは、ロード中のデータの偏りにより、一部のフィールドが大量のメモリリソースを占有するためです。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- FE を再起動した後、マテリアライズドビューのスキーマが大文字と小文字の解析が正しく行われなかったため、変更されました。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- Routine Load を使用して Kafka から JSON データを StarRocks にロードする場合、JSON データに空行があると、空行以降のデータが失われます。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

リリース日: 2022年6月13日

### バグ修正

以下のバグが修正されています:

- コンパクション対象のテーブルの列の重複値の数が 0x40000000 を超えると、コンパクションが一時停止します。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE が再起動した後、BDB JE v7.3.8 にいくつかの問題があるため、高い I/O と異常に増加するディスク使用量に遭遇し、正常に回復しないことがあります。FE は BDB JE v7.3.7 にロールバックすることで正常に復元されます。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

リリース日: 2022年5月25日

### バグ修正

以下のバグが修正されています:

- グラフィカルユーザーインターフェース（GUI）ツールの一部は、`set_sql_limit` 変数を自動的に設定します。その結果、SQL ステートメントの ORDER BY LIMIT が無視され、クエリの結果として正しくない行数が返されます。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- コロケーショングループ（CG）に大量のテーブルが含まれ、テーブルに頻繁にデータがロードされる場合、CG が `stable` 状態にならないことがあります。この場合、JOIN ステートメントは Colocate Join 操作をサポートしません。StarRocks はデータのロード中に少し長く待機するように最適化されています。これにより、データがロードされるタブレットレプリカの整合性を最大化することができます。
- いくつかのレプリカが重い負荷や高いネットワーク遅延などの理由でロードに失敗した場合、これらのレプリカでクローニングがトリガーされることがあります。この場合、デッドロックが発生し、プロセスの負荷は低いが大量のリクエストがタイムアウトする状況が発生する可能性があります。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- プライマリキーテーブルを使用するテーブルのスキーマが変更された後、そのテーブルにデータをロードする際に「重複キー xxx」エラーが発生することがあります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- データベースに対して DROP SCHEMA ステートメントを実行すると、データベースが強制的に削除され、復元できなくなります。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

リリース日: 2022年5月13日

アップグレードの推奨事項: このバージョンでは、データの正確性やデータクエリに関連するいくつかの重要なバグが修正されています。お早めに StarRocks クラスタをアップグレードすることをお勧めします。

### バグ修正

以下のバグが修正されています:

- [重要なバグ] BE の障害によりデータが失われる可能性があります。このバグは、特定のバージョンを複数の BE に同時に公開するメカニズムを導入することで修正されています。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

- [重要なバグ] 特定のデータ取り込みフェーズでテーブルのタブレットが移行されると、データはタブレットが格納されている元のディスクに書き込まれ続けます。その結果、データが失われ、クエリが正常に実行されなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)

- [重要なバグ] 複数の DELETE 操作を実行した後にクエリを実行すると、低基数列の最適化がクエリに対して実行されるため、正しいクエリ結果が得られない場合があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)

- [重要なバグ] DOUBLE 値の列と VARCHAR 値の列を組み合わせるために JOIN 句を含むクエリの結果が正しくない場合があります。[#5809](https://github.com/StarRocks/starrocks/pull/5809)

- 特定の状況下では、StarRocks クラスタにデータをロードする際に特定のバージョンのレプリカが FEs によって有効とマークされる前に有効になることがあります。この時点で、特定のバージョンのデータをクエリすると、StarRocks はデータを見つけることができず、エラーが報告されます。[#5153](https://github.com/StarRocks/starrocks/issues/5153)

- `SPLIT` 関数のパラメータが `NULL` に設定されている場合、StarRocks クラスタの BE が停止することがあります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)

- クラスタが Apache Doris 0.13 から StarRocks 1.19.x にアップグレードされ、一定期間実行された後、StarRocks 2.0.1 へのさらなるアップグレードが失敗することがあります。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

リリース日: 2022年4月18日

### バグ修正

以下のバグが修正されています:

- 列の削除、新しいパーティションの追加、タブレットのクローニング後、古いタブレットと新しいタブレットの列の一意の ID が同じでない場合、共有タブレットスキーマを使用するシステムが動作を停止する可能性があります。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks 外部テーブルにデータをロードする際、対象の StarRocks クラスタの構成された FE がリーダーではない場合、FE が停止します。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- データが重複キーテーブルにロードされ、同時にスキーマ変更とマテリアライズドビューの作成が行われる場合、クエリ結果が正しくない場合があります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- BE 障害によるデータの損失の可能性の問題（バッチパブリッシュバージョンを使用して解決）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

リリース日: 2022年3月14日

### バグ修正

以下のバグが修正されています:

- BE ノードが一時停止している場合、クエリが失敗することがあります。
- シングルテーブルテーブル結合に適切な実行計画がない場合、クエリが失敗することがあります。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FE ノードがグローバル辞書を構築するための情報を収集する際にデッドロックの問題が発生することがあります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

リリース日: 2022年3月2日

### 改善点

- メモリ使用量が最適化されました。ユーザーは `label_keep_max_num` パラメータを指定して、一定期間内に保持するロードジョブの最大数を制御することができます。これにより、データの頻繁なロードによる FE のメモリ使用量の高さによるフル GC を防ぐことができます。

### バグ修正

以下のバグが修正されています:

- 列デコーダが例外に遭遇した場合、BE ノードが失敗する問題が修正されました。
- JSON データをロードするためのコマンドで `jsonpaths` が指定されている場合、自動的な `__op` マッピングが効果を発揮しない問題が修正されました。
- データロード中にソースデータが変更されるために BE ノードが失敗する問題が修正されました。
- マテリアライズドビューが作成された後に一部の SQL ステートメントがエラーを報告する問題が修正されました。
- SQL 句にグローバル辞書をサポートする述語とサポートしない述語が含まれる場合、クエリが失敗する可能性があります。

## 2.0.1

リリース日: 2022年1月21日

### 改善点

- StarRocks が外部テーブルを使用して Hive データをクエリする際に、Hive の implicit_cast 操作を読み取ることができるようになりました。[#2829](https://github.com/StarRocks/starrocks/pull/2829)
- StarRocks CBO が高並行性クエリをサポートするために統計情報を収集する際に高い CPU 使用率が発生する問題を修正するために、読み取り/書き込みロックを使用するようになりました。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- CBO の統計情報収集と UNION 演算子が最適化されました。

### バグ修正

- レプリカのグローバル辞書が一貫していないためにクエリエラーが発生する問題が修正されました。[#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- データロード中の `exec_mem_limit` パラメータが効果を発揮しない問題が修正されました。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > `exec_mem_limit` パラメータは、データロード中の各 BE ノードのメモリ制限を指定します。
- プライマリキーテーブルにデータをインポートする際に発生する OOM エラーが修正されました。[#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- StarRocks が外部テーブルを使用して大きな MySQL テーブルをクエリする際に BE ノードが応答しなくなるエラーが修正されました。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 動作の変更

StarRocks は外部テーブルを使用して Hive および AWS S3 ベースの外部テーブルにアクセスすることができます。ただし、S3 データにアクセスするために使用される jar ファイルは非常に大きいため、StarRocks のバイナリパッケージにはこの jar ファイルが含まれていません。この jar ファイルを使用する場合は、[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz) からダウンロードすることができます。

## 2.0.0

リリース日: 2022年1月5日

### 新機能

- 外部テーブル
  - [実験的な機能] S3 上の Hive 外部テーブルのサポート
  - DecimalV3 が外部テーブルに対応 [#425](https://github.com/StarRocks/starrocks/pull/425)
- ストレージレイヤーにプッシュダウンされる複雑な式の実装により、パフォーマンスの向上が得られます
- Primary Key が正式にリリースされ、Stream Load、Broker Load、Routine Load をサポートし、また Flink-cdc をベースにした MySQL データの二次同期ツールを提供します。

### 改善点

- 算術演算子の最適化
  - 低基数辞書のパフォーマンスを最適化 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - シングルテーブルの int のスキャンパフォーマンスを最適化 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 高基数の `count(distinct int)` のパフォーマンスを最適化 [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250)  [#544](https://github.com/StarRocks/starrocks/pull/544)[#570](https://github.com/StarRocks/starrocks/pull/570)
  - `Group by int` / `limit` / `case when` / `not equal` の実装レベルでの最適化
- メモリ管理の最適化
  - メモリの統計と制御フレームワークをリファクタリングして、メモリ使用量を正確にカウントし、OOM を完全に解決します
  - メタデータのメモリ使用量を最適化
  - 実行スレッドで大量のメモリ解放が長時間スタックする問題を解決
  - プロセスの優雅な終了メカニズムを追加し、メモリリークのチェックをサポート [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### バグ修正

- Hive 外部テーブルが大量のメタデータを取得するためにタイムアウトする問題が修正されました。
- マテリアライズドビューの作成時のエラーメッセージがわかりにくい問題が修正されました。
- ベクトル化エンジンの like の実装の問題が修正されました。[#722](https://github.com/StarRocks/starrocks/pull/722)
- `alter table` で述語の解析が行われるエラーが修正されました。[#725](https://github.com/StarRocks/starrocks/pull/725)
- 外部テーブルを使用して大きな MySQL テーブルをクエリする際に BE ノードが停止する問題が修正されました。[#2881](https://github.com/StarRocks/starrocks/pull/2881)
