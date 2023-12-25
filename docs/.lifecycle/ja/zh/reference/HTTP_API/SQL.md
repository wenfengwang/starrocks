---
displayed_sidebar: Chinese
---

# HTTP SQL API

## 機能

StarRocks 3.2.0 バージョンは HTTP SQL API を提供し、HTTP プロトコルを通じて StarRocks のクエリ機能を利用できるようになりました。現在、SELECT、SHOW、EXPLAIN、KILL 文がサポートされています。

curl コマンドの使用例は以下の通りです：

```shell
curl -X POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql' \
   -u '<username>:<password>' -d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}' \
   --header "Content-Type: application/json"
```

## リクエストメッセージ

### リクエストライン

```shell
POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql'
```

| フィールド               | 説明                                                           |
| ------------------------ | :----------------------------------------------------------- |
|  fe_ip                   | FEノードのIP。                                                 |
|  fe_http_port            | FEノードのHTTPポート。                                         |
|  catalog_name            | データカタログ名。現在は StarRocks の内部テーブルクエリのみ対応しており、`<catalog_name>` は `default_catalog` のみをサポートしています。|
|  database_name           | データベース名。データベース名が指定されていない場合は、SQLクエリ内のテーブル名の前にデータベース名を付ける必要があります。例：`database_name.table_name`。 |

- catalog を指定し、database を跨いでクエリする場合。SQL文内のテーブル名の前にデータベース名を付ける必要があります。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/sql
   ```

- catalog と database を指定してクエリする場合。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/databases/<database_name>/sql
   ```

### 認証方法

```SQL
Authorization: Basic <credentials>
```

Basic認証を使用して認証します。つまり、`credentials` にはユーザー名とパスワード (`-u '<username>:<password>'`) を記入します。アカウントにパスワードが設定されていない場合は、ユーザー名のみを入力し、パスワードは空にします。例えば、root アカウントにパスワードが設定されていない場合は `-u 'root:'` と記述します。

### リクエストボディ

```shell
-d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}'
```

| フィールド               | 説明                                                           |
| ------------------------ | :----------------------------------------------------------- |
| query                    | SQL文。STRING形式。現在、SELECT、SHOW、EXPLAIN、KILL 文がサポートされています。HTTPリクエストでは1回に1つのSQL文のみ実行できます。 |
| sessionVariables         | 指定されたセッション変数。JSON形式。オプションで、デフォルトは空です。設定されたセッション変数は同一接続内で常に有効で、接続が切断されると無効になります。 |

### リクエストヘッダー

```shell
--header "Content-Type: application/json"
```

リクエストボディが JSON 文字列であることを示します。

## レスポンスメッセージ

### ステータスコード

- 200：HTTPリクエストが成功し、データをクライアントに送信する前にサーバーで例外が発生していません。
- 4xx：HTTPリクエストエラーで、クライアント側のエラーを示します。
- `500 Internal Server Error`：HTTPリクエストは成功しましたが、データをクライアントに送信する前に例外が発生しました。
- 503：HTTPリクエストは成功しましたが、FEは現在サービスを提供できません。

### レスポンスヘッダー

content-type はレスポンスボディの形式を示します。ここでは Newline delimited JSON 形式を使用しており、レスポンスボディは複数の JSON オブジェクトで構成され、JSON オブジェクト間は `\n` で区切られています。

|                      | 説明                                                           |
| -------------------- | :----------------------------------------------------------- |
| content-type         | Newline delimited JSON 形式です。デフォルトは "application/x-ndjson; charset=UTF-8" です。 |
| X-StarRocks-Query-Id | クエリID。                                                     |

### レスポンスボディ

#### 結果を送信する前に実行が失敗した場合

クライアントのリクエストがエラーであるか、サーバーがデータをクライアントに送信する前に例外が発生した場合、レスポンスボディの形式は以下の通りです。ここで `msg` は対応するエラーメッセージです。

```json
{
   "status":"FAILED",
   "msg":"xxx"
}
```

#### 結果の一部を送信した後に実行が失敗した場合

この場合、結果の一部がすでに送信されており、HTTPステータスコードは200です。したがって、データの送信を続けずに接続を閉じ、エラーログを記録します。

#### 実行が成功した場合

返される結果は各行が JSON オブジェクトで、JSON オブジェクト間は改行文字 `\n` で区切られています。これにより、クライアントは StarRocks から送信されるデータを行単位で解析でき、すべてのデータが送信されるまで現在の結果をキャッシュせずに解析を行うことができます。これにより、クライアントのメモリ消費を低減できます。

- SELECT 文に対しては、以下の JSON オブジェクトが返されます。

| オブジェクト       | 説明                                                           |
| ------------ | :----------------------------------------------------------- |
| `connectionId` | 接続に対応するIDで、KILL `<connectionId>` を使用して実行時間が長いクエリをキャンセルできます。 |
| `meta`        | 各列を記述します。キーは meta、値は JSON 配列です。配列内の各オブジェクトは一つのカラムを表します。 |
| `data`         | 一行のデータを表します。キーは data、値は JSON 配列で、配列には一つのデータが含まれます。 |
| `statistics`   | この実行の統計情報です。                                       |

- SHOW に対しては、`meta`、`data`、`statistics` が返されます。
- EXPLAIN に対しては、クエリの詳細な実行計画を示す `explain` オブジェクトが返されます。

例：ここでの改行は `\n` で表されます。送信時に StarRocks は HTTP chunked 送信方式を使用し、FE が一定量のデータを取得するたびに、そのデータをクライアントにストリーミング転送します。クライアントは StarRocks から送信されるデータを行単位で解析でき、すべてのデータが送信されるまで現在の結果をキャッシュせずに解析を行うことができます。これにより、クライアントのメモリ消費を低減できます。

```json
{"connectionId": 7}\n
{"meta": [
    {
      "name": "stock_symbol",
      "type": "varchar"
    },
    {
      "name": "closing_price",
      "type": "decimal64(8, 2)"
    },
    {
      "name": "closing_date",
      "type": "datetime"
    }
  ]}\n
{"data": ["JDR", 12.86, "2014-10-02 00:00:00"]}\n
{"data": ["JDR",14.8, "2014-10-10 00:00:00"]}\n
...
{"statistics": {"scanRows": 0,"scanBytes": 0,"returnRows": 9}}
```

## API呼び出し例

### クエリの実行

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "select * from agg;"}' --header "Content-Type: application/json"
```

返される結果：

```json
{"connectionId":49}
{"meta":[{"name":"no","type":"int(11)"},{"name":"k","type":"decimal64(10, 2)"},{"name":"v","type":"decimal64(10, 2)"}]}
{"data":[1,"10.00",null]}
{"data":[2,"10.00","11.00"]}
{"data":[2,"20.00","22.00"]}
{"data":[2,"25.00",null]}
{"data":[2,"30.00","35.00"]}
{"statistics":{"scanRows":0,"scanBytes":0,"returnRows":5}}
```

### クエリのキャンセル

実行時間が長すぎるクエリをキャンセルしたい場合は、接続を直接切断することができます。接続が切断されたことを検出すると、StarRocks は自動的にそのクエリをキャンセルします。

また、KILL `<connectionId>` を送信してクエリをキャンセルすることもできます。例えば：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "kill 17;"}' --header "Content-Type: application/json"
```

`connectionId` はレスポンスボディで返されるほか、SHOW PROCESSLIST コマンドを使用して取得することもできます。例えば：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "show processlist;"}' --header "Content-Type: application/json"
```

### クエリ実行時にセッション変数を設定

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "SHOW VARIABLES;", "sessionVariables":{"broadcast_row_limit":14000000}}' --header "Content-Type: application/json"
```
