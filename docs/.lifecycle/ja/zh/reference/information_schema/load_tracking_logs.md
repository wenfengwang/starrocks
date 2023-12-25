---
displayed_sidebar: Chinese
---

# load_tracking_logs

インポートジョブに関連するエラー情報を提供します。このビューは StarRocks v3.0 からサポートされています。

`load_tracking_logs` は以下のフィールドを提供します：

| **フィールド** | **説明**                           |
| -------------- | ---------------------------------- |
| JOB_ID         | インポートジョブの ID。            |
| LABEL          | インポートジョブのラベル。         |
| DATABASE_NAME  | インポートジョブが属するデータベース名。 |
| TRACKING_LOG   | インポートジョブのエラーログ情報（存在する場合）。 |
