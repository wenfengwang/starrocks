---
displayed_sidebar: "Japanese"
---

# StarRocks Restful API 標準

## API フォーマット

1. API のフォーマットは次のパターンに従います: `/api/{version}/{target-object-access-path}/{action}`.
2. `{version}` は `v{number}` と表記され、例えば v1、v2、v3、v4 などです。
3. `{target-object-access-path}` は階層的に組織され、後で詳しく説明します。
4. `{action}` はオプションであり、可能な限り HTTP メソッドを使用して操作の意味を伝えるべきです。HTTP メソッドの意味が満たされない場合にのみ、アクションを使用する必要があります。例えば、オブジェクトの名前を変更するための利用可能な HTTP メソッドがない場合です。

## ターゲットオブジェクトアクセスパスの定義

1. REST API でアクセスされるターゲットオブジェクトは、階層的なアクセスパスに分類されて組織化する必要があります。アクセスパスの形式は次のとおりです:
```
/primary_categories/primary_object/secondary_categories/secondary_object/.../categories/object
```

カタログ、データベース、テーブル、カラムを例に取ります:
```
/catalogs: すべてのカタログを表します。
/catalogs/hive: カタログカテゴリー内の "hive" という名前の特定のカタログオブジェクトを表します。
/catalogs/hive/databases: "hive" カタログ内のすべてのデータベースを表します。
/catalogs/hive/databases/tpch_100g: "hive" カタログ内の "tpch_100g" という名前のデータベースを表します。
/catalogs/hive/databases/tpch_100g/tables: "tpch_100g" データベース内のすべてのテーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem: tpch_100g.lineitem テーブルを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns: tpch_100g.lineitem テーブルのすべてのカラムを表します。
/catalogs/hive/databases/tpch_100g/tables/lineitem/columns/l_orderkey: tpch_100g.lineitem テーブルの特定のカラム l_orderkey を表します。
```

2. カテゴリはスネークケースで命名され、最後の単語は複数形です。すべての単語は小文字で、複数の単語はアンダースコア (_) で接続されます。具体的なオブジェクトは実際の名前を使用して命名されます。ターゲットオブジェクトの階層関係は明確に定義する必要があります。

## HTTP メソッドの選択

1. GET: GET メソッドを使用して単一のオブジェクトを表示し、特定のカテゴリのすべてのオブジェクトをリストします。GET メソッドによるオブジェクトへのアクセスは読み取り専用であり、リクエストボディは提供されません。
```
# データベース ssb_100g 内のすべてのテーブルをリストします。
GET /api/v2/catalogs/default/databases/ssb_100g/tables

# テーブル ssb_100g.lineorder を表示します。
GET /api/v2/catalogs/default/databases/ssb_100g/tables/lineorder
```

2. POST: オブジェクトを作成するために使用されます。パラメータはリクエストボディを介して渡されます。これは幂等ではありません。オブジェクトが既に存在する場合、繰り返し作成は失敗し、エラーメッセージが返されます。
```
POST /api/v2/catalogs/default/databases/ssb_100g/tables/create -d@create_customer.sql
```

3. PUT: オブジェクトを作成するために使用されます。パラメータはリクエストボディを介して渡されます。これは幂等です。オブジェクトが既に存在する場合、成功が返されます。PUT メソッドは POST メソッドの CREATE IF NOT EXISTS バージョンです。
```
PUT /api/v2/databases/ssb_100g/tables/create -d@create_customer.sql
```

4. DELETE: オブジェクトを削除するために使用されます。リクエストボディは提供されません。削除するオブジェクトが存在しない場合、成功が返されます。DELETE メソッドは DROP IF EXISTS のセマンティクスを持ちます。
```
DELETE /api/v2/catalogs/default/databases/ssb_100g/tables/customer
```

5. PATCH: オブジェクトを更新するために使用されます。リクエストボディが提供され、変更する必要がある部分のみを含みます。
```
PATCH /api/v2/databases/ssb_100g/tables/customer -d '{"unique_key_constraints": ["c_custkey"]}'
```

## 認証と承認

1. 認証および承認情報は HTTP リクエストヘッダーに渡されます。

## HTTP ステータスコード

1. HTTP ステータスコードは、REST API によって操作の成功または失敗を示すために返されます。
2. 成功した操作のステータスコード (2xx) は次のとおりです:

- 200 OK: リクエストが正常に完了したことを示します。オブジェクトの表示/リスト/削除/更新および保留中のタスクのステータスのクエリに使用されます。
- 201 Created: オブジェクトが正常に作成されたことを示します。PUT/POST メソッドに使用されます。レスポンスボディには、オブジェクトの URI が含まれている必要があり、その後の表示/リスト/削除/更新に使用されます。
- 202 Accepted: タスクの送信が成功し、タスクが保留状態にあることを示します。レスポンスボディには、タスクの URI が含まれている必要があり、その後のキャンセル、削除、およびタスクステータスのポーリングに使用されます。

3. エラーコード (4xx) はクライアントエラーを示します。ユーザーは HTTP リクエストを調整および修正し、再試行する必要があります。
- 400 Bad Request: 無効なリクエストパラメータです。
- 401 Unauthorized: 認証情報が不足している、不正な認証情報、認証の失敗です。
- 403 Forbidden: 認証は成功しましたが、ユーザーの操作が承認チェックに失敗しました。アクセス権限がありません。
- 404 Not Found: API URI のエンコーディングエラーです。登録された REST API に属していません。
- 405 Method Not Allowed: 正しくない HTTP メソッドが使用されました。
- 406 Not Acceptable: 応答形式が Accept ヘッダーで指定されたメディアタイプと一致しません。
- 415 Not Acceptable: リクエストコンテンツのメディアタイプが Content-Type ヘッダーで指定されたメディアタイプと一致しません。

4. エラーコード (5xx) はサーバーエラーを示します。ユーザーはリクエストを修正する必要はなく、後で再試行することができます。
- 500 Internal Server Error: サーバー内部エラーです。不明なエラーに類似しています。
- 503 Service Unavailable: サービスが一時的に利用できません。例えば、ユーザーのアクセス頻度が高すぎてレート制限に達した場合、またはサービスが内部の状態により現在サービスを提供できない場合、例えば 3 つのレプリカを持つテーブルを作成しようとしているが、利用可能な BE が 2 つしかない場合、ユーザーのクエリに関与するすべての Tablet レプリカが利用できない場合などです。

## HTTP レスポンスフォーマット

1. API が HTTP コード 200/201/202 を返す場合、HTTP レスポンスは空ではありません。API は JSON 形式で結果を返し、トップレベルのフィールドには "code"、"message"、"result" が含まれます。すべての JSON フィールドはキャメルケースで命名されます。

2. 成功した API レスポンスでは、"code" は "0"、"message" は "OK"、"result" には実際の結果が含まれます。
```json
{
   "code":"0",
   "message": "OK",
   "result": {....}
}
```

3. 失敗した API レスポンスでは、"code" は "0" ではなく、"message" は簡単なエラーメッセージであり、"result" にはエラースタックトレースなどの詳細なエラー情報が含まれる場合があります。
```json
{
   "code":"1",
   "message": "Analyze error",
   "result": {....}
}
```

## パラメータの渡し方

1. API パラメータは、パス、リクエストボディ、クエリパラメータ、ヘッダーの優先順位で渡されます。適切な方法を選択してパラメータを渡します。

2. パスパラメータ: オブジェクトの階層関係を表す必要がある必須パラメータは、パスパラメータに配置されます。
```
 /api/v2/warehouses/{warehouseName}/backends/{backendId}
 /api/v2/warehouses/ware0/backends/10027
```

3. リクエストボディ: パラメータは application/json を使用して渡されます。パラメータは必須またはオプションのタイプである場合があります。

4. クエリパラメータ: クエリパラメータとリクエストボディパラメータを同時に使用することはできません。同じ API の場合、どちらかを選択します。ヘッダーパラメータとパスパラメータを除くパラメータの数が 2 つ以下の場合、クエリパラメータを使用できます。それ以外の場合は、リクエストボディを使用してパラメータを渡します。

5. ヘッダーパラメータ: ヘッダーには Content-type や Accept などの HTTP 標準パラメータを渡すために使用します。実装者はカスタムパラメータを渡すために http ヘッダーを乱用しないでください。ユーザー拡張のためにヘッダーを使用してパラメータを渡す場合、ヘッダー名は `x-starrocks-{name}` の形式である必要があります。name には複数の英単語を含めることができ、各単語は小文字で、ハイフン (-) で連結されます。
