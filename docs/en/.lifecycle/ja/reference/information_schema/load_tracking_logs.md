---
displayed_sidebar: "Japanese"
---

# load_tracking_logs

`load_tracking_logs`はロードジョブのエラーログを提供します。このビューはStarRocks v3.0以降でサポートされています。

`load_tracking_logs`には以下のフィールドが提供されています：

| **フィールド**   | **説明**                                     |
| --------------- | -------------------------------------------- |
| JOB_ID          | ロードジョブのIDです。                         |
| LABEL           | ロードジョブのラベルです。                     |
| DATABASE_NAME   | ロードジョブが所属するデータベースです。       |
| TRACKING_LOG    | ロードジョブのエラー（あれば）です。           |
