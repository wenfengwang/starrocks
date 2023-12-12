---
displayed_sidebar: "Japanese"
---

# Elasticsearch カタログ

StarRocks は v3.1 以降で Elasticsearch カタログをサポートしています。

StarRocks と Elasticsearch は、それぞれ独自の強みを持つ人気のある解析システムです。StarRocks は大規模で分散した計算に優れ、外部テーブルを介して Elasticsearch からデータをクエリすることができます。Elasticsearch は全文検索の機能で知られています。StarRocks と Elasticsearch の組み合わせは、より包括的な OLAP ソリューションを提供します。Elasticsearch カタログを使用すると、データの移行を必要とせずに StarRocks 上で SQL ステートメントを使用して Elasticsearch クラスタ内のすべてのインデックス化されたデータを直接分析することができます。

他のデータソースのカタログとは異なり、Elasticsearch カタログには作成時に `default` という名前のデータベースのみが含まれます。各 Elasticsearch インデックスは自動的にデータテーブルにマップされ、`default` データベースにマウントされます。

## Elasticsearch カタログを作成

### 構文

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES ("key"="value", ...)
```

### パラメータ

#### `catalog_name`

Elasticsearch カタログの名前。以下の名前の規則に従います。

- 名前には、文字、数字 (0-9)、アンダースコア (_) を含めることができます。文字で始める必要があります。
- 名前は大文字と小文字を区別し、1023 文字を超えることはできません。

#### `comment`

Elasticsearch カタログの説明。このパラメータはオプションです。

#### PROPERTIES

Elasticsearch カタログのプロパティ。以下の表には Elasticsearch カタログでサポートされているプロパティについて説明されています。

| パラメータ                     | 必須     | デフォルト値 | 説明                                                   |
| --------------------------- | -------- | ------------- | ------------------------------------------------------------ |
| hosts                       | Yes      | None          | Elasticsearch クラスタの接続アドレスです。1 つ以上のアドレスを指定できます。StarRocks はこのアドレスから Elasticsearch のバージョンやインデックスのシャード割り当てを解析することができます。StarRocks は `GET /_nodes/http` API 操作で返されたアドレスに基づいて Elasticsearch クラスタと通信します。そのため、`hosts` パラメータの値は `GET /_nodes/http` API 操作で返されたアドレスと同じである必要があります。さもないと、BE (バックエンド プロセス) は Elasticsearch クラスタと通信できない場合があります。 |
| type                        | Yes      | None          | データソースの種類です。Elasticsearch カタログを作成するときにこのパラメータを `es` に設定します。 |
| user                        | No       | Empty         | HTTP ベーシック認証が有効になっている Elasticsearch クラスタにログインするために使用するユーザー名です。`/cluster/state/ nodes/http` などへのアクセス権があることを確認してください。 |
| password                    | No       | Empty         | Elasticsearch クラスタにログインするために使用するパスワードです。 |
| es.type                     | No       | _doc          | インデックスの種類です。Elasticsearch 8 以降ではマッピングタイプが削除されているため、Elasticsearch 8 以降のバージョンでデータをクエリする場合にはこのパラメータを構成する必要はありません。 |
| es.nodes.wan.only           | No       | FALSE         | StarRocks が `hosts` で指定したアドレスのみを使用して Elasticsearch クラスタにアクセスしデータを取得するかを指定します。<ul><li>`true`: StarRocks は Elasticsearch クラスタのデータノードのアドレスにアクセスし、その上で Elasticsearch インデックスのシャードが存在するデータノードをプロットしません。StarRocks が Elasticsearch クラスタのデータノードのアドレスにアクセスできない場合は、このパラメータを `true` に設定する必要があります。</li><li>`false`: StarRocks は Elasticsearch クラスタ内のデータノードのアドレスを探し、その後 StarRocks がクエリ実行プランを生成した後、BE が直接インデックスのシャードからデータを取得するため、デフォルト値 `false` を維持することをお勧めします。</li></ul> |
| [es.net](http://es.net).ssl | No       | FALSE         | HTTPS プロトコルを使用して Elasticsearch クラスタにアクセスするかを指定します。StarRocks v2.4 以降のみがこのパラメータの構成をサポートしています。<ul><li>`true`: HTTPS プロトコルおよび HTTP プロトコルの両方を使用して Elasticsearch クラスタにアクセスできます。</li><li>`false`: HTTP プロトコルのみで Elasticsearch クラスタにアクセスできます。</li></ul> |
| enable_docvalue_scan        | No       | TRUE          | Elasticsearch のカラム ストレージから対象フィールドの値を取得するかどうかを指定します。大抵の場合、カラム ストレージからデータを読むほうがロウ ストレージから読むよりもパフォーマンスが向上します。 |
| enable_keyword_sniff        | No       | TRUE          | KEYWORD タイプのフィールドに基づいて Elasticsearch の TEXT タイプのフィールドを嗅ぐかどうかを指定します。このパラメータを `false` に設定すると、StarRocks はトークン化の後にマッチングを実行します。 |

### 例

次の例では、`es_test` という名前の Elasticsearch カタログを作成します。

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

## プレディケート プッシュダウン

StarRocks は、クエリで指定された述語を Elasticsearch テーブルに対して Elasticsearch にダウンプッシュして実行できます。これにより、クエリエンジンとストレージ ソースとの間の距離を短くし、クエリ パフォーマンスを向上させます。次の表に、Elasticsearch にプッシュダウンできる演算子を示します。

| SQL 構文   | Elasticsearch 構文  |
| ------------ | --------------------- |
| `=`            | term query            |
| `in`           | terms query           |
| `>=, <=, >, <` | range                 |
| `and`          | bool.filter           |
| `or`           | bool.should           |
| `not`          | bool.must_not         |
| `not in`       | bool.must_not + terms |
| `esquery`      | ES Query DSL          |

## クエリの例

`esquery()` 関数を使用して、SQL で表現できないマッチやジオシェイプ クエリなどの Elasticsearch クエリを Elasticsearch にダウンプッシュしてフィルタリングや処理を行うことができます。`esquery()` 関数では、カラム名を指定する最初のパラメータがインデックスと関連付けられ、2 番目のパラメータは波括弧 (`{}`) で囲まれた Elasticsearch クエリの Elasticsearch Query DSL ベースの JSON 表現です。JSON 表現には `match`、`geo_shape`、`bool` などのルートキーが 1 つだけ含まれている必要があります。

- マッチ クエリ

  ```SQL
  SELECT * FROM es_table WHERE esquery(k4, '{
     "match": {
        "k4": "StarRocks on elasticsearch"
     }
  }');
  ```

- ジオシェイプ クエリ

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

- ブール クエリ

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

- Elasticsearch は v5.x 以降で異なるデータ走査方法を採用しています。StarRocks は Elasticsearch v5.x 以降からのデータクエリのみをサポートしています。
- StarRocks は、HTTP ベーシック認証が有効になっている Elasticsearch クラスタからのデータクエリのみをサポートしています。
- `count()` を含む一部のクエリは、条件を満たすドキュメントの数に関連するメタデータをフィルタリングする必要がないため、Elasticsearch が直接関連するデータを読むことができるため、StarRocks で実行されると Elasticsearch よりもはるかに遅くなります。