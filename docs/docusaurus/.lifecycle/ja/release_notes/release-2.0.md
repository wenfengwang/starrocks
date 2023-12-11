---
displayed_sidebar: "Japanese"
---

# StarRocksバージョン2.0

## 2.0.9

リリース日: 2022年8月6日

### 不具合の修正

以下のバグが修正されました:

- ブローカーロードジョブの場合、ブローカーが重く負荷がかかっていると、内部のハートビートがタイムアウトし、データの損失が発生する可能性があります。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- ブローカーロードジョブの場合、指定された`COLUMNS FROM PATH AS`パラメータで指定された列が存在しない場合、BEが停止します。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 特定のクエリがリーダーFEに転送され、`/api/query_detail`アクションが、SHOW FRONTENDSなどのSQLステートメントに関する不正確な実行情報を返すことがあります。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 複数のブローカーロードジョブが同じHDFSデータファイルをロードするために作成された場合、1つのジョブで例外が発生すると、他のジョブもデータを正しく読み取れなくなり、結果的に失敗する可能性があります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

リリース日: 2022年7月15日

### 不具合の修正

以下のバグが修正されました:

- リーダーFEノードの切り替えが繰り返し行われると、すべてのロードジョブが停止し、失敗する可能性があります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- MemTableのメモリ使用量の推定が4GBを超えると、データスキュー中に一部のフィールドが大量のメモリリソースを占有するため、BEがクラッシュする可能性があります。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- FEsを再起動した後、大文字と小文字の解析が正しく行われなかったために、マテリアライズドビューのスキーマが変更される可能性があります。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- Routine Loadを使用してKafkaからJSONデータをStarRocksにロードする場合、JSONデータに空行が含まれていると、空行の後のデータが失われる可能性があります。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

リリース日: 2022年6月13日

### 不具合の修正

以下のバグが修正されました:

- コンパクション中のテーブルの列に重複する値の数が0x40000000を超えると、コンパクションが一時停止します。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FEの再起動後、BDB JE v7.3.8のいくつかの問題によって高いI/Oと異常なディスク使用量が発生し、通常状態に戻らないことがあります。 BDB JE v7.3.7にロールバックすると、FEが通常状態に戻ります。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

リリース日: 2022年5月25日

### 不具合の修正

以下のバグが修正されました:

- 一部のグラフィカルユーザーインターフェース（GUI）ツールが、`set_sql_limit`変数を自動で設定します。その結果、SQLステートメントのORDER BY LIMITが無視され、クエリの結果として不正確な行数が返されることがあります。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- コロケーショングループ（CG）に多数のテーブルが含まれ、頻繁にデータがロードされる場合、CGが`stable`状態にならない可能性があります。 この場合、JOINステートメントはColocate Join操作をサポートしません。 StarRocksはデータの読み込み中に少し長く待機するように最適化されました。これにより、データがロードされたタブレットレプリカの整合性を最大限に確保できます。
- 一部のレプリカが重い負荷や高いネットワーク遅延などの理由でロードに失敗すると、これらのレプリカでクローニングがトリガーされる可能性があります。 この場合、デッドロックが発生し、プロセスの負荷は低いままで多数のリクエストがタイムアウトする状況が発生する可能性があります。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- プライマリキーテーブルを使用するテーブルのスキーマを変更した後、そのテーブルにデータをロードする際に「duplicate key xxx」エラーが発生する可能性があります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- データベースに`DROP SCHEMA`ステートメントが実行された場合、データベースが強制的に削除されて元に戻すことができなくなります。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

リリース日: 2022年5月13日

アップグレード推奨: このバージョンで格納されたデータの正確性に関連するいくつかの重大なバグが修正されています。StarRocksクラスタをできるだけ早くアップグレードすることをお勧めします。

### 不具合の修正

以下のバグが修正されました:

- [重大なバグ] BEの障害によりデータが失われる可能性があります。このバグは、特定のバージョンを複数のBEに同時に公開するメカニズムを導入することで修正されています。[#3140](https://github.com/StarRocks/starrocks/issues/3140)
- [重大なバグ] 特定のデータインジェスションフェーズでテーブレットが移行される場合、テーブレットが格納されている元のディスクに引き続きデータが書き込まれます。その結果、データが失われ、クエリが正常に実行されなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)
- [重大なバグ] 複数のDELETE操作を実行した後にクエリを実行すると、クエリの低カーディナリティ列の最適化が行われた場合に、不正確なクエリ結果が取得される可能性があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)
- [重大なバグ] DOUBLE値とVARCHAR値の列を組み合わせるためにJOIN句を含むクエリがある場合、クエリ結果が不正確になることがあります。[#5809](https://github.com/StarRocks/starrocks/pull/5809)
- 特定の状況では、StarRocksクラスタにデータをロードするとき、特定のバージョンのレプリカがFEによって有効とマークされる前に有効になります。この時点で、特定のバージョンのデータが見つからず、エラーが報告される可能性があります。[#5153](https://github.com/StarRocks/starrocks/issues/5153)
- `SPLIT`関数のパラメータが`NULL`に設定されている場合、StarRocksクラスタのBEが停止する可能性があります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)
- Apache Doris 0.13からStarRocks 1.19.xにクラスタがアップグレードされ、一定期間実行された後、StarRocks 2.0.1にさらにアップグレードしようとすると失敗する可能性があります。[#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

リリース日: 2022年4月18日

### 不具合の修正

以下のバグが修正されました:

- 列を削除し、新しいパーティションを追加し、テーブルをクローニングした後、古いテーブルと新しいテーブルの列のユニークIDが同じでないと、BEが停止する可能性があります。なぜなら、システムは共有テーブレットスキーマを使用するためです。[#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocks外部テーブルにデータをロードする際、対象のStarRocksクラスタの構成されたFEがリーダーでない場合、FEが停止する可能性があります。[#4573](https://github.com/StarRocks/starrocks/issues/4573)
- Duplicate Keyテーブルがスキーマ変更を実行し、同時にマテリアライズドビューを作成する場合、クエリ結果が正しくない場合があります。[#4839](https://github.com/StarRocks/starrocks/issues/4839)
- BEの障害によるデータの可能な損失の問題が解決されました（バッチ公開バージョンを使用）。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

リリース日: 2022年3月14日

### 不具合の修正

以下のバグが修正されました:

- BEノードがサスペンドされた状態であるとクエリが失敗することがあります。
- シングルテーブレットテーブルのJOINに適切な実行プランがない場合、クエリが失敗することがあります。[#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FEノードが低カーディナリティの最適化用のグローバルディクショナリを構築するために情報を収集する際にデッドロックの問題が発生する可能性があります。[#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

リリース日: 2022年3月2日

### 改善点

- メモリ使用量が最適化されます。ユーザーは、`label_keep_max_num`パラメータを指定して、一定期間内に保持する最大のロードジョブ数を制御できます。これにより、頻繁なデータロードによるFEのメモリ使用量が高くなることによるフルGCが抑制されます。

### 不具合の修正

以下のバグが修正されました:

- 列デコーダーが例外に遭遇した場合にBEノードが失敗することがあります。
- jsonpathsを指定してJSONデータをロードするコマンドによって、auto __op マッピングが有効にならない場合があります。
- データロード中にソースデータが変更されるため、BEノードが失敗する場合があります。
- マテリアライズド・ビューが作成された後に、一部のSQL文がエラーを報告することがあります。
- SQL句には、低カーディナリティ最適化のためのグローバル辞書をサポートする述語と、サポートしない述語が含まれている場合、クエリが失敗する可能性があります。

## 2.0.1

リリース日: 2022年1月21日

### 改善点

- StarRocksが外部テーブルを使用してHiveデータをクエリする際に、Hiveのimplicit_cast操作が読み取れるようになりました。[#2829](https://github.com/StarRocks/starrocks/pull/2829)
- StarRocks CBOが統計情報を収集して高同時クエリをサポートするために高CPU使用率を修正するために、読み書きロックが使用されます。[#2901](https://github.com/StarRocks/starrocks/pull/2901)
- CBOの統計情報収集およびUNION演算子が最適化されました。

### バグ修正

- レプリカの一貫性のないグローバル辞書によって引き起こされるクエリエラーが修正されました。[#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- データロード中のexec_mem_limitパラメータが効果を発揮しないエラーが修正されました。[#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > exec_mem_limitパラメータは、データのロード中に各BEノードのメモリ制限を指定します。
- プライマリキーテーブルへのデータインポート時に発生するOOMエラーが修正されました。[#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- StarRocksが大きなMySQLテーブルをクエリするために外部テーブルを使用すると、BEノードが応答を停止するエラーが修正されました。[#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 挙動の変更

StarRocksは、外部テーブルを使用してHiveおよびAWS S3ベースの外部テーブルにアクセスすることができます。ただし、S3データにアクセスするために使用されるjarファイルは非常に大きく、StarRocksのバイナリパッケージにはこのjarファイルが含まれていません。このjarファイルを使用する場合は、[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz)からダウンロードできます。

## 2.0.0

リリース日: 2022年1月5日

### 新機能

- 外部テーブル
  - [実験的機能]S3上のHive外部テーブルをサポート
  - 外部テーブルのDecimalV3サポート [#425](https://github.com/StarRocks/starrocks/pull/425)
- 計算のためにストレージレイヤーに複雑な式を押し下げることを実装し、パフォーマンスを向上させます
- Primary Keyが公式リリースされ、Stream Load、Broker Load、Routine Loadをサポートし、さらにFlink-cdcに基づいたMySQLデータの2次同期ツールを提供します

### 改善点

- 算術演算子の最適化
  - 低カーディナリティの辞書のパフォーマンスを最適化しました。[#791](https://github.com/StarRocks/starrocks/pull/791)
  - 単一テーブルのintのスキャンパフォーマンスを最適化しました。[#273](https://github.com/StarRocks/starrocks/issues/273)
  - 高カーディナリティの`count(distinct int)`のパフォーマンスを最適化しました。[#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250)  [#544](https://github.com/StarRocks/starrocks/pull/544)[#570](https://github.com/StarRocks/starrocks/pull/570)
  - `Group by int` / `limit` / `case when` / `not equal`の実装レベルでの最適化
- メモリ管理の最適化
  - メモリの統計および管理フレームワークを再構築し、メモリ使用量を正確にカウントし、OOMを完全に解決しました
  - メタデータのメモリ使用量を最適化
  - 実行スレッドで大量のメモリの開放が長時間にわたってスタックする問題を解決しました
  - プロセスの優雅な終了メカニズムを追加し、メモリリークチェックをサポートしました。[#1093](https://github.com/StarRocks/starrocks/pull/1093)

### バグ修正

- Hive外部テーブルが大量のメタデータを取得する際のタイムアウトの問題が修正されました。
- マテリアライズド・ビューの作成時のエラーメッセージが不明瞭な問題が修正されました。
- ベクトル化エンジンにおけるlikeの実装が修正されました。[#722](https://github.com/StarRocks/starrocks/pull/722)
- `alter table`における述語の解析エラーが修正されました。[#725](https://github.com/StarRocks/starrocks/pull/725)
- `curdate`関数が日付をフォーマットできない問題が修正されました。