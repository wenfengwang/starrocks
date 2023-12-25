---
displayed_sidebar: English
---

# StarRocks バージョン 1.19

## 1.19.0

リリース日: 2021年10月22日

### 新機能

* シャッフルジョインに対するランタイムフィルタを有効にするグローバルランタイムフィルタを実装。
* CBO Plannerがデフォルトで有効化され、コロケートジョイン、バケットシャッフル、統計情報推定などが改善されました。
* [実験機能] プライマリキーテーブルリリース：リアルタイム/頻繁な更新機能をより良くサポートするため、StarRocksは新しいテーブルタイプ「プライマリキーテーブル」を追加しました。プライマリキーテーブルはStream Load、Broker Load、Routine Loadをサポートし、Flink-cdcに基づくMySQLデータの秒単位の同期ツールも提供します。
* [実験機能] 外部テーブルの書き込み機能に対応。外部テーブルを通じて別のStarRocksクラスターテーブルにデータを書き込むことをサポートし、読み取り/書き込みの分離要件を満たし、リソースの分離を向上させます。

### 改善点

#### StarRocks

* パフォーマンス最適化。
  * count distinct int ステートメント
  * group by int ステートメント
  * or ステートメント
* ディスクバランスアルゴリズムの最適化。シングルマシンにディスクを追加した後、データは自動的にバランスされます。
* 部分的なカラムエクスポートに対応。
* 特定のSQLを表示するshow processlistの最適化。
* SET_VARで複数の変数設定に対応。
* エラーレポート情報の改善、table_sink、ルーチンロード、マテリアライズドビューの作成などを含む。

#### StarRocks-DataX コネクタ

* StarRocks-DataX Writerのインターバルフラッシュ設定に対応。

### バグ修正

* データ復旧操作完了後に動的パーティションテーブルが自動的に作成されない問題を修正。[#337](https://github.com/StarRocks/starrocks/issues/337)
* CBO有効化後にrow_number関数でエラーが報告される問題を修正。
* 統計情報収集によるFEのスタック問題を修正。
* set_varがセッションには適用されるがステートメントには適用されない問題を修正。
* Hiveパーティション外部テーブルでselect count(*)が異常な値を返す問題を修正。

## 1.19.1

リリース日: 2021年11月2日

### 改善点

* `show frontends`のパフォーマンス最適化。[#507](https://github.com/StarRocks/starrocks/pull/507) [#984](https://github.com/StarRocks/starrocks/pull/984)
* 遅延クエリの監視機能を追加。[#502](https://github.com/StarRocks/starrocks/pull/502) [#891](https://github.com/StarRocks/starrocks/pull/891)
* Hive外部メタデータの取得を最適化し、並行取得を実現。[#425](https://github.com/StarRocks/starrocks/pull/425) [#451](https://github.com/StarRocks/starrocks/pull/451)

### バグ修正

* Thriftプロトコルの互換性問題を修正し、Hive外部テーブルがKerberosと接続できるようにする。[#184](https://github.com/StarRocks/starrocks/pull/184) [#947](https://github.com/StarRocks/starrocks/pull/947) [#995](https://github.com/StarRocks/starrocks/pull/995) [#999](https://github.com/StarRocks/starrocks/pull/999)
* ビュー作成に関する複数のバグを修正。[#972](https://github.com/StarRocks/starrocks/pull/972) [#987](https://github.com/StarRocks/starrocks/pull/987) [#1001](https://github.com/StarRocks/starrocks/pull/1001)
* グレースケールでのFEアップグレードができない問題を修正。[#485](https://github.com/StarRocks/starrocks/pull/485) [#890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

リリース日: 2021年11月20日

### 改善点

* バケットシャッフルジョインが右外部結合と完全外部結合をサポート。[#1209](https://github.com/StarRocks/starrocks/pull/1209) [#1234](https://github.com/StarRocks/starrocks/pull/1234)

### バグ修正

* リピートノードが述語プッシュダウンを行えない問題を修正。[#1410](https://github.com/StarRocks/starrocks/pull/1410) [#1417](https://github.com/StarRocks/starrocks/pull/1417)
* インポート中にクラスターがリーダーノードを変更すると、ルーチンロードでデータが失われる可能性がある問題を修正。[#1074](https://github.com/StarRocks/starrocks/pull/1074) [#1272](https://github.com/StarRocks/starrocks/pull/1272)
* ビュー作成がunionをサポートできない問題を修正。[#1083](https://github.com/StarRocks/starrocks/pull/1083)
* Hive外部テーブルの安定性に関するいくつかの問題を修正。[#1408](https://github.com/StarRocks/starrocks/pull/1408)
* ビューによるgroup byの問題を修正。[#1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

リリース日: 2021年11月30日

### 改善点

* jprotobufのバージョンをアップグレードしてセキュリティを向上。[#1506](https://github.com/StarRocks/starrocks/issues/1506)

### バグ修正

* group byの結果の正確性に関する問題を修正。
* grouping setsに関する問題を修正。[#1395](https://github.com/StarRocks/starrocks/issues/1395) [#1119](https://github.com/StarRocks/starrocks/pull/1119)
* date_formatの一部指標の問題を修正。
* 集約ストリーミングの境界条件の問題を修正。[#1584](https://github.com/StarRocks/starrocks/pull/1584)
* 詳細は[リンク](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)を参照してください。

## 1.19.4

リリース日: 2021年12月9日

### 改善点

* cast(varchar as bitmap)に対応。[#1941](https://github.com/StarRocks/starrocks/pull/1941)
* Hive外部テーブルのアクセスポリシーを更新。[#1394](https://github.com/StarRocks/starrocks/pull/1394) [#1807](https://github.com/StarRocks/starrocks/pull/1807)

### バグ修正

* 述語クロスジョインで誤ったクエリ結果のバグを修正。[#1918](https://github.com/StarRocks/starrocks/pull/1918)
* decimal型とtime型の変換のバグを修正。[#1709](https://github.com/StarRocks/starrocks/pull/1709) [#1738](https://github.com/StarRocks/starrocks/pull/1738)
* コロケートジョイン/レプリケートジョインの選択エラーのバグを修正。[#1727](https://github.com/StarRocks/starrocks/pull/1727)
* プランコスト計算の問題を修正。

## 1.19.5

リリース日: 2021年12月20日

### 改善点

* シャッフルジョインの最適化計画。[#2184](https://github.com/StarRocks/starrocks/pull/2184)
* 多数の大ファイルインポートの最適化。[#2067](https://github.com/StarRocks/starrocks/pull/2067)

### バグ修正

* Log4j2を2.17.0にアップグレードし、セキュリティの脆弱性を修正。[#2284](https://github.com/StarRocks/starrocks/pull/2284) [#2290](https://github.com/StarRocks/starrocks/pull/2290)
* Hive外部テーブルの空のパーティションの問題を修正。[#707](https://github.com/StarRocks/starrocks/pull/707) [#2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

リリース日: 2022年3月18日

### バグ修正

以下のバグが修正されました：

* 異なるバージョンでdata_formatが異なる結果を生成する問題を修正。[#4165](https://github.com/StarRocks/starrocks/pull/4165)
* データロード中にParquetファイルが誤って削除されることにより、BEノードが失敗する問題を修正。[#3521](https://github.com/StarRocks/starrocks/pull/3521)
