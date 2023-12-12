---
displayed_sidebar: "Japanese"
---

# Sparkコネクタ

## **通知**

**ユーザガイド:**

- [Sparkコネクタを使用してStarRocksにデータをロード](../loading/Spark-connector-starrocks-jp.md)
- [Sparkコネクタを使用してStarRocksからデータを読み込む](../unloading/Spark_connector-jp.md)

**ソースコード**: [starrocks-connector-for-apache-spark](https://github.com/StarRocks/starrocks-connector-for-apache-spark)

**JARファイルの命名形式**: `starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

**JARファイルを取得する方法:**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) から直接SparkコネクタのJARファイルをダウンロード。
- Mavenプロジェクトの`pom.xml`ファイルにSparkコネクタを依存関係として追加し、ダウンロードする。詳細な手順については、[ユーザガイド](../loading/Spark-connector-starrocks-jp.md#obtain-spark-connector) を参照。
- ソースコードをコンパイルしてSparkコネクタのJARファイルを取得する。詳細な手順については、[ユーザガイド](../loading/Spark-connector-starrocks-jp.md#obtain-spark-connector) を参照。

**バージョン要件:**

| Sparkコネクタ | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1           | 3.2, 3.3, または 3.4 | 2.5以降 | 8    | 2.12  |
| 1.1.0           | 3.2, 3.3, または 3.4 | 2.5以降 | 8    | 2.12  |

## **リリースノート**

### 1.1

**1.1.1**

このリリースでは主にStarRocksへのデータロードに関する機能と改善が含まれています。

> **注意**
>
> Sparkコネクタをこのバージョンにアップグレードする際の一部の変更に注意してください。詳細については、[Sparkコネクタのアップグレード](../loading/Spark-connector-starrocks-jp.md#upgrade-from-version-110-to-111) を参照してください。

**機能**

- シンクがリトライをサポート。[#61](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/61)
- BITMAPとHLL列へのデータロードのサポート。[#67](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/67)
- ARRAYタイプのデータのロードのサポート。[#74](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/74)
- バッファされた行数に応じてフラッシュのサポート。[#78](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/78)

**改善**

- 不要な依存関係を削除し、SparkコネクタのJARファイルを軽量化。[#55](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/55) [#57](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/57)
- fastjsonをjacksonに置換。[#58](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/58)
- 不足していたApacheライセンスヘッダーを追加。[#60](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/60)
- SparkコネクタのJARファイルにMySQL JDBCドライバをパッケージしないようにする。[#63](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/63)
- タイムゾーンパラメータを設定し、Spark Java8 APIの日時と互換性があるようにサポート。[#64](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/64)
- CPUコストを削減するために行文字列コンバーターを最適化。[#68](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/68)
- `starrocks.fe.http.url`パラメータがhttpスキームを追加できるようにサポート。[#71](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/71)
- DataBricks 13.1で実行するために、インターフェースBatchWrite#useCommitCoordinatorを実装。[#79](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/79)
- エラーログで権限とパラメータを確認するヒントを追加。[#81](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/81)

**バグ修正**

- CSV関連のパラメータ `column_seperator` および `row_delimiter` でエスケープ文字を解析。[#85](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/85)

**ドキュメント**

- ドキュメントを再構築。[#66](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/66)
- BITMAPおよびHLL列へのデータロードの例を追加。[#70](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/70)
- Pythonで書かれたSparkアプリケーションの例を追加。[#72](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/72)
- ARRAYタイプデータのロードの例を追加。[#75](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/75)
- 主キーテーブルでの部分更新と条件付き更新の例を追加。[#80](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/80)

**1.1.0**

**機能**

- StarRocksへのデータロードをサポート。

### 1.0

**機能**

- StarRocksからデータをアンロードするサポート。