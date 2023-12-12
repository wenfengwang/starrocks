```
---
displayed_sidebar: "Japanese"
---

# ファイルの作成

CREATE FILEステートメントを実行してファイルを作成できます。ファイルが作成された後は、ファイルがStarRocksにアップロードされ、永続化されます。データベースでは、管理者ユーザーのみがファイルを作成および削除でき、データベースへのアクセス権があるすべてのユーザーがデータベースに属するファイルを使用できます。

## 基本的な概念

**ファイル**: StarRocksに作成および保存されたファイルを指します。ファイルがStarRocksに作成および保存されると、StarRocksはそのファイルに一意のIDを割り当てます。データベース名、カタログ、ファイル名に基づいてファイルを見つけることができます。

## 文法

```SQL
CREATE FILE "file_name" [IN database]
[properties]
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                    |
| -------------- | -------- | ---------------------------------------------------------- |
| file_name      | Yes      | ファイルの名前。                                             |
| database       | No       | ファイルが属するデータベース。このパラメータを指定しない場合、現在のセッションでアクセスするデータベースの名前がデフォルト値となります。 |
| properties     | Yes      | ファイルのプロパティ。以下の表に、プロパティの構成項目が記載されています。 |

**`properties`** **の構成項目**

| **構成項目** | **必須** | **説明**                                                    |
| ------------ | -------- | ---------------------------------------------------------- |
| url          | Yes      | ファイルをダウンロードできるURL。認証されていないHTTP URLのみがサポートされます。ファイルがStarRocksに保存された後は、URLは必要ありません。 |
| catalog      | Yes      | ファイルが属するカテゴリ。ビジネス要件に基づいてカタログを指定できますが、一部の状況ではこのパラメータを特定のカタログに設定する必要があります。たとえば、Kafkaからデータを読み込む場合、StarRocksはKafkaデータソースにあるカタログを検索します。 |
| MD5          | No       | ファイルをチェックするために使用されるメッセージダイジェストアルゴリズム。このパラメータを指定すると、ファイルがダウンロードされた後にStarRocksがファイルをチェックします。 |

## 例

- カテゴリがkafkaである**test.pem**という名前のファイルを作成します。

```SQL
CREATE FILE "test.pem"
PROPERTIES
(
    "url" = "https://starrocks-public.oss-cn-xxxx.aliyuncs.com/key/test.pem",
    "catalog" = "kafka"
);
```

- カテゴリがmy_catalogである**client.key**という名前のファイルを作成します。

```SQL
CREATE FILE "client.key"
IN my_database
PROPERTIES
(
    "url" = "http://test.bj.bcebos.com/kafka-key/client.key",
    "catalog" = "my_catalog",
    "md5" = "b5bb901bf10f99205b39a46ac3557dd9"
);
```
