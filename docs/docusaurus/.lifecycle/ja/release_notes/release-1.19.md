---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 1.19

## 1.19.0

リリース日：2021年10月22日

### 新機能

* グローバルランタイムフィルタを実装し、シャッフルジョイン用のランタイムフィルタを有効にできます。
* CBOプランナーがデフォルトで有効になり、共同配置ジョイン、バケットシャッフル、統計情報推定などが改善されました。
* [実験的な機能] プライマリキーテーブルリリース：リアルタイム／頻繁な更新機能をよりよくサポートするために、StarRocksは新しいテーブルタイプであるプライマリキーテーブルを追加しました。プライマリキーテーブルは、ストリームロード、ブローカーロード、定期ロードをサポートし、またFlink-cdcに基づくMySQLデータのためのセカンドレベル同期ツールを提供します。
* [実験的な機能] 外部テーブル向けの書き込み機能をサポートします。外部テーブルを使用して別のStarRocksクラスタテーブルにデータを書き込むことで、読み書きの分離要件を解決し、より良いリソース分離を提供します。

### 改善点

#### StarRocks

* パフォーマンスの最適化。
  * distinct int ステートメントのカウント
  * int ステートメントのグループ化
  * or ステートメント
* ディスクバランスアルゴリズムの最適化。データは、単一のマシンにディスクを追加した後に自動的にバランスをとることができます。
* 部分的なカラムエクスポートをサポートします。
* show processlistを最適化して特定のSQLを表示します。
* SET_VARで複数の変数設定をサポートします。
* テーブルシンク、ルーチンロード、マテリアライズドビューの作成など、エラー報告情報を改善します。

#### StarRocks-DataXコネクタ

* StarRocks-DataX Writerのインターバルフラッシュを設定するサポートを提供します。

### バグ修正

* データ回復操作完了後に、ダイナミックパーティションテーブルが自動的に作成されない問題を修正しました。[# 337](https://github.com/StarRocks/starrocks/issues/337)
* CBOを開いた後のrow_number関数のエラーを修正しました。
* 統計情報収集によるFEのスタックの問題を修正しました。
* set_varがセッションに対して効果を発揮するが、ステートメントに対して効果を発揮しない問題を修正しました。
* Hiveパーティション外部テーブルでselect count(*)が異常な結果を返す問題を修正しました。

## 1.19.1

リリース日：2021年11月2日

### 改善点

* `show frontends`のパフォーマンスを最適化しました。[# 507](https://github.com/StarRocks/starrocks/pull/507) [# 984](https://github.com/StarRocks/starrocks/pull/984)
* 遅いクエリの監視を追加しました。[# 502](https://github.com/StarRocks/starrocks/pull/502) [# 891](https://github.com/StarRocks/starrocks/pull/891)
* Hive外部メタデータのフェッチングを並列化して最適化しました。[# 425](https://github.com/StarRocks/starrocks/pull/425) [# 451](https://github.com/StarRocks/starrocks/pull/451)

### バグ修正

* Thriftプロトコルの互換性の問題を修正し、Hive外部テーブルがKerberosと接続できるようにしました。[# 184](https://github.com/StarRocks/starrocks/pull/184) [# 947](https://github.com/StarRocks/starrocks/pull/947) [# 995](https://github.com/StarRocks/starrocks/pull/995) [# 999](https://github.com/StarRocks/starrocks/pull/999)
* ビュー作成における複数のバグを修正しました。[# 972](https://github.com/StarRocks/starrocks/pull/972) [# 987](https://github.com/StarRocks/starrocks/pull/987) [# 1001](https://github.com/StarRocks/starrocks/pull/1001)
* FEをグレースケールでアップグレードできない問題を修正しました。[# 485](https://github.com/StarRocks/starrocks/pull/485) [# 890](https://github.com/StarRocks/starrocks/pull/890)

## 1.19.2

リリース日：2021年11月20日

### 改善点

* バケットシャッフルジョインが右結合とフルアウタージョインをサポートするようになりました。[# 1209](https://github.com/StarRocks/starrocks/pull/1209) [# 31234](https://github.com/StarRocks/starrocks/pull/1234)

### バグ修正

* リピートノードがプレディケートプッシュダウンを行うことができない問題を修正しました。[# 1410](https://github.com/StarRocks/starrocks/pull/1410) [# 1417](https://github.com/StarRocks/starrocks/pull/1417)
* クラスタのリーダーノードの変更中にルーチンロードがデータを失う可能性がある問題を修正しました。[# 1074](https://github.com/StarRocks/starrocks/pull/1074) [# 1272](https://github.com/StarRocks/starrocks/pull/1272)
* ビューの作成がunionをサポートしない問題を修正しました。[# 1083](https://github.com/StarRocks/starrocks/pull/1083)
* Hive外部テーブルの安定性のいくつかの問題を修正しました。[# 1408](https://github.com/StarRocks/starrocks/pull/1408)
* group byビューの問題を修正しました。[# 1231](https://github.com/StarRocks/starrocks/pull/1231)

## 1.19.3

リリース日：2021年11月30日

### 改善点

* セキュリティを向上させるためにjprotobufのバージョンをアップグレードしました。[# 1506](https://github.com/StarRocks/starrocks/issues/1506)

### バグ修正

* グループ化された結果の正確性に関するいくつかの問題を修正しました。
* grouping setsに関するいくつかの問題を修正しました。[# 1395](https://github.com/StarRocks/starrocks/issues/1395) [# 1119](https://github.com/StarRocks/starrocks/pull/1119)
* date_formatのいくつかの指標の問題を修正しました。
* 集約されたストリーミングの境界条件の問題を修正しました。[# 1584](https://github.com/StarRocks/starrocks/pull/1584)
* 詳細については、[リンク](https://github.com/StarRocks/starrocks/compare/1.19.2...1.19.3)を参照してください。

## 1.19.4

リリース日：2021年12月9日

### 改善点

* varcharをbitmapにキャストする機能をサポートします。[# 1941](https://github.com/StarRocks/starrocks/pull/1941)
* Hive外部テーブルのアクセスポリシーを更新します。[# 1394](https://github.com/StarRocks/starrocks/pull/1394) [# 1807](https://github.com/StarRocks/starrocks/pull/1807)

### バグ修正

* predicate Cross Joinの間違ったクエリ結果のバグを修正しました。[# 1918](https://github.com/StarRocks/starrocks/pull/1918)
* decimal型およびtime型の変換のバグを修正しました。[# 1709](https://github.com/StarRocks/starrocks/pull/1709) [# 1738](https://github.com/StarRocks/starrocks/pull/1738)
* colocate join/replicate joinの選択エラーのバグを修正しました。[# 1727](https://github.com/StarRocks/starrocks/pull/1727)
* いくつかの計画コスト計算の問題を修正しました。

## 1.19.5

リリース日：2021年12月20日

### 改善点

* シャッフルジョインの最適化計画を提供します。[# 2184](https://github.com/StarRocks/starrocks/pull/2184)
* 複数の大きなファイルのインポートを最適化します。[# 2067](https://github.com/StarRocks/starrocks/pull/2067)

### バグ修正

* Log4j2を2.17.0にアップグレードし、セキュリティの脆弱性を修正しました。[# 2284](https://github.com/StarRocks/starrocks/pull/2284) [# 2290](https://github.com/StarRocks/starrocks/pull/2290)
* Hive外部テーブルにおける空のパーティションの問題を修正しました。[# 707](https://github.com/StarRocks/starrocks/pull/707) [# 2082](https://github.com/StarRocks/starrocks/pull/2082)

## 1.19.7

リリース日：2022年3月18日

### バグ修正

以下のバグが修正されました：

* データ形式は異なるバージョンで異なる結果を生成します。[#4165](https://github.com/StarRocks/starrocks/pull/4165)
* データローディング中に誤ってParquetファイルが削除されたため、BEノードが失敗する可能性があります。[#3521](https://github.com/StarRocks/starrocks/pull/3521)
