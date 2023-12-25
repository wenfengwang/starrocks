---
displayed_sidebar: Chinese
---

# ファイルの作成

CREATE FILE ステートメントはファイルを作成するために使用されます。ファイルが作成されると、自動的にアップロードされ StarRocks クラスターに永続化されます。

:::tip

この操作には SYSTEM レベルの FILE 権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。ファイルがデータベースに属している場合、そのデータベースへのアクセス権を持つユーザーは誰でもそのファイルを使用できます。

:::

## 基本概念

**ファイル**：StarRocks に作成され保存されるファイルを指します。各ファイルにはグローバルに一意の識別子 (`FileId`) があります。ファイルはデータベース名 (`database`)、カテゴリ (`catalog`)、ファイル名 (`file_name`) の組み合わせで特定されます。

## 構文

```SQL
CREATE FILE "file_name" [IN database]
[properties]
```

## パラメータ説明

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| file_name      | はい     | ファイル名はニーズに応じてカスタマイズできます。             |
| database       | いいえ   | ファイルが属するデータベース。指定されていない場合は、現在のセッションのデータベースが使用されます。 |
| properties     | はい     | ファイルの属性で、具体的な設定項目は以下の `properties` 設定項目を参照してください。 |

**properties 設定項目**

| **設定項目** | **必須** | **説明**                                                     |
| ------------ | -------- | ------------------------------------------------------------ |
| url          | はい     | ファイルのダウンロードパス。現在は認証なしの HTTP ダウンロードパスのみをサポートしています。ステートメントが成功すると、ファイルはダウンロードされ StarRocks に永続化保存されます。 |
| catalog      | はい     | ファイルが属するカテゴリで、ニーズに応じてカスタマイズできます。特定のコマンドでは、指定されたカテゴリの下のファイルが検索されます。例えば、定期的なインポートでデータソースが StarRocks の場合、Apache Kafka® カテゴリの下のファイルが検索されます。 |
| md5          | いいえ   | メッセージダイジェストアルゴリズム。このパラメータが指定されている場合、StarRocks はダウンロードされたファイルを検証します。 |

## 例

- **test.pem** という名前のファイルを作成し、カテゴリは kafka です。

```SQL
CREATE FILE "test.pem"
PROPERTIES
(
    "url" = "http://starrocks-public.oss-cn-xxxx.aliyuncs.com/key/test.pem",
    "catalog" = "kafka"
);
```

- **client.key** という名前のファイルを作成し、カテゴリは my_catalog です。

```SQL
CREATE FILE "client.key"
IN my_database
PROPERTIES
(
    "url" = "http://test.bj.bcebos.com/kafka-key/client.key",
    "catalog" = "my_catalog",
);
```
