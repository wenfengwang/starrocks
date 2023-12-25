---
displayed_sidebar: English
---

# Elasticsearch カタログ

StarRocksは、v3.1以降のElasticsearchカタログをサポートしています。

StarRocksとElasticsearchは、それぞれ独自の強みを持つ人気のある分析システムです。StarRocksは大規模な分散コンピューティングに優れ、外部テーブルを介してElasticsearchのデータをクエリすることをサポートしています。Elasticsearchは、全文検索機能で知られています。StarRocksとElasticsearchの組み合わせにより、より包括的なOLAPソリューションを提供します。Elasticsearchカタログを使用すると、データ移行の必要なく、StarRocksのSQLステートメントを使用してElasticsearchクラスター内のすべてのインデックス付きデータを直接分析できます。

他のデータソースのカタログとは異なり、Elasticsearchカタログには作成時に`default`という名前のデータベースが1つだけ含まれています。各Elasticsearchインデックスは自動的にデータテーブルにマッピングされ、`default`データベースにマウントされます。

## Elasticsearch カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメーター

#### `catalog_name`

Elasticsearchカタログの名前。命名規則は以下の通りです：

- 名前には、文字、数字（0-9）、およびアンダースコア（_）を含めることができます。文字で始まる必要があります。
- 名前は大文字と小文字を区別し、長さは1023文字を超えることはできません。

#### `comment`

Elasticsearchカタログの説明。このパラメーターはオプションです。

#### PROPERTIES

Elasticsearchカタログのプロパティ。次の表は、Elasticsearchカタログでサポートされるプロパティを説明しています。

| パラメーター                   | 必須 | 既定値 | 説明                                                  |
| --------------------------- | ---- | ----- | ---------------------------------------------------- |
| hosts                       | はい | なし  | Elasticsearchクラスターの接続アドレス。1つ以上のアドレスを指定できます。StarRocksはこのアドレスからElasticsearchのバージョンとインデックスシャードの割り当てを解析できます。StarRocksは`GET /_nodes/http` API操作によって返されるアドレスに基づいてElasticsearchクラスターと通信します。したがって、`hosts`パラメーターの値は`GET /_nodes/http` API操作によって返されるアドレスと同じでなければなりません。そうでない場合、BEはElasticsearchクラスターと通信できない可能性があります。 |
| type                        | はい | なし  | データソースのタイプ。Elasticsearchカタログを作成するときは、このパラメーターを`es`に設定します。 |
| user                        | いいえ | 空   | HTTP基本認証が有効なElasticsearchクラスターにログインするために使用するユーザー名。`/cluster/state/nodes/http`などのパスへのアクセス権とインデックスの読み取り権限が必要です。 |
| password                    | いいえ | 空   | Elasticsearchクラスターにログインするために使用するパスワード。 |
| es.type                     | いいえ | _doc | インデックスのタイプ。Elasticsearch 8以降のバージョンでデータをクエリする場合、このパラメーターを設定する必要はありません。Elasticsearch 8以降でマッピングタイプが削除されているためです。 |
| es.nodes.wan.only           | いいえ | FALSE | StarRocksが`hosts`で指定されたアドレスのみを使用してElasticsearchクラスターにアクセスしデータを取得するかどうかを指定します。<ul><li>`true`: StarRocksは`hosts`で指定されたアドレスのみを使用してElasticsearchクラスターにアクセスしデータを取得し、Elasticsearchインデックスのシャードが存在するデータノードのスニッフィングは行いません。StarRocksがElasticsearchクラスター内のデータノードのアドレスにアクセスできない場合、このパラメーターを`true`に設定する必要があります。</li><li>`false`: StarRocksは`hosts`で指定されたアドレスを使用してElasticsearchクラスターインデックスのシャードが存在するデータノードをスニッフィングします。StarRocksがクエリ実行プランを生成した後、BEはElasticsearchクラスター内のデータノードに直接アクセスし、インデックスのシャードからデータを取得します。StarRocksがElasticsearchクラスター内のデータノードのアドレスにアクセスできる場合、デフォルト値`false`を保持することをお勧めします。</li></ul> |
| es.net.ssl                  | いいえ | FALSE | HTTPSプロトコルを使用してElasticsearchクラスターにアクセスできるかどうかを指定します。StarRocks v2.4以降のみがこのパラメーターの設定をサポートしています。<ul><li>`true`: HTTPSプロトコルとHTTPプロトコルの両方を使用してElasticsearchクラスターにアクセスできます。</li><li>`false`: HTTPプロトコルのみを使用してElasticsearchクラスターにアクセスできます。</li></ul> |
| enable_docvalue_scan        | いいえ | TRUE  | ターゲットフィールドの値をElasticsearchの列指向ストレージから取得するかどうかを指定します。ほとんどの場合、列指向ストレージからのデータ読み取りは行指向ストレージからの読み取りよりもパフォーマンスが優れています。 |
| enable_keyword_sniff        | いいえ | TRUE  | ElasticsearchのTEXTタイプのフィールドをKEYWORDタイプのフィールドに基づいてスニッフィングするかどうかを指定します。このパラメーターを`false`に設定すると、StarRocksはトークン化後の照合を行います。 |

### 例

次の例では、`es_test`という名前のElasticsearchカタログを作成します：

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

## 述語プッシュダウン

StarRocksは、Elasticsearchテーブルに対するクエリで指定された述語をElasticsearchにプッシュして実行することをサポートしています。これにより、クエリエンジンとストレージソース間の距離が最小限に抑えられ、クエリのパフォーマンスが向上します。次の表は、Elasticsearchにプッシュダウンできる演算子を一覧表示しています。

| SQL 構文   | Elasticsearch 構文  |
| ---------- | ------------------- |
| `=`        | term query          |
| `in`       | terms query         |
| `>=, <=, >, <` | range               |
| `and`      | bool.filter         |
| `or`       | bool.should         |
| `not`      | bool.must_not       |
| `not in`   | bool.must_not + terms |
| `esquery`  | ES Query DSL        |

## クエリ例

`esquery()`関数を使用して、SQLで表現できないmatchクエリやgeoshapeクエリなどのElasticsearchクエリをElasticsearchにプッシュしてフィルタリングと処理を行うことができます。`esquery()`関数では、インデックスとの関連付けに使用される列名を指定する最初のパラメーターと、中括弧`{}`で囲まれたElasticsearchクエリのElasticsearch Query DSLベースのJSON表現を指定する2番目のパラメーターがあります。JSON表現には`match`、`geo_shape`、`bool`など、1つのルートキーのみを持つことができます。

- Matchクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on Elasticsearch"
     }
  }');
  ```

- Geoshapeクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
  "geo_shape": {
     "location": {
        "shape": {
           "type": "envelope",
           "coordinates": [
              [13, 53],
              [14, 52]
           ]
        },
        "relation": "within"
     }
  }
  }');
  ```

- Booleanクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "bool": {
        "must": [
           {
              "terms": {
                 "k1": [11, 12]
              }
           },
           {
              "terms": {
                 "k2": [100]
              }
           }
        ]
     }
  }');
  ```

## 使用上の注意

- v5.x以降、Elasticsearchは異なるデータスキャン方法を採用しています。StarRocksは、Elasticsearch v5.x以降のデータのみをクエリすることをサポートしています。
- StarRocksは、HTTP基本認証が有効になっているElasticsearchクラスターからのデータのみをクエリすることをサポートしています。
- 一部のクエリ、例えば`count()`を含むクエリは、Elasticsearchで直接指定されたドキュメントのメタデータを読み取ることができるため、StarRocksでの実行がはるかに遅くなることがあります。
