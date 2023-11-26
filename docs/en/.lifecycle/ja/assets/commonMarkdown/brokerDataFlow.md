### Broker Loadの利点

- Broker Loadは、データの変換、UPSERT、およびDELETE操作をロード中にサポートしています。
- Broker Loadはバックグラウンドで実行され、ジョブが続行されるためにクライアントが接続されている必要はありません。
- Broker Loadは長時間実行されるジョブに適しており、デフォルトのタイムアウトは4時間です。
- Broker LoadはParquet、ORC、およびCSVファイル形式をサポートしています。

### データフロー

![Broker Loadのワークフロー](../broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します。
2. フロントエンド（FE）がクエリプランを作成し、バックエンドノード（BE）にプランを配布します。
3. バックエンド（BE）ノードはソースからデータを取得し、データをStarRocksにロードします。
