---
displayed_sidebar: English
---

# StarRocks RESTful API標準

## API フォーマット

1. APIのフォーマットは次のパターンに従います: `/api/{version}/{target-object-access-path}/{action}`。
2. `{version}`は`v{number}`として表され、例えばv1、v2、v3、v4などです。
3. `{target-object-access-path}`は階層的に構成されており、詳細は後述します。
4. `{action}`はオプションで、API実装者はできるだけHTTPメソッドを使って操作の意味を伝えるべきです。HTTPメソッドのセマンティクスで対応できない場合のみ、アクションを使用します。例えば、オブジェクトの名前を変更するHTTPメソッドが存在しない場合などです。

## ターゲットオブジェクトアクセスパスの定義

1. REST APIによってアクセスされるターゲットオブジェクトは、階層的アクセスパスに分類して整理する必要があります。アクセスパスのフォーマットは以下の通りです:
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

例としてカタログ、データベース、テーブル、カラムを取り上げます:
```
/catalogs: すべてのカタログを表します。
/catalogs/hive: カタログカテゴリー下の「hive」という名前の特定のカタログオブジェクトを表します。
/catalogs/hive/databases: 「hive」カタログ内のすべてのデータベースを表します。
/catalogs/hive/databases/tpch_100g: 「hive」カタログ内の「tpch_100g」という名前のデータベースを表します。
/catalogs/hive/databases/tpch_100g/tables: 「tpch_100g」データベース内のすべてのテーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem: 「tpch_100g.lineitem」テーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: 「tpch_100g.lineitem」テーブルのすべてのカラムを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: 「tpch_100g.lineitem」テーブルの特定のカラム「l_orderkey」を表します。
```

2. カテゴリはスネークケースで命名され、最後の単語は複数形です。全ての単語は小文字で、複数の単語はアンダースコア(_)で連結されます。特定のオブジェクトはその実際の名前で命名されます。ターゲットオブジェクトの階層関係は明確に定義する必要があります。

## HTTPメソッドの選択

1. GET: GETメソッドは単一のオブジェクトを表示したり、特定のカテゴリの全オブジェクトを一覧表示するために使用します。GETメソッドによるオブジェクトへのアクセスは読み取り専用で、リクエストボディは提供されません。
```
# database ssb_100g内の全テーブルを一覧表示
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# table ssb_100g.lineorderを表示
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST: オブジェクトを作成するために使用されます。パラメータはリクエストボディを通じて渡されます。非冪等です。オブジェクトが既に存在する場合、繰り返しの作成は失敗し、エラーメッセージが返されます。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT: オブジェクトを作成するために使用されます。パラメータはリクエストボディを通じて渡されます。冪等です。オブジェクトが既に存在する場合は成功が返されます。PUTメソッドはPOSTメソッドのCREATE IF NOT EXISTSバージョンです。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE: オブジェクトを削除するために使用されます。リクエストボディは提供されません。削除対象のオブジェクトが存在しない場合は成功が返されます。DELETEメソッドはDROP IF EXISTSの意味を持ちます。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH: オブジェクトを更新するために使用されます。リクエストボディが提供され、変更が必要な部分情報のみを含みます。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 認証と認可

1. 認証と認可の情報はHTTPリクエストヘッダーで渡されます。

## HTTPステータスコード

1. HTTPステータスコードはREST APIによって返され、操作の成功または失敗を示します。
2. 成功した操作のステータスコード(2xx)は以下の通りです:

- 200 OK: リクエストが正常に完了したことを示します。オブジェクトの表示/リスト表示/削除/更新や保留中のタスクの状態を問い合わせる際に使用されます。
- 201 Created: オブジェクトが正常に作成されたことを示します。PUT/POSTメソッドで使用されます。レスポンスボディには後続の表示/リスト表示/削除/更新のためのオブジェクトURIが含まれている必要があります。
- 202 Accepted: タスクの提出が成功し、タスクが保留状態であることを示します。レスポンスボディにはタスクのキャンセル、削除、およびステータスのポーリングのためのタスクURIが含まれている必要があります。

3. エラーコード(4xx)はクライアントエラーを示します。ユーザーはHTTPリクエストを調整し、修正して再試行する必要があります。
- 400 Bad Request: リクエストパラメータが無効です。
- 401 Unauthorized: 認証情報が不足しているか不正です、または認証に失敗しました。
- 403 Forbidden: 認証には成功しましたが、ユーザーの操作が認可チェックに失敗しました。アクセス権がありません。
- 404 Not Found: API URIのエンコードが誤っています。登録されたREST APIには属していません。
- 405 Method Not Allowed: 不適切なHTTPメソッドが使用されました。
- 406 Not Acceptable: レスポンスの形式がAcceptヘッダーで指定されたメディアタイプと一致しません。
- 415 Unsupported Media Type: リクエストコンテンツのメディアタイプがContent-Typeヘッダーで指定されたメディアタイプと一致しません。

4. エラーコード(5xx)はサーバーエラーを示します。ユーザーはリクエストを変更する必要はなく、後で再試行できます。
- 500 Internal Server Error: 内部サーバーエラーで、未知のエラーに類似しています。
- 503 Service Unavailable: サービスは一時的に利用できません。例えば、ユーザーのアクセス頻度が高すぎてレート制限に達した場合、または3つのレプリカを持つテーブルを作成する際に利用可能なBEが2つしかないなど、内部状態によりサービスが現在提供できない場合です。ユーザーのクエリに関わるすべてのタブレットレプリカが利用不可です。

## HTTPレスポンスフォーマット

1. APIが200/201/202のHTTPコードを返す場合、HTTPレスポンスは空ではありません。APIはJSON形式で結果を返し、「code」、「message」、および「result」のトップレベルフィールドを含みます。すべてのJSONフィールドはキャメルケースで命名されます。

2. APIの成功した応答では、「code」は「0」、「message」は「OK」で、「result」には実際の結果が含まれます。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 失敗したAPIの応答では、「code」は「0」ではなく、「message」は単純なエラーメッセージで、「result」にはエラースタックトレースなどの詳細なエラー情報が含まれることがあります。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## パラメータの受け渡し

1. APIパラメータは、パス、リクエストボディ、クエリパラメータ、ヘッダの優先順位で渡されます。パラメータの受け渡しに適切な方法を選択してください。

2. パスパラメータ: オブジェクトの階層関係を表す必須パラメータはパスパラメータに配置されます。
```
 /api/v2/warehouses/{warehouseName}/backends/{backendId}
 /api/v2/warehouses/ware0/backends/10027
```

3. リクエストボディ: パラメータはapplication/jsonを使用して渡されます。パラメータは必須またはオプショナルです。

4. クエリパラメータ: クエリパラメータとリクエストボディパラメータを同時に使用することは許可されません。同じAPIについては、どちらか一方を選択してください。ヘッダパラメータとパスパラメータを除くパラメータの数が2つ以下であれば、クエリパラメータを使用できます。それ以外の場合は、リクエストボディを使用してパラメータを渡してください。

5. HEADERパラメータ: ヘッダは、Content-typeやAcceptなどのHTTP標準パラメータを渡すために使用され、実装者はカスタマイズされたパラメータを渡すためにHTTPヘッダを乱用すべきではありません。ユーザー拡張機能のパラメータをヘッダで渡す場合、ヘッダ名は`x-starrocks-{name}`の形式であるべきです。名前には複数の英単語を含めることができ、各単語は小文字でハイフン(-)で連結されます。
