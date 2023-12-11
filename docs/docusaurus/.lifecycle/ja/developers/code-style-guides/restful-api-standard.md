---
displayed_sidebar: "Japanese"
---

# StarRocks Restful API Standard

## APIの形式

1. APIの形式は次のパターンに従います: `/api/{version}/{target-object-access-path}/{action}`。
2. `{version}` は `v{number}` と表示され、例えば v1, v2, v3, v4 などです。
3. `{target-object-access-path}` は階層的に整理されており、後で詳しく説明します。
4. `{action}` はオプションで、API実装者はできるだけHTTPメソッドを利用して操作を表現するべきです。HTTPメソッドの意味を満たすことができない場合のみ、アクションを使用するべきです。たとえば、オブジェクトの名前を変更するための利用可能なHTTPメソッドがない場合など。

## ターゲットオブジェクトアクセスパスの定義

1. REST APIによってアクセスされるターゲットオブジェクトは、階層的なアクセスパスにカテゴリ分けされて整理される必要があります。アクセスパスの形式は次のとおりです:
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

カタログ、データベース、テーブル、カラムを例に取ると:
```
/catalogs: すべてのカタログを表します。
/catalogs/hive: カタログカテゴリ内の名前が"hive"である特定のカタログオブジェクトを表します。
/catalogs/hive/databases: "hive"カタログ内のすべてのデータベースを表します。
/catalogs/hive/databases/tpch_100g: "hive"カタログ内の"tpch_100g"という名前のデータベースを表します。
/catalogs/hive/databases/tpch_100g/tables: "tpch_100g"データベース内のすべてのテーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem: tpch_100g.lineitem テーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: tpch_100g.lineitem テーブル内のすべてのカラムを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: tpch_100g.lineitem テーブル内の特定のカラム l_orderkey を表します。
```

2. カテゴリはスネークケースで命名され、最後の単語は複数形になります。すべての単語は小文字であり、複数の単語はアンダースコア(_)で連結されます。具体的なオブジェクトは実際の名前を使用して命名されます。ターゲットオブジェクトの階層関係は明確に定義されている必要があります。

## HTTPメソッドの選択

1. GET: GETメソッドは単一のオブジェクトを表示したり、特定のカテゴリのすべてのオブジェクトをリストしたりするために使用されます。GETメソッドによるオブジェクトへのアクセスは読み取り専用であり、リクエストボディは提供されません。
```
# データベースssb_100g内のすべてのテーブルをリストします
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# テーブルssb_100g.lineorderを表示します
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST: オブジェクトの作成に使用されます。パラメータはリクエストボディを介して渡されます。これは冪等ではありません。オブジェクトがすでに存在する場合、繰り返し作成は失敗し、エラーメッセージが返されます。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT: オブジェクトの作成に使用されます。パラメータはリクエストボディを介して渡されます。これは冪等です。オブジェクトがすでに存在する場合、成功を返します。PUTメソッドはPOSTメソッドのCREATE IF NOT EXISTSバージョンです。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE: オブジェクトを削除するために使用されます。リクエストボディは提供されません。削除するオブジェクトが存在しない場合、成功を返します。DELETEメソッドには、DROP IF EXISTSのセマンティクスがあります。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH: オブジェクトを更新するために使用されます。リクエストボディが提供され、修正する必要がある部分情報のみが含まれます。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 認証と承認

1. 認証および承認情報はHTTPリクエストヘッダで渡されます。

## HTTPステータスコード

1. HTTPステータスコードは、操作の成功または失敗を示すためにREST APIによって返されます。
2. 成功した操作のステータスコード(2xx)は次のとおりです:

- 200 OK: リクエストが正常に完了したことを示します。これはオブジェクトの表示/リスト化/削除/更新および保留中のタスクの状態を問い合わせるために使用されます。
- 201 Created: オブジェクトの作成が正常に完了したことを示します。これはPUT/POSTメソッドに使用されます。応答ボディには、後続の表示/リスト化/削除/更新のためのオブジェクトURIを含める必要があります。
- 202 Accepted: タスクの提出が成功し、タスクが保留状態にあることを示します。応答ボディには、後続のタスクの取り消し、削除、およびタスクの状態のポーリングのためのタスクURIを含める必要があります。

3. エラーコード(4xx)はクライアントエラーを示します。ユーザーはHTTPリクエストを調整・修正し、再試行する必要があります。
- 400 Bad Request: 無効なリクエストパラメータ。
- 401 Unauthorized: 認証情報が欠落しているか、不正な認証情報、認証の失敗。
- 403 Forbidden: 認証は成功したが、ユーザーの操作は承認チェックに失敗しました。アクセス許可がありません。
- 404 Not Found: API URIのエンコードエラー。登録されたREST APIに属していません。
- 405 Method Not Allowed: 正しくないHTTPメソッドが使用されました。
- 406 Not Acceptable: 応答形式がAcceptヘッダで指定されたメディアタイプと一致しません。
- 415 Not Acceptable: リクエストコンテンツのメディアタイプがContent-Typeヘッダで指定されたメディアタイプと一致しません。

4. エラーコード(5xx)はサーバーエラーを示します。ユーザーはリクエストを変更する必要はなく、後で再試行することができます。
- 500 Internal Server Error: サーバー内部エラー、未知のエラーに類似しています。
- 503 Service Unavailable: サービスが一時的に利用できません。たとえば、ユーザーのアクセス頻度が高すぎてレート制限に達した場合、または3つのレプリカでテーブルを作成しようとしましたが、2つしか利用可能なBEがある場合、あるいはユーザーのクエリに関与するすべてのタブレットレプリカが利用できないため、サービスは現在サービスを提供できません。

## HTTP応答フォーマット

1. APIがHTTPコード200/201/202を返す場合、HTTP応答は空ではありません。APIはJSON形式で結果を返し、トップレベルフィールドには「code」、「message」、「result」が含まれます。すべてのJSONフィールドはキャメルケースで命名されます。

2. 成功したAPI応答では、「code」は「0」、 「message」は「OK」、 「result」に実際の結果が含まれます。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 失敗したAPI応答では、「code」は「0」ではありません、「message」は簡単なエラーメッセージであり、「result」にはエラースタックトレースなどの詳細なエラー情報が含まれることがあります。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## パラメータの渡し方

1. APIパラメータはパス、リクエストボディ、クエリパラメータ、ヘッダの優先順位で渡されます。適切なメソッドを選択してパラメータを渡す必要があります。

2. パスパラメータ: オブジェクトの階層関係を表す必須パラメータがパスパラメータに配置されます。
```
 /api/v2/warehouses/{warehouseName}/backends/{backendId}
 /api/v2/warehouses/ware0/backends/10027
```

3. リクエストボディ: パラメータはapplication/jsonを使用して渡されます。パラメータは必須またはオプションのタイプであることができます。

4. クエリパラメータ: クエリパラメータとリクエストボディパラメータの同時使用は許可されていません。同じAPIについては、どちらかを選択してください。ヘッダパラメータとパスパラメータを除くパラメータの数が2つ以下の場合、クエリパラメータを使用できます。それ以上の場合は、リクエストボディを使用してパラメータを渡す必要があります。

5. HEADERパラメータ: ヘッダは、Content-typeやAcceptなどのHTTP標準パラメータを渡すために使用されます。実装者はカスタマイズされたパラメータを渡すためにHTTPヘッダを濫用すべきではありません。ユーザー拡張のためにヘッダを使用する場合、ヘッダ名は `x-starrocks-{name}` の形式でなければなりません。ここで、nameには複数の英単語が含まれ、各単語は小文字で連結され、ハイフン(-)で結合されます。