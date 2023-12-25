---
displayed_sidebar: Chinese
---

# Elasticsearchカタログ

StarRocksはバージョン3.1からElasticsearchカタログをサポートしています。

StarRocksとElasticsearchは、現在人気のある分析システムです。StarRocksは大規模分散計算を得意とし、外部テーブルを介してElasticsearchをクエリすることができます。Elasticsearchは全文検索に優れています。両者の組み合わせにより、より完全なOLAPソリューションを提供します。Elasticsearchカタログを使用すると、StarRocksを介してSQLでElasticsearchクラスタ内のすべてのインデックスデータを直接分析し、データ移行することなく利用できます。

他のデータソースのカタログとは異なり、Elasticsearchカタログを作成すると、`default`という名前のデータベース(Database)が1つだけ作成され、各Elasticsearchインデックス(Index)は自動的にデータテーブル(Table)にマッピングされ、すべて`default`データベースに自動的にマウントされます。

## Elasticsearchカタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ説明

#### `catalog_name`

Elasticsearchカタログの名前。命名規則は以下の通りです：

- 英字(a-zまたはA-Z)、数字(0-9)、アンダースコア(_)で構成され、英字で始まる必要があります。
- 全体の長さは1023文字を超えることはできません。
- カタログ名は大文字と小文字を区別します。

#### `comment`

Elasticsearchカタログの説明。このパラメータはオプションです。

#### PROPERTIES

Elasticsearchカタログのプロパティ。以下のプロパティがサポートされています：

| **パラメータ**       | **必須** | **デフォルト値** | **説明**                                                     |
| -------------------- | -------- | ---------------- | ------------------------------------------------------------ |
| hosts                | はい     | なし             | Elasticsearchクラスタへの接続アドレス。Elasticsearchのバージョン番号やインデックスのシャード分布情報を取得するために使用されます。1つ以上指定できます。StarRocksは`GET /_nodes/http` APIから返されるアドレスを使用してElasticsearchクラスタと通信するため、`hosts`パラメータの値は`GET /_nodes/http`から返されるアドレスと一致する必要があります。そうでない場合、BEがElasticsearchクラスタと正常に通信できない可能性があります。 |
| type                 | はい     | なし             | データソースのタイプ。Elasticsearchカタログを作成する際には、`es`と設定する必要があります。 |
| user                 | いいえ   | 空               | HTTP Basic認証を使用するElasticsearchクラスタのユーザー名。このユーザーが`/cluster/state/nodes/http`などのパスへのアクセス権とインデックスの読み取り権限を持っていることを確認する必要があります。 |
| password             | いいえ   | 空               | 対応するユーザーのパスワード情報。                           |
| es.type              | いいえ   | `_doc`           | インデックスのタイプを指定します。Elasticsearch 8以降のバージョンのデータをクエリする場合、StarRocksで外部テーブルを作成する際にこのパラメータを設定する必要はありません。Elasticsearch 8以降のバージョンではmapping typesが削除されています。 |
| es.nodes.wan.only    | いいえ   | `false`          | StarRocksが`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データを取得するかどうかを示します。バージョン2.3.0から、StarRocksはこのパラメータの設定をサポートしています。<ul><li>`true`：StarRocksは`hosts`で指定されたアドレスのみを使用してElasticsearchクラスタにアクセスし、データを取得します。Elasticsearchクラスタのインデックスの各シャードが存在するデータノードのアドレスを検出しません。StarRocksがElasticsearchクラスタ内のデータノードのアドレスにアクセスできない場合は、`true`に設定する必要があります。</li><li>`false`：StarRocksは`hosts`のアドレスを使用してElasticsearchクラスタのインデックスの各シャードが存在するデータノードのアドレスを検出します。クエリプランニング後、関連するBEノードはElasticsearchクラスタ内のデータノードに直接リクエストを送り、インデックスのシャードデータを取得します。StarRocksがElasticsearchクラスタ内のデータノードのアドレスにアクセスできる場合は、デフォルト値の`false`を維持することをお勧めします。</li></ul> |
| es.net.ssl           | いいえ   | `false`          | HTTPSプロトコルを使用してElasticsearchクラスタにアクセスすることを許可するかどうか。バージョン2.4から、StarRocksはこのパラメータの設定をサポートしています。<ul><li>`true`：許可します。HTTPプロトコルとHTTPSプロトコルの両方がアクセス可能です。</li><li>`false`：許可しません。HTTPプロトコルのみを使用してアクセスします。</li></ul> |
| enable_docvalue_scan | いいえ   | `true`           | Elasticsearchの列指向ストレージからクエリフィールドの値を取得するかどうか。ほとんどの場合、列指向ストレージからデータを読み取るパフォーマンスは、行指向ストレージから読み取るパフォーマンスよりも優れています。 |
| enable_keyword_sniff | いいえ   | `true`           | Elasticsearch内のTEXTタイプのフィールドを検出し、KEYWORDタイプのフィールドを使用してクエリを行うかどうか。`false`に設定すると、分割されたコンテンツに基づいてマッチングを行います。 |

### 作成例

以下のステートメントを実行して、`es_test`という名前のElasticsearchカタログを作成します：

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

## 谓词下推

StarRocksはElasticsearchテーブルに対する谓词下推をサポートしており、フィルタ条件をElasticsearchにプッシュして実行させ、ストレージに近いところで実行を行い、クエリパフォーマンスを向上させます。現在、下推がサポートされているオペレータは以下の表を参照してください：

| SQL構文              | Elasticsearch構文     |
| -------------------- | --------------------- |
| `=`                  | term query            |
| `in`                 | terms query           |
| `>=`, `<=`, `>`, `<` | range                 |
| `and`                | bool.filter           |
| `or`                 | bool.should           |
| `not`                | bool.must_not         |
| `not in`             | bool.must_not + terms |
| `esquery`            | ES Query DSL          |

## クエリ例

`esquery()`関数を使用して、SQLでは表現できないElasticsearchのクエリ（MatchクエリやGeoshapeクエリなど）をElasticsearchにプッシュしてフィルタリングを行います。`esquery()`の最初の列名パラメータはインデックスに関連付けられ、2番目のパラメータはElasticsearchの基本Query DSLのJSON表現で、中括弧(`{}`)で囲まれます。JSONのルートキー(Root Key)は1つだけでなければならず、`match`、`geo_shape`、`bool`などがあります。

- Matchクエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on elasticsearch"
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

- Booleanクエリ

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

## 注意事項

- Elasticsearch 5.x以前と以降のデータスキャン方法が異なり、現在StarRocksはElasticsearch 5.x以降のバージョンのみをサポートしています。
- HTTP Basic認証を使用するElasticsearchクラスタのクエリがサポートされています。
- StarRocksを介した一部のクエリは、直接Elasticsearchにリクエストするよりも遅くなることがあります。例えば`count()`関連のクエリです。これは、Elasticsearchが内部で条件に合致するドキュメントの数に関連するメタデータを直接読み取り、実際のデータに対するフィルタリング操作を行わないため、`count()`の速度が非常に速いためです。
