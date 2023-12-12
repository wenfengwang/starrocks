```yaml
---
displayed_sidebar: "Japanese"
---

# HTTP SQL API

## 説明

StarRocks v3.2.0 では、ユーザーがHTTPを使用してさまざまな種類のクエリを実行するためのHTTP SQL APIが導入されます。現時点では、このAPIはSELECT、SHOW、EXPLAIN、KILLステートメントをサポートしています。

curlコマンドを使用した構文：

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

| フィールド                 | 説明                                                        |
| ------------------------ | :----------------------------------------------------------- |
|  fe_ip                   | FEノードIPアドレス。                                                  |
|  fe_http_port            | FE HTTPポート。                                           |
|  catalog_name            | カタログ名。現在、このAPIは内部テーブルのクエリのみをサポートしているため、`<catalog_name>` は`default_catalog`のみに設定できます。 |
|  database_name           | データベース名。リクエストラインにデータベース名が指定されていない場合、SQLクエリでテーブル名を使用する場合は、テーブル名をそのデータベース名と共に接頭辞として付ける必要があります。例：`database_name.table_name`。 |

- 指定されたカタログ内の複数のデータベースをクエリする。SQLクエリでテーブルを使用する場合、テーブル名をそのデータベース名と共に接頭辞として付ける必要があります。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/sql
   ```

- 指定されたカタログとデータベースからデータをクエリする。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/databases/<database_name>/sql
   ```

### 認証方法

```shell
Authorization: Basic <credentials>
```

Basic認証が使用されます。つまり、`credentials`にユーザー名とパスワードを入力します（`-u '<username>:<password>'`）。ユーザー名のパスワードが設定されていない場合、`<username>:` だけを渡してパスワードを空白のままにしてください。たとえば、rootアカウントを使用する場合、`-u 'root:'` と入力できます。

### リクエストボディ

```shell
-d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}'
```

| フィールド                 | 説明                                                        |
| ------------------------ | :----------------------------------------------------------- |
| query                    | SQLクエリ（STRING形式）。SELECT、SHOW、EXPLAIN、KILLステートメントのみがサポートされています。HTTPリクエストに対して1つのSQLクエリのみを実行できます。 |
| sessionVariables         | クエリのために設定したい[セッション変数](../System_variable.md)、JSON形式。このフィールドはオプションです。デフォルトは空です。設定したセッション変数は同じ接続に対して有効であり、接続が閉じられると無効になります。|

### リクエストヘッダ

```shell
--header "Content-Type: application/json"
```

このヘッダは、リクエストボディがJSON文字列であることを示しています。

## レスポンスメッセージ

### ステータスコード

- 200: HTTPリクエストが成功し、データがクライアントに送信される前のサーバーが正常であることを示します。
- 4xx: HTTPリクエストエラー、クライアントエラーを示します。
- `500 Internal Server Error`: HTTPリクエストが成功しましたが、サーバーがクライアントにデータを送信する前にエラーが発生しました。
- 503: HTTPリクエストは成功しましたが、FEがサービスを提供できません。

### レスポンスヘッダ

`content-type`は、レスポンスボディの形式を示します。Newline delimited JSONが使用され、レスポンスボディは`\n`で区切られた複数のJSONオブジェクトで構成されています。

|                      | 説明                                                        |
| -------------------- | :----------------------------------------------------------- |
| content-type         | フォーマットは、Newline delimited JSON で、デフォルトは "application/x-ndjson charset=UTF-8" です。 |
| X-StarRocks-Query-Id | クエリID。                                                          |

### レスポンスボディ

#### リクエストが送信される前に失敗した場合

クライアント側でリクエストが失敗したか、サーバーがクライアントにデータを返す前にエラーが発生した場合、レスポンスボディは次の形式になります。`msg`はエラー情報です。

```json
{
   "status":"FAILED",
   "msg":"xxx"
}
```

#### リクエストが送信された後に失敗した場合

一部の結果が返され、HTTPステータスコードが200になります。データ送信が中断され、接続が閉じられ、エラーがログに記録されます。

#### 成功した場合

レスポンスメッセージの各行はJSONオブジェクトです。JSONオブジェクトは`\n`で区切られます。

- SELECTステートメントの場合、次のJSONオブジェクトが返されます。

| オブジェクト       | 説明                                                        |
| ------------ | :----------------------------------------------------------- |
| `connectionId` | 接続ID。長時間保留されたクエリをKILL `<connectionId>` を呼び出すことでキャンセルできます。 |
| `meta`        | 列を表します。キーは`meta`で、値は各列を表すJSON配列です。 |
| `data`         | データ行です。キーは`data`で、値はデータの行を含むJSON配列です。 |
| `statistics`   | クエリの統計情報。                                       |

- SHOWステートメントの場合、`meta`、`data`、`statistics`が返されます。
- EXPLAINステートメントの場合、クエリの詳細な実行計画を示す`explain`オブジェクトが返されます。

下記の例は、区切り文字として`\n`を使用しています。StarRocksはHTTPのチャンクモードを使用してデータを転送します。FEはデータチャンクを取得するたびに、それをクライアントにストリームします。クライアントは新しいレコードごとにデータをパースできますので、データのキャッシュやデータの完全な待ちの必要がなくなり、クライアントのメモリ消費が低減されます。

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

### SELECTクエリの実行

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "select * from agg;"}' --header "Content-Type: application/json"
```

結果：

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

予期せず長時間実行されるクエリをキャンセルするには、接続を閉じることができます。StarRocksは接続が閉じられたと検出すると、このクエリをキャンセルします。

また、KILL `connectionId` を呼び出してこのクエリをキャンセルすることもできます。例：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "kill 17;"}' --header "Content-Type: application/json"
```

レスポンスボディまたはSHOW PROCESSLISTを呼び出すことで、`connectionId` を取得することができます。例：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "show processlist;"}' --header "Content-Type: application/json"
```

### セッション変数を設定してクエリを実行

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:'  -d '{"query": "SHOW VARIABLES;", "sessionVariables":{"broadcast_row_limit":14000000}}'  --header "Content-Type: application/json"
```