---
displayed_sidebar: "Japanese"
---

# Insert Into

## データの挿入時、SQLにおける各挿入は50から100msを要します。効率を上げる方法はありますか？

OLAPへのデータ挿入について、一つ一つのデータを挿入することは推奨されません。通常はバッチで挿入されます。どちらの方法も同じ時間を要します。

## 「Insert into select」タスクでエラーが発生する：index channel has intoleralbe failure

Stream Load RPCのタイムアウト期間を変更することで、この問題を解決できます。「be.conf」の以下の項目を変更し、変更を有効にするためにマシンを再起動します。

`streaming_load_rpc_max_alive_time_sec`: Stream LoadのRPCタイムアウト。単位: 秒。デフォルト: `1200`。

または、次の変数を使用してクエリのタイムアウトを設定することもできます。

`query_timeout`: クエリのタイムアウト期間。単位: 秒。デフォルト値: `300`。

## 大容量のデータをロードするためにINSER INTO SELECTコマンドを実行すると、「execute timeout」というエラーが発生します

デフォルトでは、クエリのタイムアウト期間は300秒です。この期間を延長するには、変数`query_timeout`を設定できます。単位は秒です。