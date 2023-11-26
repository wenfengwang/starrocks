---
displayed_sidebar: "Japanese"
---

# Elasticsearch カタログ

StarRocks は v3.1 以降、Elasticsearch カタログをサポートしています。

StarRocks と Elasticsearch は、それぞれ独自の強みを持つ人気のある分析システムです。StarRocks は大規模な分散コンピューティングに優れており、外部テーブルを介して Elasticsearch のデータをクエリすることができます。Elasticsearch は全文検索の機能で知られています。StarRocks と Elasticsearch の組み合わせにより、より包括的な OLAP ソリューションが提供されます。Elasticsearch カタログを使用すると、データの移行を必要とせずに、StarRocks 上で SQL ステートメントを使用して Elasticsearch クラスタ内のすべてのインデックスデータを直接分析することができます。

他のデータソースのカタログとは異なり、Elasticsearch カタログには作成時に `default` という名前のデータベースが 1 つだけ含まれています。各 Elasticsearch インデックスは自動的にデータテーブルにマッピングされ、`default` データベースにマウントされます。

## Elasticsearch カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ

#### `catalog_name`

Elasticsearch カタログの名前です。命名規則は次のとおりです:

- 名前には、文字、数字 (0-9)、アンダースコア (_) を含めることができます。ただし、文字で始める必要があります。
- 名前は大文字と小文字を区別し、長さは 1023 文字を超えることはできません。

#### `comment`

Elasticsearch カタログの説明です。このパラメータはオプションです。

#### PROPERTIES

Elasticsearch カタログのプロパティです。次の表に、Elasticsearch カタログでサポートされるプロパティについて説明します。

| パラメータ                   | 必須     | デフォルト値 | 説明                                                         |
| --------------------------- | -------- | ------------- | ------------------------------------------------------------ |
| hosts                       | Yes      | None          | Elasticsearch クラスタの接続アドレスです。1 つまたは複数のアドレスを指定できます。StarRocks はこのアドレスから Elasticsearch のバージョンとインデックスのシャード割り当てを解析することができます。StarRocks は `GET /_nodes/http` API 操作によって返されるアドレスを基に Elasticsearch クラスタと通信します。そのため、`hosts` パラメータの値は `GET /_nodes/http` API 操作によって返されるアドレスと同じである必要があります。そうでない場合、BE は Elasticsearch クラスタと通信できない場合があります。 |
| type                        | Yes      | None          | データソースのタイプです。Elasticsearch カタログを作成する場合は、このパラメータを `es` に設定します。 |
| user                        | No       | Empty         | HTTP ベーシック認証が有効になっている Elasticsearch クラスタにログインするために使用されるユーザー名です。`/cluster/state/ nodes/http` などのパスにアクセスする権限があることを確認してください。また、インデックスを読み取る権限も必要です。 |
| password                    | No       | Empty         | Elasticsearch クラスタにログインするために使用されるパスワードです。 |
| es.type                     | No       | _doc          | インデックスのタイプです。Elasticsearch 8 以降のバージョンでデータをクエリする場合は、このパラメータを設定する必要はありません。なぜなら、Elasticsearch 8 以降のバージョンではマッピングタイプが削除されているためです。 |
| es.nodes.wan.only           | No       | FALSE         | StarRocks が Elasticsearch クラスタにアクセスしてデータを取得する際に、`hosts` で指定されたアドレスのみを使用するかどうかを指定します。<ul><li>`true`: StarRocks は Elasticsearch クラスタ内のデータノードのアドレスをスニッフしないで、`hosts` で指定されたアドレスのみを使用して Elasticsearch クラスタにアクセスしてデータを取得します。Elasticsearch クラスタ内のデータノードのアドレスにアクセスできない場合は、このパラメータを `true` に設定する必要があります。</li><li>`false`: StarRocks は Elasticsearch クラスタのインデックスのシャードが存在するデータノードをスニッフするために `hosts` で指定されたアドレスを使用します。StarRocks がクエリの実行計画を生成した後、BE は Elasticsearch クラスタ内のデータノードに直接アクセスしてインデックスのシャードからデータを取得します。Elasticsearch クラスタ内のデータノードのアドレスにアクセスできる場合は、デフォルト値 `false` を維持することをおすすめします。</li></ul> |
| [es.net](http://es.net).ssl | No       | FALSE         | HTTPS プロトコルを使用して Elasticsearch クラスタにアクセスできるかどうかを指定します。このパラメータの設定は、StarRocks v2.4 以降でのみサポートされています。<ul><li>`true`: HTTPS および HTTP の両方のプロトコルを使用して Elasticsearch クラスタにアクセスできます。</li><li>`false`: HTTP のみを使用して Elasticsearch クラスタにアクセスできます。</li></ul> |
| enable_docvalue_scan        | No       | TRUE          | Elasticsearch の列ストレージから対象フィールドの値を取得するかどうかを指定します。ほとんどの場合、列ストレージからデータを読み取る方が行ストレージからデータを読み取るよりもパフォーマンスが向上します。 |
| enable_keyword_sniff        | No       | TRUE          | KEYWORD タイプのフィールドに基づいて Elasticsearch の TEXT タイプのフィールドをスニッフするかどうかを指定します。このパラメータを `false` に設定すると、StarRocks はトークン化後にマッチングを実行します。 |

### 例

次の例では、`es_test` という名前の Elasticsearch カタログを作成します:

```SQL
CREATE EXTERNAL CATALOG es_test
COMMENT 'test123'
PROPERTIES
(
    "type" = "es",
    "es.type" = "_doc",
    "hosts" = "https://xxx:9200",
    "es.net.ssl" = "true",
    "user" = "admin",
    "password" = "xxx",
    "es.nodes.wan.only" = "true"
);
```

## プレディケートのプッシュダウン

StarRocks は、Elasticsearch テーブルに指定されたクエリのプレディケートを Elasticsearch にプッシュダウンして実行することをサポートしています。これにより、クエリエンジンとストレージソースの間の距離が短縮され、クエリのパフォーマンスが向上します。次の表に、Elasticsearch にプッシュダウンできる演算子を示します。

| SQL 構文   | Elasticsearch 構文  |
| ------------ | --------------------- |
| `=`            | term クエリ            |
| `in`           | terms クエリ           |
| `>=, <=, >, <` | range                 |
| `and`          | bool.filter           |
| `or`           | bool.should           |
| `not`          | bool.must_not         |
| `not in`       | bool.must_not + terms |
| `esquery`      | ES クエリ DSL          |

## クエリの例

`esquery()` 関数を使用して、SQL では表現できない Elasticsearch のマッチやジオシェイプクエリなどのクエリを Elasticsearch にプッシュダウンしてフィルタリングや処理を行うことができます。`esquery()` 関数では、最初のパラメータにはカラム名を指定し、2 番目のパラメータには Elasticsearch クエリの Elasticsearch Query DSL ベースの JSON 表現を中括弧 (`{}`) で囲んで指定します。JSON 表現は、`match`、`geo_shape`、`bool` などのルートキーを 1 つだけ持つことができ、かつ必要です。

- マッチクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on elasticsearch"
     }
  }');
  ```

- ジオシェイプクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
  "geo_shape": {
     "location": {
        "shape": {
           "type": "envelope",
           "coordinates": [
              [
                 13,
                 53
              ],
              [
                 14,
                 52
              ]
           ]
        },
        "relation": "within"
     }
  }
  }');
  ```

- ブールクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, ' {
     "bool": {
        "must": [
           {
              "terms": {
                 "k1": [
                    11,
                    12
                 ]
              }
           },
           {
              "terms": {
                 "k2": [
                    100
                 ]
              }
           }
        ]
     }
  }');
  ```

## 使用上の注意

- v5.x 以降、Elasticsearch は異なるデータスキャン方法を採用しています。StarRocks は Elasticsearch v5.x 以降のデータのクエリのみをサポートしています。
- StarRocks は、HTTP ベーシック認証が有効になっている Elasticsearch クラスタからのデータのクエリのみをサポートしています。
- `count()` を含む一部のクエリは、Elasticsearch では指定されたクエリ条件を満たすドキュメントの数に関連するメタデータを直接読み取るため、StarRocks よりも遅く実行される場合があります。
