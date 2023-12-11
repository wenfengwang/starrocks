---
displayed_sidebar: "Japanese"
---

# HTTP SQL API（HTTP SQL API）

## 説明

StarRocks v3.2.0 では、HTTP SQL API が導入され、ユーザーは HTTP を使用してさまざまな種類のクエリを実行できます。現在、このAPIはSELECT、SHOW、EXPLAIN、およびKILLステートメントをサポートしています。

curlコマンドを使用した構文:

```shell
curl -X POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql' \
   -u '<username>:<password>'  -d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}' \
   --header "Content-Type: application/json"
```

## リクエストメッセージ

### リクエストライン

```shell
POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql'
```

| フィールド         | 説明                                                  |
| ---------------- | :----------------------------------------------------------- |
|  fe_ip           | FEノードのIPアドレス。                                                |
|  fe_http_port    | FE HTTPポート。                                           |
|  catalog_name    | カタログ名。現在、このAPIは内部テーブルのクエリのみをサポートしているため、「<catalog_name>」は「default_catalog」にのみ設定できます。 |
|  database_name   | データベース名。リクエストラインでデータベース名が指定されていない場合、SQLクエリでテーブル名が使用される場合は、テーブル名をそのデータベース名で接頭辞として付ける必要があります。たとえば、`database_name.table_name`。

- 指定されたカタログ内のデータベースを横断してデータをクエリします。SQLクエリでテーブルが使用される場合、テーブル名をそのデータベース名で接頭辞として付ける必要があります。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/sql
   ```

- 指定されたカタログおよびデータベースからデータをクエリします。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/databases/<database_name>/sql
   ```

### 認証方法

```shell
Authorization: Basic <credentials>
```

基本認証を使用し、`<credentials>` にユーザー名とパスワードを入力します（`-u '<username>:<password>'`）。ユーザー名に対してパスワードが設定されていない場合は、`<username>:` のみを入力し、パスワードを空白のままにします。たとえば、ルートアカウントを使用する場合は、`-u 'root:'` と入力します。

### リクエストボディ

```shell
-d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}'
```

| フィールド         | 説明                                                  |
| ---------------- | :----------------------------------------------------------- |
| query            | SQLクエリ（STRING形式）。SELECT、SHOW、EXPLAIN、KILLステートメントのみをサポートしています。1つのHTTPリクエストにつき1つのSQLクエリのみを実行できます。 |
| sessionVariables | JSON形式でクエリに設定したい [セッション変数](../System_variable.md)。このフィールドはオプションです。デフォルトは空です。設定したセッション変数は同じ接続に対して有効であり、接続が閉じられると無効になります。 |

### リクエストヘッダー

```shell
--header "Content-Type: application/json"
```

このヘッダーは、リクエストボディがJSON文字列であることを示します。

## レスポンスメッセージ

### ステータスコード

- 200: HTTPリクエストが成功し、データがクライアントに送信される前のサーバーが正常であることを示します。
- 4xx: HTTPリクエストエラー。クライアントエラーを示します。
- `500 Internal Server Error`: HTTPリクエストが成功しましたが、データがクライアントに送信される前にサーバーでエラーが発生しました。
- 503: HTTPリクエストが成功しましたが、FEがサービスを提供できません。

### レスポンスヘッダー

`content-type` は、レスポンスボディの形式を示します。改行区切りのJSONが使用され、これによりレスポンスボディは複数のJSONオブジェクトで構成され、それらは`\n` で区切られます。

|                      | 説明                                                  |
| -------------------- | :----------------------------------------------------------- |
| content-type         | 形式は改行区切りのJSONで、デフォルトは "application/x-ndjson charset=UTF-8" です。 |
| X-StarRocks-Query-Id | クエリID。                                                          |

### レスポンスボディ

#### リクエストが送信される前に失敗した場合

リクエストがクライアント側で失敗したか、サーバーがクライアントにデータを返す前にエラーが発生した場合のレスポンスボディは以下の形式です。ここで、`msg` はエラー情報です。

```json
{
   "status":"FAILED",
   "msg":"xxx"
}
```

#### リクエストが通信後に失敗した場合

一部の結果が返され、HTTPステータスコードが200である場合。データ送信が中断され、接続が閉じられ、エラーがログに記録されます。

#### 成功した場合

レスポンスメッセージの各行はJSONオブジェクトです。JSONオブジェクトは`\n` で区切られます。

- SELECTステートメントの場合、次のJSONオブジェクトが返されます。

| オブジェクト        | 説明                                                  |
| -------------- | :----------------------------------------------------------- |
| `connectionId` | 接続ID。時間がかかるクエリをキャンセルするには、KILL `<connectionId>` を呼び出します。 |
| `meta`         | 列を表します。キーは `meta` で、値は列を表すJSON配列です。 |
| `data`         | データ行で、キーは `data` で、値はデータ行を含むJSON配列です。 |
| `statistics`   | クエリの統計情報。                                       |

- SHOWステートメントの場合、`meta`、`data`、`statistics` が返されます。
- EXPLAINステートメントの場合、クエリの詳細な実行計画を示す `explain` オブジェクトが返されます。

以下の例では、`\n` を区切りとして使用しています。StarRocksはHTTPチャンクモードを使用してデータを送信します。各FEがデータチャンクを取得するたびに、データチャンクをクライアントにストリーミングします。クライアントは行ごとにデータを解析できるため、データのキャッシュやデータ全体の待機をする必要がなく、クライアントのメモリ消費を削減できます。

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

## 例

### SELECTクエリを実行する

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "select * from agg;"}' --header "Content-Type: application/json"
```

結果:

```json
{"connectionId":49}
{"meta":[{"name":"no","type":"int(11)"},{"name":"k","type":"decimal64(10, 2)"},{"name":"v","type":"decimal64(10, 2)"]}
{"data":[1,"10.00",null]}
{"data":[2,"10.00","11.00"]}
{"data":[2,"20.00","22.00"]}
{"data":[2,"25.00",null]}
{"data":[2,"30.00","35.00"]}
{"statistics":{"scanRows":0,"scanBytes":0,"returnRows":5}}
```

### クエリをキャンセルする

予期しない長時間実行されたクエリをキャンセルするには、接続を閉じます。StarRocks は、接続が閉じられるとこのクエリをキャンセルします。

また、KILL `connectionId` を呼び出して、このクエリをキャンセルすることもできます。例:

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "kill 17;"}' --header "Content-Type: application/json"
```

`connectionId` は、レスポンスボディから取得するか、SHOW PROCESSLIST を呼び出すことで取得できます。例:

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "show processlist;"}' --header "Content-Type: application/json"
```

### セッション変数を設定してクエリを実行する

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:'  -d '{"query": "SHOW VARIABLES;", "sessionVariables":{"broadcast_row_limit":14000000}}'  --header "Content-Type: application/json"
```