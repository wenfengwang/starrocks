---
displayed_sidebar: Chinese
---

# StarRocks バージョン 1.19

## 1.19.0

リリース日：2021年10月25日

### 新機能

* Global Runtime Filterを実装し、shuffle joinに対するRuntime filterをサポート。
* CBO Plannerをデフォルトで有効化し、colocate join / bucket shuffle / 統計情報などの機能を強化。[参考文献](../using_starrocks/Cost_based_optimizer.md)
* [実験機能] 主キーモデル（Primary Key）をリリース：リアルタイム/頻繁な更新機能をより良くサポートするため、StarRocksは新しいテーブルタイプを追加しました：主キーモデル。このモデルはStream Load、Broker Load、Routine Loadをサポートし、Flink-cdcを用いたMySQLデータの秒単位での同期ツールを提供します。[参考文献](../table_design/table_types/primary_key_table.md)
* [実験機能] 外部テーブル書き込み機能を新たに追加。別のStarRocksクラスタのテーブルにデータを外部テーブル経由で書き込むことをサポートし、読み書き分離の要求を解決し、より良いリソース分離を提供します。[参考文献](../data_source/External_table.md)

### 改善点

#### StarRocks

* パフォーマンス最適化：
  * count distinct intステートメント
  * group by int ステートメント
  * orステートメント
* ディスクバランスアルゴリズムを最適化し、単一ノードにディスクを追加した後に自動的にデータバランスを行うことができます。
* 一部の列のエクスポートをサポート。 [参考文献](../unloading/Export.md)
* show processlistを最適化し、具体的なSQLを表示。
* SET_VARが複数の変数設定をサポート。
* table_sink、routine load、物化ビューの作成など、より多くのエラーメッセージを完備。

#### StarRocks-Datax コネクタ

* StarRocks-DataX Writerがインターバルフラッシュの設定をサポート。

### バグ修正

* データリカバリジョブが完了した後に新しいパーティションが自動的に作成されない問題を修正。 [#337](https://github.com/StarRocks/starrocks/issues/337)
* CBOを有効にした後にrow_number関数でエラーが発生する問題を修正。
* 統計情報の収集がFEをフリーズさせる問題を修正。
* set_varがセッションに対してではなくステートメントに対して効果を発揮する問題を修正。
* Hiveパーティション外部テーブルの`select count(*)`が異常な結果を返す問題を修正。

## 1.19.1

リリース日：2021年11月2日

### 改善点

* show frontendsのパフォーマンスを最適化。 [#507](https://github.com/StarRocks/starrocks/pull/507) [#984](https://github.com/StarRocks/starrocks/pull/984)
* 遅いクエリの監視を補完。 [#502](https://github.com/StarRocks/starrocks/pull/502) [#891](https://github.com/StarRocks/starrocks/pull/891)
* Hive外部テーブルのメタデータ取得を最適化し、メタデータの並行取得をサポート。[#425](https://github.com/StarRocks/starrocks/pull/425) [#451](https://github.com/StarRocks/starrocks/pull/451)

### バグ修正

* Thriftプロトコルの互換性問題を修正し、Hive外部テーブルがKerberosと連携する問題を解決。 [#184](https://github.com/StarRocks/starrocks/pull/184) [#947](https://github.com/StarRocks/starrocks/pull/947) [#995](https://github.com/StarRocks/starrocks/pull/995) [#999](https://github.com/StarRocks/starrocks/pull/999)
* ビュー作成のいくつかのバグを修正。 [#972](https://github.com/StarRocks/starrocks/pull/972) [#987](https://github.com/StarRocks/starrocks/pull/987)[#1001](https://github.com/StarRocks/starrocks/pull/1001)
* FEがグレードアップグレードできない問題を修正。 [#485](https://github.com/StarRocks/starrocks/pull/485) [#890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

リリース日：2021年11月20日

### 改善点

* bucket shuffle joinがright joinとfull outer joinをサポート。 [#1209](https://github.com/StarRocks/starrocks/pull/1209) [#1234](https://github.com/StarRocks/starrocks/pull/1234)

### バグ修正

* repeat nodeが述語下推を行えない問題を修正。 [#1410](https://github.com/StarRocks/starrocks/pull/1410) [#1417](https://github.com/StarRocks/starrocks/pull/1417)
* クラスタのマスタースイッチシナリオでroutine loadがデータを失う可能性がある問題を修正。 [#1074](https://github.com/StarRocks/starrocks/pull/1074) [#1272](https://github.com/StarRocks/starrocks/pull/1272)
* ビュー作成がunionをサポートできない問題を修正。 [#1083](https://github.com/StarRocks/starrocks/pull/1083)
* Hive外部テーブルの安定性に関するいくつかの問題を修正。 [#1408](https://github.com/StarRocks/starrocks/pull/1408)
* group byビューの問題を修正。 [#1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

リリース日：2021年11月30日

### 改善点

* jprotobufのバージョンをアップグレードしてセキュリティを向上。 [#1506](https://github.com/StarRocks/starrocks/issues/1506)

### バグ修正

* 一部のgroup byの結果の正確性に関する問題を修正。
* grouping setsのいくつかの問題を修正。 [#1395](https://github.com/StarRocks/starrocks/issues/1395) [#1119](https://github.com/StarRocks/starrocks/pull/1119)
* date_formatの一部のパラメータの問題を修正。
* 集約ストリーミングの境界条件の問題を修正。 [#1584](https://github.com/StarRocks/starrocks/pull/1584)
* 詳細は[リンク](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)を参照してください。

## 1.19.4

リリース日：2021年12月09日

### 改善点

* cast(varchar as bitmap)をサポート。 [#1941](https://github.com/StarRocks/starrocks/pull/1941)
* Hive外部テーブルのアクセスポリシーを更新。 [#1394](https://github.com/StarRocks/starrocks/pull/1394) [#1807](https://github.com/StarRocks/starrocks/pull/1807)

### バグ修正

* 谓词を含むCross Joinのクエリ結果が誤っているバグを修正。 [#1918](https://github.com/StarRocks/starrocks/pull/1918)
* decimal型とtime型の変換バグを修正。 [#1709](https://github.com/StarRocks/starrocks/pull/1709) [#1738](https://github.com/StarRocks/starrocks/pull/1738)
* colocate join/replicate joinの選択ミスバグを修正。 [#1727](https://github.com/StarRocks/starrocks/pull/1727)
* 複数のプランコスト計算問題を修正。

## 1.19.5

リリース日：2021年12月20日

### 改善点

* shuffle joinの計画を最適化。 [#2184](https://github.com/StarRocks/starrocks/pull/2184)
* 複数の大きなファイルのインポートを最適化。 [#2067](https://github.com/StarRocks/starrocks/pull/2067)

### バグ修正

* Log4j2を2.17.0にアップグレードし、セキュリティの脆弱性を修正。 [#2284](https://github.com/StarRocks/starrocks/pull/2284)[#2290](https://github.com/StarRocks/starrocks/pull/2290)
* Hive外部テーブルの空のパーティションの問題を修正。 [#707](https://github.com/StarRocks/starrocks/pull/707)[#2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

リリース日：2022年3月18日

### バグ修正

* 異なるバージョンでdateformatの出力結果が一貫性がない問題を修正。 [#4165](https://github.com/StarRocks/starrocks/pull/4165)
* データのインポート時に誤ってParquetファイルを削除することでBEノードがクラッシュする問題を修正。 [#3521](https://github.com/StarRocks/starrocks/pull/3521)
