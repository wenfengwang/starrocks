---
displayed_sidebar: English
---

# CloudCanal を使用してデータを読み込む

## はじめに

CloudCanal Community Editionは、[ClouGence Co., Ltd](https://www.cloudcanalx.com)が提供する無料のデータ移行および同期プラットフォームで、スキーマ移行、完全データ移行、検証、修正、リアルタイム増分同期を統合しています。
CloudCanalは、ユーザーがシンプルな方法でモダンなデータスタックを構築するのに役立ちます。
![image.png](../assets/3.11-1.png)

## ダウンロード

[CloudCanal ダウンロードリンク](https://www.cloudcanalx.com)

[CloudCanal クイックスタート](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 機能説明

- StarRocksへの効率的なデータインポートには、CloudCanalバージョン2.2.5.0以上の利用を強く推奨します。
- CloudCanalを使用してStarRocksに**インクリメンタルデータ**をインポートする際は、取り込み頻度をコントロールすることが望ましいです。CloudCanalからStarRocksへのデータ書き込みのデフォルトインポート頻度は、デフォルトで10秒に設定されている`realFlushPauseSec`パラメータを使用して調整可能です。
- 最大メモリ設定が2GBの現在のコミュニティエディションでは、DataJobsがOOM例外に遭遇したり、GCが大幅に停止したりする場合、バッチサイズを減らしてメモリ使用量を最小限に抑えることを推奨します。
  - Full DataTaskの場合、`fullBatchSize`と`fullRingBufferSize`パラメータを調整できます。
  - Incremental DataTaskの場合、`increBatchSize`と`increRingBufferSize`パラメータをそれぞれ調整できます。
- サポートされるソースエンドポイントと機能：

  | ソースエンドポイント \ 機能 | スキーマ移行 | 完全データ | インクリメンタル | 検証 |
    | --- | --- | --- | --- | --- |
  | Oracle                     | はい | はい | はい | はい |
  | PostgreSQL                 | はい | はい | はい | はい |
  | Greenplum                  | はい | はい | いいえ | はい |
  | MySQL                      | はい | はい | はい | はい |
  | Kafka                      | いいえ | いいえ | はい | いいえ |
  | OceanBase                  | はい | はい | はい | はい |
  | PolarDB for MySQL          | はい | はい | はい | はい |
  | Db2                        | はい | はい | はい | はい |

## 典型的な例

CloudCanalを使用すると、ユーザーはビジュアルインターフェースで操作を行い、データソースを追加し、データジョブを作成することができます。これにより、スキーマ移行、完全データ移行、リアルタイム増分同期が自動化されます。以下の例では、MySQLからStarRocksへのデータ移行と同期の方法を示しています。他のデータソースとStarRocks間のデータ同期も同様の手順です。

### 前提条件

まず、[CloudCanal クイックスタート](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)を参照し、CloudCanal Community Editionのインストールとデプロイを完了させてください。

### データソースを追加する

- CloudCanalプラットフォームにログインします
- **データソース管理** -> **データソースを追加**に進みます
- 自己構築データベースのオプションから**StarRocks**を選択します

![image.png](../assets/3.11-2.png)

> ヒント：
>
> - クライアントアドレス：StarRocksサーバーのMySQLクライアントサービスポートのアドレスです。CloudCanalは主にこのアドレスを使用してデータベーステーブルのメタデータ情報をクエリします。
>
> - HTTPアドレス：HTTPアドレスは主にCloudCanalからのデータインポートリクエストを受け取るために使用されます。

### データジョブを作成する

データソースが正常に追加された後、以下の手順でデータ移行と同期のデータジョブを作成できます。

- CloudCanalで**データジョブ管理** -> **データジョブを作成**に進みます
- DataJobのソースデータベースとターゲットデータベースを選択します
- 「次へ」をクリックします

![image.png](../assets/3.11-3.png)

- **インクリメンタル**を選択し、**フルデータ**を有効にします
- DDL同期を選択します
- 「次へ」をクリックします

![image.png](../assets/3.11-4.png)

- サブスクライブしたいソーステーブルを選択します。スキーマ移行後に自動的に作成されるStarRocksのターゲットテーブルはプライマリキーテーブルであるため、プライマリキーがないソーステーブルは現在サポートされていません**

- 「次へ」をクリックします

![image.png](../assets/3.11-5.png)

- カラムマッピングを設定します
- 「次へ」をクリックします

![image.png](../assets/3.11-6.png)

- データジョブを作成します

![image.png](../assets/3.11-7.png)

- データジョブの状態を確認します。データジョブは作成後、自動的にスキーマ移行、フルデータ、インクリメンタルの各ステージを経ます

![image.png](../assets/3.11-8.png)
