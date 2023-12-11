---
displayed_sidebar: "Japanese"
---

# Elasticsearch カタログ

StarRocks は v3.1 以降で Elasticsearch カタログをサポートしています。

StarRocks と Elasticsearch はどちらも個々の利点を持つ人気のある解析システムです。StarRocks は大規模な分散コンピューティングを得意とし、外部テーブルを介して Elasticsearch からデータをクエリすることができます。一方、Elasticsearch はその全文検索能力で知られています。StarRocks と Elasticsearch を組み合わせることで、より包括的な OLAP ソリューションが提供されます。Elasticsearch カタログを使用すると、データ移行の必要なく、StarRocks 上で SQL ステートメントを使用して Elasticsearch クラスタ内のすべてのインデックス化されたデータを直接分析することができます。

他のデータソースのカタログとは異なり、Elasticsearch カタログは作成時に `default` という名前のデータベースしか持ちません。各 Elasticsearch インデックスは自動的にテーブルにマッピングされ、`default` データベースにマウントされます。

## Elasticsearch カタログの作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ

#### `catalog_name`

Elasticsearch カタログの名前。以下の命名規則に従います:

- 名前は英字、数字 (0-9)、アンダースコア (_) を含めることができます。英字から始まる必要があります。
- 名前は大文字と小文字を区別し、長さが1023文字を超えてはいけません。

#### `comment`

Elasticsearch カタログの説明。このパラメータはオプションです。

#### PROPERTIES

Elasticsearch カタログのプロパティ。以下の表は Elasticsearch カタログでサポートされているプロパティを示しています。

| パラメータ             | 必須    | デフォルト値 | 説明                                                          |
| -------------------- | ------ | ---------- | ------------------------------------------------------------ |
| hosts                | はい    | None       | Elasticsearch クラスタの接続アドレス。1つまたは複数のアドレスを指定できます。StarRocks はこのアドレスから Elasticsearch のバージョンやインデックスのシャード割り当てを抽出することができます。StarRocks は `GET /_nodes/http` API 操作から返されたアドレスに基づいて Elasticsearch クラスタと通信します。そのため、`hosts` パラメータの値は `GET /_nodes/http` API 操作によって返されたアドレスと同じである必要があります。そうでない場合、BEs は Elasticsearch クラスタと通信できなくなる可能性があります。 |
| type                 | はい    | None       | データソースのタイプ。Elasticsearch カタログを作成する際にはこのパラメータを `es` に設定します。 |
| user                 | いいえ  | 空         | HTTP 基本認証を有効にした Elasticsearch クラスタにログインするためのユーザー名。`/cluster/state/nodes/http` などのパスへのアクセス権があることを確認してください。 |
| password             | いいえ  | 空         | Elasticsearch クラスタにログインするためのパスワード。 |
| es.type              | いいえ  | _doc       | インデックスのタイプ。Elasticsearch 8 以降のバージョンでデータをクエリしたい場合、このパラメータを構成する必要はありません。なぜならば Elasticsearch 8 以降のバージョンではマッピングタイプが削除されているためです。 |
| es.nodes.wan.only    | いいえ  | FALSE      | StarRocks が Elasticsearch クラスタにアクセスしデータをフェッチする際、`hosts` で指定されたアドレスのみを使用するかどうかを指定します。<ul><li>`true`: StarRocks は Elasticsearch クラスタ内のデータノードのアドレスにアクセスし、データをフェッチする際にシャードが存在するデータノードに対してデータをスニッフィングしません。StarRocks が Elasticsearch クラスタ内のデータノードのアドレスにアクセスできない場合、このパラメータを `true` に設定する必要があります。</li><li>`false`: StarRocks は Elasticsearch クラスタ内のデータノードのアドレスを使用して、インデックスのシャードからデータをフェッチするためにクエリ実行プランが生成された後、BEs は Elasticsearch クラスタ内のデータノードに直接アクセスします。StarRocks が Elasticsearch クラスタ内のデータノードのアドレスにアクセスできる場合、デフォルト値 `false` を維持することをお勧めします。</li></ul> |
| [es.net](http://es.net).ssl | いいえ  | FALSE      | HTTPS プロトコルを使用して Elasticsearch クラスタにアクセスできるかどうかを指定します。これは StarRocks v2.4 以降しかサポートしていません。<ul><li>`true`: HTTPS および HTTP プロトコルの両方を使用して Elasticsearch クラスタにアクセスできます。</li><li>`false`: HTTP プロトコルのみを使用して Elasticsearch クラスタにアクセスできます。</li></ul> |
| enable_docvalue_scan | いいえ  | TRUE       | Elasticsearch カラムストレージから対象フィールドの値を取得するかどうかを指定します。ほとんどの場合、カラムストレージからデータを読み取る方がロウストレージからデータを読み取るよりもパフォーマンスが優れています。 |
| enable_keyword_sniff | いいえ  | TRUE       | キーワード型フィールドをベースに TEXT 型フィールドをスニッフィングするかどうかを指定します。このパラメータを `false` に設定すると、StarRocks はトークン化後に一致を行います。 |

### 例

次の例は `es_test` という名前の Elasticsearch カタログを作成します:

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

StarRocks は、クエリで指定されたプレディケートを Elasticsearch テーブルに対して Elasticsearch にプッシュすることをサポートしています。これにより、クエリエンジンとストレージソースの間の距離を最小限に抑え、クエリのパフォーマンスが向上します。次の表に、Elasticsearch にプッシュダウンできる演算子をリストします。

| SQL 構文   | Elasticsearch 構文  |
| ---------- | ------------------- |
| `=`          | term クエリ            |
| `in`         | terms クエリ           |
| `>=, <=, >, <` | range                |
| `and`        | bool.filter          |
| `or`         | bool.should          |
| `not`        | bool.must_not        |
| `not in`     | bool.must_not +  terms |
| `esquery`    | ES クエリ DSL           |

## クエリの例

`esquery()` 関数を使用して、SQL で表現できない match や geoshape クエリなどの Elasticsearch クエリを Elasticsearch にプッシュしてフィルタリングおよび処理することができます。 `esquery()` 関数では、最初のパラメータで列名を指定し、2番目のパラメータでは Elasticsearch クエリの Elasticsearch クエリ DSL ベースの JSON 表現を波括弧 (`{}`) で囲んで使用します。JSON 表現は `match` や `geo_shape`、`bool` などのルートキーを 1 つだけ持つことができます。 

- Match クエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on elasticsearch"
     }
  }');
  ```

- Geoshape クエリ

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

- v5.x 以降では、Elasticsearch は異なるデータ走査方式を採用しています。StarRocks は Elasticsearch v5.x およびそれ以降のバージョンからのデータのクエリのみをサポートしています。
- StarRocks は HTTP 基本認証が有効になっている Elasticsearch クラスタからのデータのクエリのみをサポートします。
- `count()` を含む一部のクエリは、要求されたデータをフィルタリングする必要なく、クエリ条件を満たすドキュメントの数に関連するメタデータを直接読み取るため、Elasticsearch は StarRocks よりもはるかに遅く実行されます。