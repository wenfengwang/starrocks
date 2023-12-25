### Broker Loadの利点

- Broker Loadは、ロード中のデータ変換、UPSERT、およびDELETE操作をサポートします。
- Broker Loadはバックグラウンドで実行され、クライアントはジョブが続行されるために接続されている必要はありません。
- 長時間実行されるジョブにはBroker Loadが推奨され、デフォルトのタイムアウトは4時間です。
- Broker LoadはParquet、ORC、CSVファイル形式をサポートしています。

### データフロー

![Broker Loadのワークフロー](../broker_load_how-to-work_en.png)

1. ユーザーがロードジョブを作成します
2. フロントエンド(FE)がクエリプランを作成し、そのプランをバックエンドノード(BE)に配布します
3. バックエンド(BE)ノードはソースからデータを取得し、StarRocksにデータをロードします
