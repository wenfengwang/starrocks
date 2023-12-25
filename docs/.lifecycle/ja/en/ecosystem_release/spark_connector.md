---
displayed_sidebar: English
---

# Spark コネクタ

## **通知**

**ユーザーガイド:**

- [Spark コネクタを使用して StarRocks にデータをロードする](../loading/Spark-connector-starrocks.md)
- [Spark コネクタを使用して StarRocks からデータを読み取る](../unloading/Spark_connector.md)

**ソースコード**: [starrocks-connector-for-apache-spark](https://github.com/StarRocks/starrocks-connector-for-apache-spark)

**JAR ファイルの命名形式**: `starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

**JAR ファイルの取得方法:**

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) から直接 Spark コネクタの JAR ファイルをダウンロードします。
- Maven プロジェクトの `pom.xml` ファイルに Spark コネクタを依存関係として追加し、ダウンロードします。具体的な手順については、[ユーザーガイドを参照してください](../loading/Spark-connector-starrocks.md#obtain-spark-connector)。
- ソースコードを Spark コネクタの JAR ファイルにコンパイルします。具体的な手順については、[ユーザーガイドを参照してください](../loading/Spark-connector-starrocks.md#obtain-spark-connector)。

**バージョン要件:**

| Spark コネクタ | Spark            | StarRocks     | Java | Scala |
| --------------- | ---------------- | ------------- | ---- | ----- |
| 1.1.1           | 3.2、3.3、または 3.4 | 2.5 以降 | 8    | 2.12  |
| 1.1.0           | 3.2、3.3、または 3.4 | 2.5 以降 | 8    | 2.12  |

## **リリースノート**

### 1.1

**1.1.1**

このリリースは、主に StarRocks へのデータロードに関する機能と改善を含んでいます。

> **注意**
>
> このバージョンへの Spark コネクタのアップグレード時の変更点に注意してください。詳細については、[Spark コネクタのアップグレード](../loading/Spark-connector-starrocks.md#upgrade-from-version-110-to-111)を参照してください。

**機能**

- シンクがリトライをサポートします。 [#61](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/61)
- BITMAP および HLL 列へのデータロードをサポートします。 [#67](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/67)
- ARRAY 型データのロードをサポートします。 [#74](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/74)
- バッファリングされた行数に基づいてフラッシュする機能をサポートします。 [#78](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/78)

**改善点**

- 不要な依存関係を削除し、Spark コネクタの JAR ファイルを軽量化します。 [#55](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/55) [#57](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/57)
- fastjson を jackson に置き換えます。 [#58](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/58)
- 不足している Apache ライセンスヘッダーを追加します。 [#60](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/60)
- MySQL JDBC ドライバーを Spark コネクタの JAR ファイルに含めないようにします。 [#63](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/63)
- タイムゾーン パラメーターを設定し、Spark Java8 API の日時との互換性を確保します。 [#64](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/64)
- 行文字列コンバーターを最適化して CPU コストを削減します。 [#68](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/68)
- `starrocks.fe.http.url` パラメーターが http スキームを追加することをサポートします。 [#71](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/71)
- インターフェース BatchWrite#useCommitCoordinator が DataBricks 13.1 で実行されるように実装されました。 [#79](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/79)
- エラーログに権限とパラメーターの確認ヒントを追加します。 [#81](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/81)

**バグ修正**

- CSV 関連パラメーター `column_seperator` と `row_delimiter` のエスケープ文字を解析します。 [#85](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/85)

**ドキュメント**

- ドキュメントをリファクタリングしました。 [#66](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/66)
- BITMAP 列と HLL 列へのデータロードの例を追加します。 [#70](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/70)
- Python で記述された Spark アプリケーションの例を追加します。 [#72](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/72)
- ARRAY 型データのロードの例を追加します。 [#75](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/75)
- 主キーテーブルで部分更新および条件付き更新を行う例を追加します。 [#80](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/80)

**1.1.0**

**機能**

- StarRocks へのデータロードをサポートします。

### 1.0

**機能**

- StarRocks からのデータアンロードをサポートします。
