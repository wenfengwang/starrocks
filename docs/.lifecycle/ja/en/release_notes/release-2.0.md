---
displayed_sidebar: English
---

# StarRocks バージョン 2.0

## 2.0.9

リリース日: 2022年8月6日

### バグ修正

以下のバグが修正されました：

- Broker Load ジョブの場合、ブローカーが過負荷のとき、内部ハートビートがタイムアウトしてデータが失われる可能性があります。[#8282](https://github.com/StarRocks/starrocks/issues/8282)
- Broker Load ジョブで、宛先の StarRocks テーブルに `COLUMNS FROM PATH AS` パラメータで指定された列が存在しない場合、BE が停止します。[#5346](https://github.com/StarRocks/starrocks/issues/5346)
- 一部のクエリが Leader FE に転送され、`/api/query_detail` アクションが SHOW FRONTENDS などの SQL ステートメントに関する誤った実行情報を返すことがあります。[#9185](https://github.com/StarRocks/starrocks/issues/9185)
- 複数の Broker Load ジョブが同じ HDFS データファイルをロードするために作成された場合、1つのジョブが例外に遭遇すると、他のジョブもデータを適切に読み取れず、結果的に失敗する可能性があります。[#9506](https://github.com/StarRocks/starrocks/issues/9506)

## 2.0.8

リリース日: 2022年7月15日

### バグ修正

以下のバグが修正されました：

- Leader FE ノードを繰り返し切り替えると、全てのロードジョブがハングアップし、失敗する可能性があります。[#7350](https://github.com/StarRocks/starrocks/issues/7350)
- MemTable のメモリ使用量の見積もりが 4GB を超えた場合、データの偏りにより一部のフィールドが大量のメモリリソースを占有する可能性があるため、BE がクラッシュすることがあります。[#7161](https://github.com/StarRocks/starrocks/issues/7161)
- FE を再起動した後、大文字と小文字の解析が誤っていたため、マテリアライズドビューのスキーマが変更されてしまうことがあります。[#7362](https://github.com/StarRocks/starrocks/issues/7362)
- Routine Load を使用して Kafka から StarRocks に JSON データをロードする際、JSON データに空行が含まれていると、空行以降のデータが失われることがあります。[#8534](https://github.com/StarRocks/starrocks/issues/8534)

## 2.0.7

リリース日: 2022年6月13日

### バグ修正

以下のバグが修正されました：

- 圧縮中のテーブルのカラム内の重複値の数が 0x40000000 を超えると、圧縮が中断されます。[#6513](https://github.com/StarRocks/starrocks/issues/6513)
- FE を再起動した後、BDB JE v7.3.8 のいくつかの問題により I/O が高くなり、ディスク使用量が異常に増加し、正常に戻る兆候が見られないことがあります。FE は BDB JE v7.3.7 にロールバックすることで正常な状態に戻ります。[#6634](https://github.com/StarRocks/starrocks/issues/6634)

## 2.0.6

リリース日: 2022年5月25日

### バグ修正

以下のバグが修正されました：

- 一部の GUI ツールは `set_sql_limit` 変数を自動的に設定します。その結果、SQL ステートメントの ORDER BY LIMIT が無視され、クエリで不正確な行数が返されることがあります。[#5966](https://github.com/StarRocks/starrocks/issues/5966)
- Colocation Group (CG) に多数のテーブルが含まれ、テーブルに頻繁にデータがロードされる場合、CG が `stable` 状態を維持できないことがあります。この場合、JOIN ステートメントは Colocate Join 操作をサポートしません。データロード中にもう少し待機するように StarRocks が最適化されており、データがロードされるタブレットレプリカの整合性を最大限に高めることができます。
- 負荷が高い、ネットワーク遅延が長いなどの理由でいくつかのレプリカのロードに失敗すると、これらのレプリカのクローニングがトリガーされます。この場合、プロセスの負荷は低いが多数のリクエストがタイムアウトするというデッドロックが発生する可能性があります。[#5646](https://github.com/StarRocks/starrocks/issues/5646) [#6290](https://github.com/StarRocks/starrocks/issues/6290)
- Primary Key テーブルを使用するテーブルのスキーマが変更された後、そのテーブルにデータをロードする際に「重複キー xxx」というエラーが発生することがあります。[#5878](https://github.com/StarRocks/starrocks/issues/5878)
- データベースに対して `DROP SCHEMA` ステートメントを実行すると、データベースが強制的に削除され、復元できなくなります。[#6201](https://github.com/StarRocks/starrocks/issues/6201)

## 2.0.5

リリース日: 2022年5月13日

アップグレード推奨: このバージョンでは、保存されたデータやデータクエリの正確性に関連するいくつかの重大なバグが修正されています。できるだけ早く StarRocks クラスタをアップグレードすることを推奨します。

### バグ修正

以下のバグが修正されました：

- [重大なバグ] BE の障害によりデータが失われる可能性があります。このバグは、特定のバージョンを一度に複数の BE に公開するメカニズムを導入することで修正されました。[#3140](https://github.com/StarRocks/starrocks/issues/3140)

- [重大なバグ] タブレットが特定のデータ取り込みフェーズで移行されると、データはタブレットが保存されている元のディスクに引き続き書き込まれます。その結果、データが失われ、クエリが正しく実行できなくなります。[#5160](https://github.com/StarRocks/starrocks/issues/5160)

- [重大なバグ] 複数の DELETE 操作を実行した後にクエリを実行すると、低カーディナリティ列のクエリ最適化が行われると、誤ったクエリ結果を得る可能性があります。[#5712](https://github.com/StarRocks/starrocks/issues/5712)

- [重大なバグ] DOUBLE 値の列と VARCHAR 値の列を結合する JOIN 句を含むクエリを実行すると、クエリ結果が誤っている可能性があります。[#5809](https://github.com/StarRocks/starrocks/pull/5809)

- 特定の状況下で StarRocks クラスタにデータをロードすると、特定のバージョンのレプリカが有効になる前に FE によって有効とマークされることがあります。この時、特定のバージョンのデータをクエリすると、StarRocks はデータを見つけられず、エラーを報告することがあります。[#5153](https://github.com/StarRocks/starrocks/issues/5153)

- `SPLIT` 関数のパラメータが `NULL` に設定されている場合、StarRocks クラスタの BE が停止することがあります。[#4092](https://github.com/StarRocks/starrocks/issues/4092)  

- Apache Doris 0.13 から StarRocks 1.19.x にクラスターをアップグレードし、一定期間稼働させた後、StarRocks 2.0.1 へのさらなるアップグレードが失敗する可能性があります。 [#5309](https://github.com/StarRocks/starrocks/issues/5309)

## 2.0.4

リリース日: 2022年4月18日

### バグ修正

以下のバグが修正されました：

- 列を削除し、新しいパーティションを追加し、タブレットをクローンした後、古いタブレットと新しいタブレットの列のユニークIDが一致しない場合があり、システムが共有タブレットスキーマを使用しているため、BEが停止する可能性があります。 [#4514](https://github.com/StarRocks/starrocks/issues/4514)
- StarRocksの外部テーブルにデータをロードする際、ターゲットStarRocksクラスタの設定済みFEがリーダーではない場合、FEが停止することがあります。 [#4573](https://github.com/StarRocks/starrocks/issues/4573)
- Duplicate Keyテーブルでスキーマ変更を行い、同時にマテリアライズドビューを作成すると、クエリ結果が不正確になる可能性があります。 [#4839](https://github.com/StarRocks/starrocks/issues/4839)
- BE障害によるデータ損失の可能性の問題（バッチパブリッシュバージョンを使用して解決）。 [#3140](https://github.com/StarRocks/starrocks/issues/3140)

## 2.0.3

リリース日: 2022年3月14日

### バグ修正

以下のバグが修正されました：

- BEノードがサスペンド状態にあるときにクエリが失敗する問題。
- 単一タブレットテーブルの結合に適切な実行プランがない場合、クエリが失敗する問題。 [#3854](https://github.com/StarRocks/starrocks/issues/3854)
- FEノードが低カーディナリティ最適化のためのグローバルディクショナリを構築する情報を収集する際に、デッドロックが発生する可能性がある問題。 [#3839](https://github.com/StarRocks/starrocks/issues/3839)

## 2.0.2

リリース日: 2022年3月2日

### 改善点

- メモリ使用量が最適化されました。ユーザーは `label_keep_max_num` パラメータを指定して、一定期間内に保持するロードジョブの最大数を制御できます。これにより、データロードが頻繁に行われる際のFEの高メモリ使用によるフルGCを防ぐことができます。

### バグ修正

以下のバグが修正されました：

- 列デコーダが例外に遭遇した際にBEノードが失敗する問題。
- JSONデータのロードに使用されるコマンドで `jsonpaths` が指定されている場合、自動的な `__op` マッピングが機能しない問題。
- Broker Loadを使用してデータロード中にソースデータが変更されたためにBEノードが失敗する問題。
- マテリアライズドビューが作成された後に一部のSQL文がエラーを報告する問題。
- SQL句に低カーディナリティ最適化のグローバルディクショナリをサポートする述語とサポートしない述語が含まれている場合、クエリが失敗する問題。

## 2.0.1

リリース日: 2022年1月21日

### 改善点

- Hiveの `implicit_cast` 操作が読み取れるようになりました。これはStarRocksが外部テーブルを使用してHiveデータをクエリする際に有効です。 [#2829](https://github.com/StarRocks/starrocks/pull/2829)
- StarRocks CBOが統計を収集し、高並行性クエリをサポートする際のCPU使用率を低減するために、読み書きロックが使用されます。 [#2901](https://github.com/StarRocks/starrocks/pull/2901)
- CBOの統計収集とUNIONオペレータが最適化されました。

### バグ修正

- レプリカ間のグローバルディクショナリの不整合によって引き起こされるクエリエラーが修正されました。 [#2700](https://github.com/StarRocks/starrocks/pull/2700) [#2765](https://github.com/StarRocks/starrocks/pull/2765)
- データロード中に `exec_mem_limit` パラメータが機能しない問題が修正されました。 [#2693](https://github.com/StarRocks/starrocks/pull/2693)
  > この `exec_mem_limit` パラメータは、データロード中の各BEノードのメモリ制限を指定します。
- プライマリキーテーブルへのデータインポート時に発生するOOMエラーが修正されました。 [#2743](https://github.com/StarRocks/starrocks/pull/2743) [#2777](https://github.com/StarRocks/starrocks/pull/2777)
- StarRocksが外部テーブルを使用して大規模なMySQLテーブルをクエリする際にBEノードが応答を停止する問題が修正されました。 [#2881](https://github.com/StarRocks/starrocks/pull/2881)

### 動作変更

StarRocksは外部テーブルを使用してHiveおよびそのAWS S3ベースの外部テーブルにアクセスできます。ただし、S3データにアクセスするためのjarファイルが大きすぎるため、StarRocksのバイナリパッケージにはこのjarファイルが含まれていません。このjarファイルを使用したい場合は、[Hive_s3_lib](https://releases.starrocks.io/resources/hive_s3_jar.tar.gz)からダウンロードしてください。

## 2.0.0

リリース日: 2022年1月5日

### 新機能

- 外部テーブル
  - [実験的機能] S3上のHive外部テーブルのサポート
  - 外部テーブルのDecimalV3サポート [#425](https://github.com/StarRocks/starrocks/pull/425)
- ストレージ層にプッシュダウンして計算を行う複雑な式を実装し、パフォーマンス向上を実現
- Primary Keyが正式リリースされ、Stream Load、Broker Load、Routine Loadをサポートし、Flink-cdcに基づくMySQLデータの秒単位の同期ツールも提供

### 改善点

- 算術演算子の最適化
  - 低カーディナリティの辞書におけるパフォーマンスを最適化 [#791](https://github.com/StarRocks/starrocks/pull/791)
  - 単一テーブルでのintスキャンパフォーマンスを最適化 [#273](https://github.com/StarRocks/starrocks/issues/273)
  - 高カーディナリティにおける `count(distinct int)` のパフォーマンスを最適化 [#139](https://github.com/StarRocks/starrocks/pull/139) [#250](https://github.com/StarRocks/starrocks/pull/250) [#544](https://github.com/StarRocks/starrocks/pull/544) [#570](https://github.com/StarRocks/starrocks/pull/570)
  - `Group by int` / `limit` / `case when` / `not equal` の実装レベルでの最適化
- メモリ管理の最適化
  - メモリ統計と制御フレームワークをリファクタリングし、メモリ使用量を正確にカウントし、OOM問題を完全に解決
  - メタデータのメモリ使用量を最適化
  - 実行スレッドで大量のメモリ解放が長時間停滞する問題を解決
  - プロセスの正常終了メカニズムを追加し、メモリリークチェックをサポート [#1093](https://github.com/StarRocks/starrocks/pull/1093)

### バグ修正

- Hive外部テーブルが大量のメタデータを取得する際にタイムアウトする問題を修正しました。
- マテリアライズド・ビュー作成時のエラーメッセージが不明瞭な問題を修正しました。
- ベクトル化エンジンでの`like`の実装を修正しました[#722](https://github.com/StarRocks/starrocks/pull/722)
- `alter table`での述語の解析エラーを修正しました[#725](https://github.com/StarRocks/starrocks/pull/725)
- `curdate`関数が日付をフォーマットできない問題を修正しました
