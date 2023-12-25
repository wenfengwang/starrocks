---
displayed_sidebar: English
---

# StarRocks バージョン 2.4

## 2.4.5

リリース日: 2023年4月21日

### 改善点

- メタデータのアップグレード時にエラーを引き起こす可能性があるため、List Partition 構文を禁止しました。 [#15401](https://github.com/StarRocks/starrocks/pull/15401)
- 実体化ビューでの BITMAP、HLL、PERCENTILE 型をサポート。 [#15731](https://github.com/StarRocks/starrocks/pull/15731)
- `storage_medium` の推論を最適化。BEがSSDとHDDを両方使用している場合、`storage_cooldown_time` が指定されていれば `storage_medium` を `SSD` に、そうでなければ `HDD` に設定します。 [#18649](https://github.com/StarRocks/starrocks/pull/18649)
- スレッドダンプの精度を最適化。 [#16748](https://github.com/StarRocks/starrocks/pull/16748)
- メタデータの圧縮をロード前にトリガーすることでロード効率を最適化。 [#19347](https://github.com/StarRocks/starrocks/pull/19347)
- Stream Load プランナーのタイムアウトを最適化。 [#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- 値列からの統計情報収集を禁止することで、Unique key テーブルのパフォーマンスを最適化。 [#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

以下のバグが修正されました:

- CREATE TABLE でサポートされていないデータ型を使用した場合、NPEが返される問題。 [#20999](https://github.com/StarRocks/starrocks/issues/20999)
- ショートサーキットを使用した Broadcast Join のクエリで誤った結果が返される問題。 [#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 誤ったデータ切り捨てロジックによるディスク占有問題。 [#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoader プラグインがインストールも削除もできない問題。 [#20468](https://github.com/StarRocks/starrocks/issues/20468)
- タブレットがスケジュールされている際に例外が発生すると、同じバッチの他のタブレットがスケジュールされない問題。 [#20681](https://github.com/StarRocks/starrocks/pull/20681)
- サポートされていない SQL 関数を使用して同期実体化ビューを作成する際に Unknown Error が返される問題。 [#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 複数の COUNT DISTINCT 計算が誤って書き換えられる問題。 [#19714](https://github.com/StarRocks/starrocks/pull/19714)
- 圧縮中のタブレットに対するクエリで誤った結果が返される問題。 [#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 集約を伴うクエリで誤った結果が返される問題。 [#19725](https://github.com/StarRocks/starrocks/issues/19725)
- NULL の Parquet データを NOT NULL 列にロードする際にエラーメッセージが返されない問題。 [#19885](https://github.com/StarRocks/starrocks/pull/19885)
- リソースグループのコンカレンシー制限に連続して達した場合、クエリの並行性メトリックが遅く減少する問題。 [#19363](https://github.com/StarRocks/starrocks/pull/19363)
- `InsertOverwriteJob` の状態変更ログを再生する際に FE が起動しない問題。 [#19061](https://github.com/StarRocks/starrocks/issues/19061)
- Primary Key テーブルのデッドロック問題。 [#18488](https://github.com/StarRocks/starrocks/pull/18488)
- Colocation テーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");` のようなステートメントを使用してレプリカのステータスを手動で `bad` に設定できます。BEの数がレプリカの数以下である場合、破損したレプリカは修復できない問題。 [#17876](https://github.com/StarRocks/starrocks/issues/17876)
- ARRAY 関連関数によって引き起こされる問題。 [#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

リリース日: 2023年2月22日

### 改善点

- ロードの高速キャンセルをサポート。 [#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- コンパクションフレームワークの CPU 使用率を最適化。 [#11747](https://github.com/StarRocks/starrocks/pull/11747)
- バージョンが欠落しているタブレットに対する累積コンパクションをサポート。 [#17030](https://github.com/StarRocks/starrocks/pull/17030)

### バグ修正

以下のバグが修正されました:

- 無効な DATE 値が指定された場合に過剰な動的パーティションが作成される問題。 [#17966](https://github.com/StarRocks/starrocks/pull/17966)
- デフォルトの HTTPS ポートを使用して Elasticsearch 外部テーブルに接続できない問題。 [#13726](https://github.com/StarRocks/starrocks/pull/13726)
- トランザクションがタイムアウトした後に BE が Stream Load トランザクションをキャンセルできない問題。 [#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 単一の BE 上でのローカルシャッフル集約から誤ったクエリ結果が返される問題。 [#17845](https://github.com/StarRocks/starrocks/pull/17845)
- 「wait_for_version version: failed: apply stopped」というエラーメッセージでクエリが失敗する問題。 [#17848](https://github.com/StarRocks/starrocks/pull/17850)
- OLAP スキャンバケット式が正しくクリアされないために誤ったクエリ結果が返される問題。 [#17666](https://github.com/StarRocks/starrocks/pull/17666)
- Colocate Group 内の動的パーティションテーブルのバケット数を変更できず、エラーメッセージが返される問題。 [#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 非リーダー FE ノードに接続して `USE <catalog_name>.<database_name>` SQL ステートメントを送信すると、非リーダー FE ノードが `<catalog_name>` を除外してリーダー FE ノードに転送し、結果としてリーダー FE ノードが `default_catalog` を使用し、指定されたデータベースを見つけられない問題。 [#17302](https://github.com/StarRocks/starrocks/pull/17302)
- 書き換え前の dictmapping チェックの不正確なロジックによる問題。 [#17405](https://github.com/StarRocks/starrocks/pull/17405)
- FE が不定期に BE にハートビートを送信し、ハートビート接続がタイムアウトすると、FE が BE を使用不可と判断し、BE 上でトランザクションが失敗する問題。 [#16386](https://github.com/StarRocks/starrocks/pull/16386)
- フォロワー FE で新しくクローンされたタブレットに対する `get_applied_rowsets` クエリが失敗する問題。 [#17192](https://github.com/StarRocks/starrocks/pull/17192)
- フォロワー FE で `SET variable = default` を実行した際に NPE が発生する問題。 [#17549](https://github.com/StarRocks/starrocks/pull/17549)
- 射影内の式が辞書によって書き換えられない問題。 [#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 週単位で動的パーティションを作成するための不正確なロジックによる問題。 [#17163](https://github.com/StarRocks/starrocks/pull/17163)
- ローカルシャッフルから誤ったクエリ結果が返される問題。 [#17130](https://github.com/StarRocks/starrocks/pull/17130)
- インクリメンタルクローンが失敗する問題。 [#16930](https://github.com/StarRocks/starrocks/pull/16930)
- CBO が 2 つの演算子が等価であるかどうかを比較するために不正確なロジックを使用する場合がある問題。 [#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- JuiceFS スキーマが正しくチェックおよび解決されないため、JuiceFS にアクセスできない問題。 [#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

リリース日: 2023年1月19日

### 改善点

- StarRocksは、NPEを防ぐために、Analyze実行時に対応するデータベースとテーブルが存在するかどうかをチェックします。 [#14467](https://github.com/StarRocks/starrocks/pull/14467)
- サポートされていないデータ型の列は、外部テーブルのクエリではマテリアライズされません。 [#13305](https://github.com/StarRocks/starrocks/pull/13305)
- FE起動スクリプト **start_fe.sh** のJavaバージョンチェックを追加しました。 [#14333](https://github.com/StarRocks/starrocks/pull/14333)

### バグ修正

以下のバグが修正されました:

- タイムアウトが設定されていない場合、Stream Loadが失敗することがあります。 [#16241](https://github.com/StarRocks/starrocks/pull/16241)
- メモリ使用率が高いときにbRPC Sendがクラッシュすることがあります。 [#16046](https://github.com/StarRocks/starrocks/issues/16046)
- StarRocksは、旧バージョンのStarRocksインスタンスから外部テーブルへのデータロードに失敗することがあります。 [#16130](https://github.com/StarRocks/starrocks/pull/16130)
- マテリアライズドビューのRefresh失敗がメモリリークを引き起こす可能性があります。 [#16041](https://github.com/StarRocks/starrocks/pull/16041)
- スキーマ変更がPublishステージでハングすることがあります。 [#14148](https://github.com/StarRocks/starrocks/issues/14148)
- マテリアライズドビューのQeProcessorImpl問題によるメモリリークが発生することがあります。 [#15699](https://github.com/StarRocks/starrocks/pull/15699)
- `limit` を使用したクエリの結果が一貫性がないことがあります。 [#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERTによるメモリリークが発生することがあります。 [#14718](https://github.com/StarRocks/starrocks/pull/14718)
- Primary KeyテーブルがTablet Migrationを実行します。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker Load中にBrokerのKerberosチケットがタイムアウトすることがあります。 [#16149](https://github.com/StarRocks/starrocks/pull/16149)
- テーブルのビューにおける`nullable`情報が誤って推測されることがあります。 [#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 動作変更

- ThriftのListenバックログのデフォルト値を`1024`に変更しました。 [#13911](https://github.com/StarRocks/starrocks/pull/13911)
- SQLモード`FORBID_INVALID_DATES`を追加しました。このSQLモードはデフォルトで無効です。有効にすると、StarRocksはDATE型の入力を検証し、無効な入力がある場合はエラーを返します。 [#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

リリース日: 2022年12月14日

### 改善点

- 多数のバケットが存在する場合のBucket Hintのパフォーマンスを最適化しました。 [#13142](https://github.com/StarRocks/starrocks/pull/13142)

### バグ修正

以下のバグが修正されました:

- Primary KeyインデックスのフラッシュによりBEがクラッシュすることがあります。 [#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- `SHOW FULL TABLES`でマテリアライズドビューの型が正しく識別されないことがあります。 [#13954](https://github.com/StarRocks/starrocks/pull/13954)
- StarRocks v2.2からv2.4へのアップグレードによりBEがクラッシュすることがあります。 [#13795](https://github.com/StarRocks/starrocks/pull/13795)
- Broker LoadによりBEがクラッシュすることがあります。 [#13973](https://github.com/StarRocks/starrocks/pull/13973)
- セッション変数`statistic_collect_parallel`が機能しないことがあります。 [#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTOによりBEがクラッシュすることがあります。 [#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDFによりBEがクラッシュすることがあります。 [#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 部分更新中にレプリカをクローンすると、BEがクラッシュし再起動に失敗することがあります。 [#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Colocated Joinが機能しないことがあります。 [#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 動作変更

- セッション変数`query_timeout`の上限を`259200`、下限を`1`に制限しました。
- FEパラメータ`default_storage_medium`を非推奨にしました。テーブルのストレージメディアはシステムによって自動的に推測されます。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

リリース日: 2022年11月14日

### 新機能

- 非等価結合（LEFT SEMI JOINおよびANTI JOIN）をサポートします。JOIN機能を最適化しました。 [#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 改善点

- `HeartbeatResponse`の`aliveStatus`プロパティをサポートします。`aliveStatus`はノードがクラスタ内で生存しているかどうかを示します。`aliveStatus`を判断するメカニズムがさらに最適化されました。 [#12713](https://github.com/StarRocks/starrocks/pull/12713)

- Routine Loadのエラーメッセージを最適化しました。 [#12155](https://github.com/StarRocks/starrocks/pull/12155)

### バグ修正

- v2.4.0RCからv2.4.0へのアップグレード後にBEがクラッシュすることがあります。 [#13128](https://github.com/StarRocks/starrocks/pull/13128)

- 遅延マテリアライズにより、データレイク上のクエリ結果が不正確になることがあります。 [#13133](https://github.com/StarRocks/starrocks/pull/13133)

- `get_json_int`関数が例外をスローすることがあります。 [#12997](https://github.com/StarRocks/starrocks/pull/12997)

- 永続インデックスを持つPRIMARY KEYテーブルからの削除後にデータの不整合が生じることがあります。[#12719](https://github.com/StarRocks/starrocks/pull/12719)

- PRIMARY KEYテーブルのコンパクション中にBEがクラッシュすることがあります。 [#12914](https://github.com/StarRocks/starrocks/pull/12914)

- 入力に空文字が含まれる場合、`json_object`関数が不正な結果を返すことがあります。 [#13030](https://github.com/StarRocks/starrocks/issues/13030)

- `RuntimeFilter`によりBEがクラッシュすることがあります。 [#12807](https://github.com/StarRocks/starrocks/pull/12807)

- CBOの過度な再帰計算によりFEがハングすることがあります。 [#12788](https://github.com/StarRocks/starrocks/pull/12788)

- 正常終了時にBEがクラッシュしたりエラーを報告したりすることがあります。 [#12852](https://github.com/StarRocks/starrocks/pull/12852)

- 新しい列を追加したテーブルからのデータ削除後にコンパクションがクラッシュすることがあります。 [#12907](https://github.com/StarRocks/starrocks/pull/12907)

- OLAP外部テーブルのメタデータ同期の不正なメカニズムによりデータの不整合が生じることがあります。 [#12368](https://github.com/StarRocks/starrocks/pull/12368)

- 1つのBEがクラッシュした場合、他のBEがタイムアウトまで関連するクエリを実行することがあります。 [#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 動作変更

- Hive外部テーブルの解析に失敗した場合、StarRocksは関連する列をNULL列に変換する代わりにエラーメッセージを投げます。 [#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

リリース日: 2022年10月20日

### 新機能

- 複数のベーステーブルに基づく非同期マテリアライズドビューの作成をサポートし、JOIN操作を使用したクエリの高速化を実現します。非同期マテリアライズドビューは、すべての[テーブルタイプ](../table_design/table_types/table_types.md)に対応しています。詳細は[マテリアライズドビュー](../using_starrocks/Materialized_view.md)をご覧ください。

- `INSERT OVERWRITE`を使用したデータの上書きをサポートします。詳細については、[INSERTを使用したデータのロード](../loading/InsertInto.md)を参照してください。

- [プレビュー] 水平方向にスケーリング可能なステートレスなCompute Nodes (CN)を提供します。StarRocks Operatorを使用してCNをKubernetes (K8s)クラスターにデプロイし、自動水平スケーリングを実現できます。詳細については、[KubernetesでStarRocks Operatorを使用したCNのデプロイと管理](../deployment/sr_operator.md)を参照してください。

- Outer Joinは`<`、`<=`、`>`、`>=`、`<>`などの比較演算子を使用した非等価結合をサポートします。詳細については、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

- IcebergカタログとHudiカタログの作成をサポートし、Apache IcebergおよびApache Hudiからのデータに直接クエリを実行できます。詳細については、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)と[Hudiカタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

- CSV形式のApache Hive™テーブルのARRAY型カラムのクエリをサポートします。詳細については、[外部テーブル](../data_source/External_table.md)を参照してください。

- `DESC`を使用して外部データのスキーマを表示する機能をサポートします。詳細については、[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)を参照してください。

- `GRANT`を使用してユーザーに特定のロールまたは`IMPERSONATE`権限を付与し、`REVOKE`でそれらを取り消し、`EXECUTE AS`で`IMPERSONATE`権限を持つSQLステートメントを実行する機能をサポートします。詳細については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)、および[EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)を参照してください。

- FQDNアクセスをサポート：ドメイン名またはホスト名とポートの組み合わせをBEまたはFEノードの一意の識別子として使用できます。これにより、IPアドレスの変更によるアクセス障害を防ぐことができます。詳細については、[FQDNアクセスの有効化](../administration/enable_fqdn.md)を参照してください。

- `flink-connector-starrocks`は、プライマリキーテーブルの部分更新をサポートします。詳細については、[flink-connector-starrocksを使用したデータのロード](../loading/Flink-connector-starrocks.md)を参照してください。

- 以下の新機能を提供します：

  - `array_contains_all`：特定の配列が別の配列のサブセットであるかどうかをチェックします。詳細については、[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)を参照してください。
  - `percentile_cont`：線形補間によるパーセンタイル値を計算します。詳細については、[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)を参照してください。

### 改善点

- プライマリキーテーブルは、VARCHAR型プライマリキーインデックスをディスクにフラッシュすることをサポートします。バージョン2.4.0から、プライマリキーテーブルは、永続的なプライマリキーインデックスが有効かどうかに関わらず、プライマリキーインデックスに対して同じデータ型をサポートします。

- 外部テーブルのクエリパフォーマンスを最適化しました。

  - Parquet形式の外部テーブルに対するクエリで遅延マテリアライゼーションをサポートし、小規模なフィルタリングが行われるデータレイクでのクエリパフォーマンスを最適化します。
  - 小規模なI/O操作をマージしてデータレイクでのクエリ遅延を減らし、外部テーブルのクエリパフォーマンスを向上させます。

- ウィンドウ関数のパフォーマンスを最適化しました。

- 述語プッシュダウンをサポートすることで、クロスジョインのパフォーマンスを最適化しました。

- CBO統計にヒストグラムを追加しました。完全な統計収集がさらに最適化されています。詳細については、[CBO統計の収集](../using_starrocks/Cost_based_optimizer.md)を参照してください。

- タブレットスキャンに適応型マルチスレッディングを有効にし、タブレット数に対するスキャンパフォーマンスの依存性を軽減しました。その結果、バケット数をより容易に設定できます。詳細については、[バケット数の決定](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)を参照してください。

- Apache Hiveでの圧縮されたTXTファイルのクエリをサポートします。

- マルチインスタンス展開時のOOM問題を回避するために、デフォルトのPageCacheサイズ計算とメモリ整合性チェックのメカニズムを調整しました。

- final_merge操作を削除することで、プライマリキーテーブルでの大規模バッチロードのパフォーマンスを2倍に向上させました。

- Apache Flink®やApache Kafka®などの外部システムからデータをロードするトランザクションの2フェーズコミット（2PC）を実装するストリームロードトランザクションインターフェースをサポートし、高い同時実行性を持つストリームロードのパフォーマンスを向上させます。

- 関数：

  - 1つのステートメントで複数の`COUNT(DISTINCT)`を使用できます。詳細については、[count](../sql-reference/sql-functions/aggregate-functions/count.md)を参照してください。
  - ウィンドウ関数`min()`と`max()`はスライディングウィンドウをサポートします。詳細については、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。
  - `window_funnel`関数のパフォーマンスを最適化しました。詳細については、[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)を参照してください。

### バグ修正

以下のバグが修正されました：

- `DESC`によって返される`DECIMAL`データタイプが、`CREATE TABLE`ステートメントで指定されたものと異なります。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- FEのメタデータ管理の問題がFEの安定性に影響を与えます。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- データロード関連の問題：

  - `ARRAY`列が指定された場合、ロードが失敗します。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - Broker Loadを介して非Duplicate Keyテーブルにデータがロードされた後、レプリカが一致しません。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - `ALTER ROUTINE LOAD`の実行時にNPEが発生します。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- データレイク分析関連の問題：

  - Hive外部テーブルのParquetデータに対するクエリが失敗します。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - Elasticsearch外部テーブルで`limit`句を含むクエリに対して不正な結果が返されます。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 複合データ型を持つApache Icebergテーブルに対するクエリ中に未知のエラーが発生します。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- Leader FEノードとFollower FEノード間でメタデータが一致しません。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- `BITMAP`データのサイズが2GBを超えると、BEがクラッシュします。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 動作変更

- ページキャッシュはデフォルトで有効です（"disable_storage_page_cache" = "false"）。デフォルトのキャッシュサイズ（`storage_page_cache_limit`）はシステムメモリの20%です。
- CBOはデフォルトで有効です。セッション変数`enable_cbo`は非推奨となりました。
- ベクトル化エンジンはデフォルトで有効です。セッション変数`vectorized_engine_enable`は非推奨となりました。

### その他

- リソースグループの安定版リリースを発表します。
- JSONデータ型および関連する関数の安定版リリースを発表します。
