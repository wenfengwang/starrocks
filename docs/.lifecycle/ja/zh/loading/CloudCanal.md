---
displayed_sidebar: Chinese
---

# CloudCanal を使用したインポート

## 紹介

CloudCanal コミュニティ版は、[ClouGence 社](https://www.clougence.com)が提供する、構造移行、データ全量移行/検証/修正、インクリメンタルリアルタイム同期を一体化した無料のデータ移行同期プラットフォームです。製品は完全な製品化機能を含み、企業がデータの孤島を打破し、データの融合と相互運用を実現し、データをより良く活用することを支援します。
![image.png](../assets/3.11-1.png)

## ダウンロードとインストール

[CloudCanal 最新版のダウンロードリンク](https://www.clougence.com)

[CloudCanal クイックスタート](https://www.clougence.com/cc-doc/quick/quick_start)

## 機能説明

- StarRocks への書き込みには、v2.2.5.0 以上の CloudCanal バージョンの使用を推奨します。
- CloudCanal を使用して **インクリメンタルデータ** を StarRocks にインポートする際は、インポート頻度を制御することをお勧めします。CloudCanal による StarRocks へのデフォルトのインポート頻度は、パラメータ `realFlushPauseSec` で調整可能で、デフォルトは 10 秒です。
- 現在のコミュニティ版の最大メモリ設定は 2GB です。同期タスクの実行中に OOM 例外が発生したり、GC 停止が深刻な場合は、以下のパラメータを小さくしてバッチサイズを減らし、メモリ使用量を減らすことができます。
  - 全量パラメータは `fullBatchSize` と `fullRingBufferSize` です。
  - インクリメンタルパラメータは `increBatchSize` と `increRingBufferSize` です。
- サポートされているソースと機能項目：
  
  | データソース \ 機能項目 | 構造移行 | 全量データ移行 | インクリメンタルリアルタイム同期 | データ検証 |
     | --- | --- | --- | --- | --- |
  | Oracle ソース            | 対応 | 対応 | 対応 | 対応 |
  | PostgreSQL ソース        | 対応 | 対応 | 対応 | 対応 |
  | Greenplum ソース         | 対応 | 対応 | 非対応 | 対応 |
  | MySQL ソース             | 対応 | 対応 | 対応 | 対応 |
  | Kafka ソース             | 非対応 | 非対応 | 対応 | 非対応 |
  | OceanBase ソース         | 対応 | 対応 | 対応 | 対応 |
  | PolarDB for MySQL ソース | 対応 | 対応 | 対応 | 対応 |
  | Db2 ソース               | 対応 | 対応 | 対応 | 対応 |
  
## 使用方法

CloudCanal は完全な製品化機能を提供し、ユーザーは視覚的なインターフェースでデータソースの追加とタスクの作成を完了するだけで、構造移行、全量移行、インクリメンタルリアルタイム同期が自動的に行われます。以下では、MySQL データベースから StarRocks へのデータ移行同期の方法を示します。他のソースから StarRocks への同期も同様の方法で行うことができます。

### 前提条件

まず、[CloudCanal クイックスタート](https://www.clougence.com/cc-doc/quick/quick_start) を参照して、CloudCanal コミュニティ版のインストールとデプロイを完了してください。

### データソースの追加

- CloudCanal プラットフォームにログイン
- データソース管理 -> 新規データソースの追加
- 自己構築のデータベースから StarRocks を選択

![image.png](../assets/3.11-2.png)

> ヒント：
>
> - Client アドレス：StarRocks が MySQL Client に提供するサービスポートで、CloudCanal はこれを使用してデータベースのメタデータ情報を照会します。
>
> - Http アドレス：Http アドレスは主に CloudCanal からのデータインポートリクエストを受け取るために使用されます。
>
> - アカウント：インポート操作には対象テーブルの INSERT 権限が必要です。ユーザーアカウントに INSERT 権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を参照してユーザーに権限を付与してください。

### タスクの作成

データソースを追加した後、以下の手順でデータ移行・同期タスクを作成できます。

- **タスク管理** -> **タスク作成**
- **ソース** と **ターゲット** データベースを選択
- 次へをクリック

![image.png](../assets/3.11-3.png)

- **インクリメンタル同期** を選択し、**全量データ初期化** を有効にする
- DDL 同期を選択
- 次へをクリック

![image.png](../assets/3.11-4.png)

- サブスクライブするテーブルを選択します。**構造移行で自動的に作成されるテーブルはプライマリキーモデルのテーブルであるため、プライマリキーのないテーブルはサポートされていません**
- 次へをクリック

![image.png](../assets/3.11-5.png)

- 列マッピングを設定
- 次へをクリック

![image.png](../assets/3.11-6.png)

- タスクを作成

![image.png](../assets/3.11-7.png)

- タスクの状態を確認します。タスクが作成されると、構造移行、全量、インクリメンタルの各フェーズが自動的に完了します。

![image.png](../assets/3.11-8.png)

## 参考資料

CloudCanal による StarRocks 同期に関する詳細は、以下を参照してください。

- [5 分で完了 PostgreSQL から StarRocks へのデータ移行同期 - CloudCanal 実践](https://www.askcug.com/topic/262)
