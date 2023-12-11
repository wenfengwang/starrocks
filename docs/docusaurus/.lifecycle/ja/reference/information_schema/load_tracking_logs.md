---
displayed_sidebar: "Japanese"
---

# load_tracking_logs

`load_tracking_logs`は、ロードジョブのエラーログを提供します。このビューは、StarRocks v3.0以降でサポートされています。

`load_tracking_logs`には、以下のフィールドが提供されています：

| **Field**     | **Description**                            |
| ------------- | ------------------------------------------ |
| JOB_ID        | ロードジョブのIDです。                     |
| LABEL         | ロードジョブのラベルです。                 |
| DATABASE_NAME | ロードジョブが属するデータベースです。     |
| TRACKING_LOG  | ロードジョブのエラー（あれば）です。       |