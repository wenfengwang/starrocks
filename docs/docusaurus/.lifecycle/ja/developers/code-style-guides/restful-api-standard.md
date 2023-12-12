---
displayed_sidebar: "Japanese"
---

# StarRocks Restful API Standard

## API フォーマット

1. API フォーマットは次のパターンに従います：`/api/{version}/{target-object-access-path}/{action}`。
2. `{version}` は `v{number}` として表され、たとえば v1、v2、v3、v4、などです。
3. `{target-object-access-path}` は階層的に構成されており、後で詳しく説明します。
4. `{action}` はオプションであり、API 実装者はできる限り HTTP メソッドを利用して操作の意味を伝えるべきです。HTTP メソッドの意味が満たされない場合のみ、アクションを使用するべきです。たとえば、オブジェクトの名前を変更するための利用可能な HTTP メソッドがない場合にのみアクションを使用します。

## ターゲットオブジェクトアクセスパスの定義

1. REST API によってアクセスされるターゲットオブジェクトは、階層的なアクセスパスに分類されて整理される必要があります。アクセスパスの形式は次のとおりです：
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

カタログ、データベース、テーブル、カラムを例に取り上げます：
```
/catalogs: すべてのカタログを表します。
/catalogs/hive: カタログカテゴリー内の "hive" という特定のカタログオブジェクトを表します。
/catalogs/hive/databases: "hive" カタログ内のすべてのデータベースを表します。
/catalogs/hive/databases/tpch_100g: "hive" カタログ内の "tpch_100g" というデータベースを表します。
/catalogs/hive/databases/tpch_100g/tables: "tpch_100g" データベース内のすべてのテーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem: tpch_100g.lineitem テーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: tpch_100g.lineitem テーブル内のすべてのカラムを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: tpch_100g.lineitem テーブル内の特定のカラム l_orderkey を表します。
```

2. カテゴリーはスネークケースで命名され、最後の単語は複数形になります。すべての単語は小文字で、複数の単語はアンダースコア(_)で接続されます。具体的なオブジェクトはその実際の名前で命名されます。ターゲットオブジェクトの階層関係は明確に定義されている必要があります。

## HTTP メソッドの選択

1. GET: GET メソッドを使用して単一のオブジェクトを表示し、特定のカテゴリーのすべてのオブジェクトを一覧表示します。GET メソッドによるオブジェクトへのアクセスは読み取り専用であり、リクエストボディは提供されません。
```
# データベース ssb_100g 内のすべてのテーブルを一覧表示する
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# テーブル ssb_100g.lineorder を表示する
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST: オブジェクトを作成するために使用されます。パラメータはリクエストボディを介して渡されます。これは冪等ではありません。オブジェクトが既に存在する場合、繰り返しの作成は失敗し、エラーメッセージが返されます。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT: オブジェクトを作成するために使用されます。パラメータはリクエストボディを介して渡されます。これは冪等です。オブジェクトが既に存在する場合、成功を返します。PUT メソッドは POST メソッドの CREATE IF NOT EXISTS バージョンです。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE: オブジェクトを削除するために使用されます。リクエストボディは提供されません。削除するオブジェクトが存在しない場合、成功を返します。DELETE メソッドには DROP IF EXISTS の意味があります。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH: オブジェクトを更新するために使用されます。リクエストボディが提供され、修正する必要のある部分の情報のみを含みます。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 認証と認可

1. 認証および認可情報は HTTP リクエストヘッダーで渡されます。

## HTTP ステータスコード

1. HTTP ステータスコードは、操作の成功または失敗を示すために REST API によって返されます。
2. 成功した操作のステータスコード（2xx）は以下のとおりです：

- 200 OK: リクエストが正常に完了したことを示します。オブジェクトの表示/一覧表示/削除/更新や保留中のタスクの状態のクエリに使用されます。
- 201 Created: オブジェクトが正常に作成されたことを示します。PUT/POST メソッドに使用されます。レスポンスボディには、後続の表示/一覧表示/削除/更新のためのオブジェクト URI を含める必要があります。
- 202 Accepted: タスクの送信が成功し、タスクが保留状態にあることを示します。レスポンスボディには、後続のキャンセル、削除、タスクの状態のポーリングのためのタスク URI を含める必要があります。

3. エラーコード（4xx）はクライアントエラーを示します。ユーザーは HTTP リクエストを調整して再試行する必要があります。
- 400 Bad Request: 無効なリクエストパラメータ。
- 401 Unauthorized: 認証情報の不足、不正な認証情報、認証の失敗。
- 403 Forbidden: 認証に成功しましたが、ユーザーの操作が認可チェックに失敗しました。アクセス許可がありません。
- 404 Not Found: API URI エンコーディングエラー。登録された REST API に属していません。
- 405 Method Not Allowed: 不正な HTTP メソッドが使用されました。
- 406 Not Acceptable: 応答形式が Accept ヘッダーで指定されたメディアタイプと一致しません。
- 415 Not Acceptable: リクエストコンテンツのメディアタイプが Content-Type ヘッダーで指定されたメディアタイプと一致しません。

4. エラーコード（5xx）はサーバーエラーを示します。ユーザーはリクエストを変更する必要はなく、後で再試行できます。
- 500 Internal Server Error: サーバー内部エラー、未知のエラーに類似しています。
- 503 Service Unavailable: サービスが一時的に利用できません。たとえば、ユーザーのアクセス頻度が高すぎてレート制限に達した場合、または現在サービスが内部の状態によりサービスを提供できない場合（たとえば、3 つのレプリカを持つテーブルを作成しても、使用可能な BE が 2 つしかない場合）。ユーザーのクエリに関与するすべての Tablet レプリカが利用できません。

## HTTP 応答フォーマット

1. API が 200/201/202 の HTTP コードを返す場合、HTTP 応答は空でないです。API は JSON 形式で結果を返し、「code」、「message」、「result」というトップレベルフィールドを含みます。すべての JSON フィールドはキャメルケースで命名されます。

2. 成功した API 応答では、「code」は "0"、「message」は "OK" であり、「result」に実際の結果が含まれます。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 失敗した API 応答では、「code」は "0" ではなく、 「message」は簡単なエラーメッセージであり、「result」にエラースタックトレースなどの詳細なエラー情報が含まれる場合があります。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## パラメータの渡し方

1. API パラメータはパス、リクエストボディ、クエリパラメータ、およびヘッダーの優先順位で渡されます。パラメータの渡し方は適切に選択する必要があります。

2. パスパラメータ: オブジェクトの階層関係を表す必須パラメータはパスパラメータに配置されます。
```
/api/v2/warehouses/{warehouseName}/backends/{backendId}
/api/v2/warehouses/ware0/backends/10027
```

3. リクエストボディ: パラメータは application/json を使用して渡されます。パラメータは必須またはオプションのタイプであることができます。

4. クエリパラメータ: 同じ API でクエリパラメータとリクエストボディパラメータの両方を使用することはできません。パラメータの数がヘッダーパラメータとパスパラメータを除くとき、2 つを超えない場合はクエリパラメータを使用し、それ以外の場合はリクエストボディを使用してパラメータを渡す必要があります。

5. ヘッダーパラメータ: ヘッダーには Content-type や Accept などの HTTP 標準パラメータを渡すために使用されます。実装者はカスタムパラメータを渡すために HTTP ヘッダーを濫用すべきではありません。ユーザー拡張のためにヘッダーを使用する場合、ヘッダー名は `x-starrocks-{name}` の形式である必要があります。ここで、name は複数の英単語を含み、各単語は小文字でハイフン(-)で接続されています。