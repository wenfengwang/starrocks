---
displayed_sidebar: "Japanese"
---

# Sparkコネクタ

## **通知**

**ユーザーガイド:**

- [Sparkコネクタを使用してStarRocksにデータをロード](../loading/Spark-connector-starrocks.md)
- [Sparkコネクタを使用してStarRocksからデータを読み込む](../unloading/Spark_connector.md)

**ソースコード**: [starrocks-connector-for-apache-spark](https://github.com/StarRocks/starrocks-connector-for-apache-spark)

**JARファイルの命名形式**: `starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

**JARファイルの取得方法:**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) から直接SparkコネクタJARファイルをダウンロードします。
- Mavenプロジェクトの `pom.xml` ファイルにSparkコネクタを依存関係として追加し、ダウンロードします。具体的な手順については、[ユーザーガイド](../loading/Spark-connector-starrocks.md#obtain-spark-connector) を参照してください。
- ソースコードをコンパイルして、SparkコネクタJARファイルを作成します。具体的な手順については、[ユーザーガイド](../loading/Spark-connector-starrocks.md#obtain-spark-connector) を参照してください。

**バージョン要件:**

| Sparkコネクタ | Spark            | StarRocks       | Java | Scala |
| --------------- | ---------------- | ---------------- | ---- | ----- |
| 1.1.1           | 3.2、3.3、または3.4 | 2.5以降        | 8    | 2.12  |
| 1.1.0           | 3.2、3.3、または3.4 | 2.5以降        | 8    | 2.12  |

## **リリースノート**

### 1.1

**1.1.1**

このリリースでは、主にStarRocksへのデータのロードに関する機能や改善が含まれています。

> **注意**
>
> Sparkコネクタをこのバージョンにアップグレードする際のいくつかの変更に注意してください。詳細については、[Sparkコネクタのアップグレード](../loading/Spark-connector-starrocks.md#upgrade-from-version-110-to-111) を参照してください。

**機能**

- シンクはリトライをサポートしています。[#61](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/61)
- BITMAPおよびHLL列へのデータのロードをサポートしています。[#67](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/67)
- ARRAY型データのロードをサポートしています。[#74](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/74)
- バッファされた行の数に応じてフラッシュをサポートしています。[#78](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/78)

**改善**

- 不要な依存関係を削除し、SparkコネクタJARファイルを軽量化しました。[#55](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/55) [#57](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/57)
- fastjsonをjacksonに置き換えました。[#58](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/58)
- 不足していたApacheライセンスヘッダーを追加しました。[#60](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/60)
- MySQL JDBCドライバーをSparkコネクタJARファイルにパッケージ化しないようにしました。[#63](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/63)
- タイムゾーンパラメータを構成し、Spark Java8 API datetimeと互換性があるようにしました。[#64](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/64)
- CPUコストを削減するために行文字列コンバータを最適化しました。[#68](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/68)
- `starrocks.fe.http.url`パラメータはhttpスキームを追加するようになりました。[#71](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/71)
- インタフェースBatchWrite#useCommitCoordinatorを実装し、DataBricks 13.1で実行できるようにしました。[#79](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/79)
- エラーログで権限とパラメータの確認ヒントを追加しました。[#81](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/81)

**バグ修正**

- CSV関連パラメータ `column_seperator` および `row_delimiter` でエスケープ文字を解析しました。[#85](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/85)

**ドキュメント**

- ドキュメントをリファクタリングしました。[#66](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/66)
- BITMAPおよびHLL列にデータをロードする例を追加しました。[#70](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/70)
- Pythonで記述されたSparkアプリケーションの例を追加しました。[#72](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/72)
- ARRAY型データのロードの例を追加しました。[#75](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/75)
- 主キーテーブルでの部分更新と条件付き更新の例を追加しました。[#80](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/80)

**1.1.0**

**機能**

- StarRocksにデータをロードすることをサポートしました。

### 1.0

**機能**

- StarRocksからデータをアンロードすることをサポートしました。