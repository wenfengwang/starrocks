---
displayed_sidebar: English
---

# HTTP SQL API

## 説明

StarRocks v3.2.0では、HTTPを使用してさまざまなタイプのクエリを実行できるHTTP SQL APIが導入されました。現在、このAPIはSELECT、SHOW、EXPLAIN、およびKILLステートメントをサポートしています。

curlコマンドを使用した構文:

```shell
curl -X POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql' \
   -u '<username>:<password>' -d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}' \
   --header "Content-Type: application/json"
```

## リクエストメッセージ

### リクエスト行

```shell
POST 'http://<fe_ip>:<fe_http_port>/api/v1/catalogs/<catalog_name>/databases/<database_name>/sql'
```

| 項目                    | 説明                                                  |
| ------------------------ | ----------------------------------------------------------- |
|  fe_ip                   | FEノードのIPアドレス。                                                  |
|  fe_http_port            | FEのHTTPポート。                                           |
|  catalog_name            | カタログ名。現在、このAPIは`default_catalog`のみを`<catalog_name>`として設定できます。 |
|  database_name           | データベース名。リクエスト行にデータベース名が指定されていない場合、SQLクエリでテーブル名が使用されると、テーブル名の前にデータベース名を付ける必要があります（例：`database_name.table_name`）。 |

- 指定されたカタログ内のデータベース間でデータをクエリします。SQLクエリでテーブルが使用される場合、テーブル名の前にデータベース名を付ける必要があります。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/sql
   ```

- 指定されたカタログとデータベースからデータをクエリします。

   ```shell
   POST /api/v1/catalogs/<catalog_name>/databases/<database_name>/sql
   ```

### 認証方法

```shell
Authorization: Basic <credentials>
```

基本認証が使用されます。つまり、`credentials`にはユーザー名とパスワードを入力します（`-u '<username>:<password>'`）。ユーザー名にパスワードが設定されていない場合、`<username>:`のみを入力し、パスワードを空にすることができます。たとえば、rootアカウントを使用する場合、`-u 'root:'`と入力できます。

### リクエストボディ

```shell
-d '{"query": "<sql_query>;", "sessionVariables":{"<var_name>":<var_value>}}'
```

| 項目                    | 説明                                                  |
| ------------------------ | ----------------------------------------------------------- |
| query                    | STRING形式のSQLクエリ。SELECT、SHOW、EXPLAIN、およびKILLステートメントのみがサポートされています。HTTPリクエストに対して1つのSQLクエリのみを実行できます。 |
| sessionVariables         | クエリに設定する[セッション変数](../System_variable.md)（JSON形式）。このフィールドはオプションです。デフォルトは空です。設定したセッション変数は、同じ接続に対して有効であり、接続が閉じられると無効になります。 |

### リクエストヘッダー

```shell
--header "Content-Type: application/json"
```

このヘッダーは、リクエストボディがJSON文字列であることを示します。

## レスポンスメッセージ

### ステータスコード

- 200: HTTPリクエストが成功し、データがクライアントに送信される前にサーバーは正常です。
- 4xx: クライアントエラーを示すHTTPリクエストエラー。
- `500 Internal Server Error`: HTTPリクエストは成功しましたが、データがクライアントに送信される前にサーバーでエラーが発生しました。
- 503: HTTPリクエストは成功しましたが、FEはサービスを提供できません。

### レスポンスヘッダー

`content-type`はレスポンスボディの形式を示します。改行で区切られたJSONが使用され、レスポンスボディは`\n`で区切られた複数のJSONオブジェクトで構成されます。

|                      | 説明                                                  |
| -------------------- | ----------------------------------------------------------- |
| content-type         | 形式は改行で区切られたJSONで、デフォルトは"application/x-ndjson charset=UTF-8"です。 |
| X-StarRocks-Query-Id | クエリID。                                                          |

### レスポンスボディ

#### リクエストが送信される前に失敗

リクエストがクライアント側で失敗するか、サーバーがクライアントにデータを返す前にエラーが発生します。レスポンスボディは次の形式です。ここで`msg`はエラー情報です。

```json
{
   "status":"FAILED",
   "msg":"xxx"
}
```

#### リクエストが送信された後に失敗

結果の一部が返され、HTTPステータスコードは200です。データ送信が中断され、接続が閉じられ、エラーがログに記録されます。

#### 成功

レスポンスメッセージの各行はJSONオブジェクトです。`\n`で区切られたJSONオブジェクトです。

- SELECTステートメントの場合、以下のJSONオブジェクトが返されます。

| オブジェクト       | 説明                                                  |
| ------------ | ----------------------------------------------------------- |
| `connectionId` | 接続ID。長時間保留中のクエリをキャンセルするには、KILL `<connectionId>`を呼び出します。 |
| `meta`        | 列を表します。キーは`meta`で、値は列を表すオブジェクトが含まれるJSON配列です。 |
| `data`         | データ行です。キーは`data`で、値はデータ行を含むJSON配列です。 |
| `statistics`   | クエリの統計情報です。                                       |

- SHOWステートメントの場合、`meta`、`data`、および`statistics`が返されます。
- EXPLAINステートメントの場合、クエリの詳細な実行プランを示す`explain`オブジェクトが返されます。

次の例では、`\n`を区切り文字として使用します。StarRocksはHTTPチャンクモードを使用してデータを送信します。FEがデータチャンクを取得するたびに、それをクライアントにストリーミングします。クライアントは行ごとにデータを解析できるため、データキャッシュやデータ全体を待つ必要がなくなり、クライアントのメモリ消費を削減できます。

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

### クエリをキャンセルする

予期せず長い時間実行されるクエリをキャンセルするには、接続を閉じます。StarRocksは接続が閉じられたことを検出すると、このクエリをキャンセルします。

また、KILL `<connectionId>`を呼び出してこのクエリをキャンセルすることもできます。例えば：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "kill 17;"}' --header "Content-Type: application/json"
```

`connectionId`はレスポンスボディから、またはSHOW PROCESSLISTを呼び出すことによって取得できます。例えば：

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:' -d '{"query": "show processlist;"}' --header "Content-Type: application/json"
```

### このクエリに設定されたセッション変数を使用してクエリを実行する

```shell
curl -X POST 'http://127.0.0.1:8030/api/v1/catalogs/default_catalog/databases/test/sql' -u 'root:'  -d '{"query": "SHOW VARIABLES;", "sessionVariables":{"broadcast_row_limit":14000000}}'  --header "Content-Type: application/json"
```
