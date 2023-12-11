---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 1.19

## 1.19.0

リリース日: 2021年10月22日

### 新機能

* グローバルランタイムフィルターを実装し、シャッフル結合用のランタイムフィルターを有効にできるようにしました。
* CBO Planner がデフォルトで有効化され、共有配置結合やバケットシャッフル、統計情報の推定などが改善されました。
* [実験的な機能] プライマリキーテーブルのリリース: リアルタイム/頻繁な更新機能をサポートするために、StarRocks は新しいテーブルタイプであるプライマリキーテーブルを追加しました。プライマリキーテーブルは、Stream Load、Broker Load、Routine Load をサポートし、また Flink-cdc に基づく MySQL データのための第二レベル同期ツールを提供します。
* [実験的な機能] 外部テーブルへの書き込み機能をサポート。外部テーブルを使用して別の StarRocks クラスターテーブルにデータを書き込むことができ、読み書きの分離要件を解決し、より良いリソースの分離を提供します。

### 改善

#### StarRocks

* パフォーマンスの最適化。
  * count distinct int ステートメント
  * group by int ステートメント
  * or ステートメント
* ディスクバランスアルゴリズムの最適化。単一のマシンにディスクを追加した後、データが自動的にバランスされます。
* 部分的なカラムエクスポートをサポート。
* show processlist を最適化して特定の SQL を表示するようにします。
* SET_VAR で複数の変数設定をサポート。
* テーブルシンク、ルーチンロード、マテリアライズドビューの作成など、エラー報告情報の改善。

#### StarRocks-DataX コネクタ

* インターバルフラッシュ StarRocks-DataX Writer の設定をサポート。

### バグ修正

* データ復旧操作が完了した後に動的パーティションテーブルが自動的に作成されない問題を修正しました。[# 337](https://github.com/StarRocks/starrocks/issues/337)
* CBO を開いた後に row_number 関数によるエラーが報告される問題を修正しました。
* 統計情報の収集による FE の動作停止問題を修正しました。
* set_var がセッションに影響を与えるがステートメントには影響を与えない問題を修正しました。
* Hive パーティション外部テーブルで select count(*) が異常な結果を返す問題を修正しました。

## 1.19.1

リリース日: 2021年11月2日

### 改善

* `show frontends` のパフォーマンスを最適化します。[# 507](https://github.com/StarRocks/starrocks/pull/507) [# 984](https://github.com/StarRocks/starrocks/pull/984)
* 遅いクエリの監視を追加します。[# 502](https://github.com/StarRocks/starrocks/pull/502) [# 891](https://github.com/StarRocks/starrocks/pull/891)
* Hive 外部メタデータのフェッチを並列フェッチして最適化します。[# 425](https://github.com/StarRocks/starrocks/pull/425) [# 451](https://github.com/StarRocks/starrocks/pull/451)

### バグ修正

* Thrift プロトコルの互換性の問題を修正し、Hive 外部テーブルが Kerberos で接続できるようにしました。[# 184](https://github.com/StarRocks/starrocks/pull/184) [# 947](https://github.com/StarRocks/starrocks/pull/947) [# 995](https://github.com/StarRocks/starrocks/pull/995) [# 999](https://github.com/StarRocks/starrocks/pull/999)
* ビュー作成に関するいくつかのバグを修正しました。[# 972](https://github.com/StarRocks/starrocks/pull/972) [# 987](https://github.com/StarRocks/starrocks/pull/987)[# 1001](https://github.com/StarRocks/starrocks/pull/1001)
* FE をグレースケールでアップグレードできない問題を修正しました。[# 485](https://github.com/StarRocks/starrocks/pull/485) [# 890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

リリース日: 2021年11月20日

### 改善

* バケットシャッフル結合が right join と full outer join をサポート [# 1209](https://github.com/StarRocks/starrocks/pull/1209)  [# 31234](https://github.com/StarRocks/starrocks/pull/1234)

### バグ修正

* 重複ノードが predicate push-down を行えない問題を修正[# 1410](https://github.com/StarRocks/starrocks/pull/1410) [# 1417](https://github.com/StarRocks/starrocks/pull/1417)
* クラスターがリーダーノードを変更中に routine load がデータを失う可能性がある問題を修正。[# 1074](https://github.com/StarRocks/starrocks/pull/1074) [# 1272](https://github.com/StarRocks/starrocks/pull/1272)
* ビュー作成が union をサポートできない問題を修正 [# 1083](https://github.com/StarRocks/starrocks/pull/1083)
* Hive 外部テーブルの安定性のいくつかの問題を修正[# 1408](https://github.com/StarRocks/starrocks/pull/1408)
* グループバイビューの問題を修正[# 1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

リリース日: 2021年11月30日

### 改善

* セキュリティを改善するため、jprotobuf のバージョンをアップグレードしました。[# 1506](https://github.com/StarRocks/starrocks/issues/1506)

### バグ修正

* グループバイの結果の正確性に関する問題を修正しました。
* グループ化セットに関する問題を修正[# 1395](https://github.com/StarRocks/starrocks/issues/1395) [# 1119](https://github.com/StarRocks/starrocks/pull/1119)
* date_format のいくつかの指標に関する問題を修正
* 集約されたストリーミングの境界条件の問題を修正[# 1584](https://github.com/StarRocks/starrocks/pull/1584)
* 詳細については[リンク](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)を参照してください。

## 1.19.4

リリース日: 2021年12月9日

### 改善

* cast(varchar as bitmap) をサポート [# 1941](https://github.com/StarRocks/starrocks/pull/1941)
* Hive 外部テーブルのアクセスポリシーを更新する[# 1394](https://github.com/StarRocks/starrocks/pull/1394) [# 1807](https://github.com/StarRocks/starrocks/pull/1807)

### バグ修正

* predicate Cross Join によるクエリ結果の間違いの問題を修正 [# 1918](https://github.com/StarRocks/starrocks/pull/1918)
* decimal 型および time 型の変換の問題を修正 [# 1709](https://github.com/StarRocks/starrocks/pull/1709) [# 1738](https://github.com/StarRocks/starrocks/pull/1738)
* 共有結合/複製結合の選択エラーの問題を修正 [# 1727](https://github.com/StarRocks/starrocks/pull/1727)
* いくつかのプランコスト計算の問題を修正

## 1.19.5

リリース日: 2021年12月20日

### 改善

* シャッフル結合を最適化するための計画 [# 2184](https://github.com/StarRocks/starrocks/pull/2184)
* 複数の大きなファイルのインポートを最適化 [# 2067](https://github.com/StarRocks/starrocks/pull/2067)

### バグ修正

* Log4j2 を 2.17.0 にアップグレードし、セキュリティの脆弱性を修正[# 2284](https://github.com/StarRocks/starrocks/pull/2284)[# 2290](https://github.com/StarRocks/starrocks/pull/2290)
* Hive 外部テーブルで空のパーティションの問題を修正[# 707](https://github.com/StarRocks/starrocks/pull/707)[# 2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

リリース日: 2022年3月18日

### バグ修正

以下のバグが修正されました:

* データフォーマットが異なるバージョンで異なる結果を生成する問題を修正しました。[#4165](https://github.com/StarRocks/starrocks/pull/4165)
* データロード時に誤って Parquet ファイルが削除されることにより BE ノードが失敗する可能性がある問題を修正しました。[#3521](https://github.com/StarRocks/starrocks/pull/3521)