---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 1.19

## 1.19.0

リリース日: 2021年10月22日

### 新機能

* グローバルランタイムフィルターを実装しました。これにより、シャッフルジョインに対してランタイムフィルターを有効にすることができます。
* CBO Plannerがデフォルトで有効になり、コロケートジョイン、バケットシャッフル、統計情報の推定などが改善されました。
* [実験的な機能] プライマリキーテーブルリリース: リアルタイム/頻繁な更新機能をより良くサポートするために、StarRocksには新しいテーブルタイプであるプライマリキーテーブルが追加されました。プライマリキーテーブルは、ストリームロード、ブローカーロード、ルーチンロードをサポートし、またFlink-cdcに基づくMySQLデータのセカンドレベルの同期ツールも提供します。
* [実験的な機能] 外部テーブルへの書き込み機能をサポートしました。外部テーブルによって別のStarRocksクラスターテーブルにデータを書き込むことができ、読み取り/書き込みの分離要件を解決し、より良いリソースの分離を提供します。

### 改善点

#### StarRocks

* パフォーマンスの最適化。
  * count distinct int ステートメント
  * group by int ステートメント
  * or ステートメント
* ディスクバランスアルゴリズムを最適化しました。ディスクを単一のマシンに追加した後、データを自動的にバランスすることができます。
* 部分的なカラムのエクスポートをサポートします。
* 特定のSQLを表示するためにshow processlistを最適化しました。
* SET_VARで複数の変数設定をサポートします。
* テーブルシンク、ルーチンロード、マテリアライズドビューの作成など、エラーレポート情報を改善しました。

#### StarRocks-DataXコネクタ

* StarRocks-DataX Writerのインターバルフラッシュを設定する機能をサポートします。

### バグ修正

* データ回復操作が完了した後、動的パーティションテーブルが自動的に作成されない問題を修正しました。[# 337](https://github.com/StarRocks/starrocks/issues/337)
* CBOがオープンされた後、row_number関数でエラーが報告される問題を修正しました。
* 統計情報の収集によるFEのスタック問題を修正しました。
* set_varがセッションに対して効果があり、ステートメントに対して効果がない問題を修正しました。
* Hiveパーティション外部テーブルでselect count(*)が異常な結果を返す問題を修正しました。

## 1.19.1

リリース日: 2021年11月2日

### 改善点

* `show frontends`のパフォーマンスを最適化しました。[# 507](https://github.com/StarRocks/starrocks/pull/507) [# 984](https://github.com/StarRocks/starrocks/pull/984)
* 遅いクエリの監視を追加しました。[# 502](https://github.com/StarRocks/starrocks/pull/502) [# 891](https://github.com/StarRocks/starrocks/pull/891)
* Hive外部メタデータのフェッチを並列フェッチするための最適化を行いました。[# 425](https://github.com/StarRocks/starrocks/pull/425) [# 451](https://github.com/StarRocks/starrocks/pull/451)

### バグ修正

* Thriftプロトコルの互換性の問題を修正し、Hive外部テーブルをKerberosで接続できるようにしました。[# 184](https://github.com/StarRocks/starrocks/pull/184) [# 947](https://github.com/StarRocks/starrocks/pull/947) [# 995](https://github.com/StarRocks/starrocks/pull/995) [# 999](https://github.com/StarRocks/starrocks/pull/999)
* ビュー作成時のいくつかのバグを修正しました。[# 972](https://github.com/StarRocks/starrocks/pull/972) [# 987](https://github.com/StarRocks/starrocks/pull/987)[# 1001](https://github.com/StarRocks/starrocks/pull/1001)
* FEがグレースケールでアップグレードできない問題を修正しました。[# 485](https://github.com/StarRocks/starrocks/pull/485) [# 890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

リリース日: 2021年11月20日

### 改善点

* バケットシャッフルジョインは、右ジョインとフルアウタージョインをサポートします。[# 1209](https://github.com/StarRocks/starrocks/pull/1209)  [# 31234](https://github.com/StarRocks/starrocks/pull/1234)

### バグ修正

* リピートノードがプレディケートプッシュダウンを行えない問題を修正しました。[# 1410](https://github.com/StarRocks/starrocks/pull/1410) [# 1417](https://github.com/StarRocks/starrocks/pull/1417)
* クラスターがリーダーノードを変更する間にルーチンロードがデータを失う可能性がある問題を修正しました。[# 1074](https://github.com/StarRocks/starrocks/pull/1074) [# 1272](https://github.com/StarRocks/starrocks/pull/1272)
* ビューの作成がunionをサポートしない問題を修正しました。[# 1083](https://github.com/StarRocks/starrocks/pull/1083)
* Hive外部テーブルの安定性の問題を修正しました。[# 1408](https://github.com/StarRocks/starrocks/pull/1408)
* グループバイビューの問題を修正しました。[# 1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

リリース日: 2021年11月30日

### 改善点

* jprotobufのバージョンをアップグレードしてセキュリティを改善しました。[# 1506](https://github.com/StarRocks/starrocks/issues/1506)

### バグ修正

* グループバイの結果の正確性に関するいくつかの問題を修正しました。
* グルーピングセットの問題を修正しました。[# 1395](https://github.com/StarRocks/starrocks/issues/1395) [# 1119](https://github.com/StarRocks/starrocks/pull/1119)
* date_formatのいくつかの指標の問題を修正しました。
* 集約ストリーミングの境界条件の問題を修正しました。[# 1584](https://github.com/StarRocks/starrocks/pull/1584)
* 詳細については、[リンク](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)を参照してください。

## 1.19.4

リリース日: 2021年12月9日

### 改善点

* cast(varchar as bitmap)をサポートしました。[# 1941](https://github.com/StarRocks/starrocks/pull/1941)
* Hive外部テーブルのアクセスポリシーを更新しました。[# 1394](https://github.com/StarRocks/starrocks/pull/1394) [# 1807](https://github.com/StarRocks/starrocks/pull/1807)

### バグ修正

* Cross Joinによるクエリ結果の間違いを修正しました。[# 1918](https://github.com/StarRocks/starrocks/pull/1918)
* decimal型とtime型の変換のバグを修正しました。[# 1709](https://github.com/StarRocks/starrocks/pull/1709) [# 1738](https://github.com/StarRocks/starrocks/pull/1738)
* コロケートジョイン/レプリケートジョインの選択エラーのバグを修正しました。[# 1727](https://github.com/StarRocks/starrocks/pull/1727)
* いくつかのプランのコスト計算の問題を修正しました。

## 1.19.5

リリース日: 2021年12月20日

### 改善点

* シャッフルジョインの最適化計画を実施する予定です。[# 2184](https://github.com/StarRocks/starrocks/pull/2184)
* 複数の大きなファイルのインポートを最適化します。[# 2067](https://github.com/StarRocks/starrocks/pull/2067)

### バグ修正

* Log4j2を2.17.0にアップグレードし、セキュリティの脆弱性を修正しました。[# 2284](https://github.com/StarRocks/starrocks/pull/2284)[# 2290](https://github.com/StarRocks/starrocks/pull/2290)
* Hive外部テーブルの空のパーティションの問題を修正しました。[# 707](https://github.com/StarRocks/starrocks/pull/707)[# 2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

リリース日: 2022年3月18日

### バグ修正

以下のバグが修正されました:

* データフォーマットは、異なるバージョンで異なる結果を生成します。[#4165](https://github.com/StarRocks/starrocks/pull/4165)
* データロード中にParquetファイルが誤って削除されることにより、BEノードが失敗する可能性があります。[#3521](https://github.com/StarRocks/starrocks/pull/3521)
