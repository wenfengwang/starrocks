---
displayed_sidebar: Chinese
---

# Spark connector のバージョンリリース

## リリースノート

**使用文書：**

- [Spark connector を使用して StarRocks へデータをインポートする](../loading/Spark-connector-starrocks.md)
- [Spark connector を使用して StarRocks からデータを読み取る](../unloading/Spark_connector.md)

**ソースコードダウンロード先：**[starrocks-connector-for-apache-spark](https://github.com/StarRocks/starrocks-connector-for-apache-spark)

**JAR ファイル命名規則：**`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

**JAR ファイルの取得方法:**

以下の方法で Spark connector の JAR ファイルを取得できます：

- [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) からコンパイル済みの JAR ファイルを直接ダウンロードします。
- Maven プロジェクトの pom ファイルに Spark connector を依存関係として追加し、依存関係としてダウンロードします。詳細は[使用文書](../loading/Spark-connector-starrocks.md)を参照してください。
- ソースコードから手動で JAR ファイルをコンパイルします。詳細は[使用文書](../loading/Spark-connector-starrocks.md)を参照してください。

**バージョン要件：**

| Connector | Spark         | StarRocks  | Java | Scala |
| --------- | ------------- | ---------- | ---- | ----- |
| 1.1.1     | 3.2, 3.3, 3.4 | 2.5 以上   | 8    | 2.12  |
| 1.1.0     | 3.2, 3.3, 3.4 | 2.5 以上   | 8    | 2.12  |

## リリース履歴

### 1.1

**1.1.1**

このバージョンのリリースには、以下の新機能と機能最適化が含まれており、StarRocks へのデータインポートに関連しています。

> **注意**
>
> このバージョンへのアップグレードは、動作の変更を伴います。詳細は[Spark connector のアップグレード](../loading/Spark-connector-starrocks.md#升级-spark-connector)を参照してください。

**新機能**

- Sink がリトライをサポート。[#61](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/61)
- BITMAP または HLL 型の列へのデータインポートをサポート。[#67](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/67)
- ARRAY 型のデータのインポートをサポート。[#74](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/74)
- バッファされたデータ行数に基づいてデータを flush する機能をサポート。[#78](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/78)

**機能最適化**

- 不要な依存関係を削除し、Spark connector JAR ファイルをより軽量に。[#55](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/55) [#57](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/57)
- fastjson を Jackson に置き換え。[#58](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/58)
- 不足していた Apache license header を追加。[#60](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/60)
- Spark connector JAR ファイルに MySQL JDBC ドライバーを含めないように変更。[#63](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/63)
- Spark Java 8 API datetime との互換性を確保。[#64](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/64)
- CPU コストを削減するために row-string コンバーターを最適化。[#68](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/68)
- starrocks.fe.http.url に HTTP スキームを追加するサポート。[#71](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/71)
- DataBricks v13.1 をサポートするために BatchWrite#useCommitCoordinator インターフェースを実装。 [#79](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/79)
- エラーログに権限とパラメータチェックのヒントを追加。[#81](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/81)

**問題修正**

- CSV 関連パラメータ `column_seperator` と `row_delimiter` のエスケープ文字を解析する問題を修正。[#85](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/85)

**ドキュメント**

- ドキュメントを再構築。[#66](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/66)
- BITMAP および HLL 型の列へのインポート方法に関する新しい例を追加。[#70](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/70)
- Python で書かれた Spark アプリケーションの新しい例を追加。[#72](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/72)
- ARRAY 型データのインポートに関する新しい例を追加。[#75](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/75)
- 主キーモデルテーブルの部分更新と条件更新を実現する方法に関する新しい例を追加。[#80](https://github.com/StarRocks/starrocks-connector-for-apache-spark/pull/80)

**1.1.0**

**新機能**

- StarRocks へのデータインポートをサポート。

### 1.0

**1.0.0**

**新機能**

- StarRocks からのデータ読み取りをサポート。
