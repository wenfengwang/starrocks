---
displayed_sidebar: "日本語"
---

# CloudCanalを使用してデータをロードする

## はじめに

CloudCanal コミュニティ版は、[ClouGence Co., Ltd](https://www.cloudcanalx.com) によって公開されている無料のデータ移行および同期プラットフォームであり、Schema Migration、Full Data Migration、検証、修正、およびリアルタイムのIncremental Synchronization を統合しています。
CloudCanal は、ユーザーが簡単な方法でモダンなデータスタックを構築するのを支援します。
![image.png](../assets/3.11-1.png)

## ダウンロード

[CloudCanal ダウンロードリンク](https://www.cloudcanalx.com)

[CloudCanalクイックスタート](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 機能説明

- StarRocks に効率的にデータをインポートするために、CloudCanal バージョン 2.2.5.0 以上を利用することを強くお勧めします。
- StarRocks への**インクリメンタルデータ**のインポート時には、データの書き込み頻度を制御することを推奨します。CloudCanal から StarRocks へのデータの書き込みのデフォルトの頻度は、`realFlushPauseSec` パラメータで調整できます。デフォルトでは、10秒に設定されています。
- 現在のコミュニティ版では、最大メモリ構成が2GBであり、DataJobs がOOM例外や大きなGC停止を遭遇した場合、メモリ使用量を最小限に抑えるためにバッチサイズを減らすことをお勧めします。
  - Full DataTask の場合、`fullBatchSize` および `fullRingBufferSize` パラメータを調整できます。
  - Incremental DataTask の場合、`increBatchSize` および `increRingBufferSize` パラメータをそれぞれ調整できます。
- サポートされているソースエンドポイントと機能：

  | ソースエンドポイント \ 機能 | Schema Migration | Full Data | Incremental | Verification |
    | --- | --- | --- | --- | --- |
  | Oracle                     | はい | はい | はい | はい |
  | PostgreSQL                 | はい | はい | はい | はい |
  | Greenplum                  | はい | はい | いいえ | はい |
  | MySQL                      | はい | はい | はい | はい |
  | Kafka                      | いいえ | いいえ | はい | いいえ |
  | OceanBase                  | はい | はい | はい | はい |
  | PolarDb for MySQL          | はい | はい | はい | はい |
  | Db2                        | はい | はい | はい | はい |

## 典型的な例

CloudCanal を使用すると、ユーザーはビジュアルインタフェースで操作を行うことができ、そこでユーザーはシームレスにDataSourcesを追加し、ビジュアルインタフェースを介してDataJobsを作成できます。これにより、自動的なスキーマ移行、フルデータ移行、およびリアルタイムのインクリメンタル同期が可能になります。以下の例は、MySQL から StarRocks へのデータの移行と同期方法を示しており、プロシージャは他のデータソースと StarRocks 間のデータ同期についても類似しています。

### 前提条件

まず、[CloudCanalクイックスタート](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start) を参照して、CloudCanalコミュニティ版のインストールと展開を完了してください。

### DataSourceの追加

- CloudCanalプラットフォームにログインします
- **DataSource Management** -> **Add DataSource** に移動します
- オプションから**StarRocks**を選択します

![image.png](../assets/3.11-2.png)

> ヒント：
>
> - クライアントアドレス: StarRocksサーバーのMySQLクライアントサービスポートのアドレスです。CloudCanalは主にこのアドレスを使用してデータベーステーブルのメタデータ情報をクエリします。
>
> - HTTPアドレス: HTTPアドレスは、CloudCanalからのデータインポートリクエストを受信するために主に使用されます。

### DataJobの作成

DataSourceが正常に追加されたら、データ移行および同期のDataJobを作成するために次の手順に従ってください。

- CloudCanalに移動し、**DataJob Management** -> **Create DataJob** に移動します
- DataJobのソースおよびターゲットデータベースを選択します
- 次のステップをクリックします

![image.png](../assets/3.11-3.png)

- **Incremental**を選択し、**Full Data**を有効にします
- DDL Syncを選択します
- 次のステップをクリックします

![image.png](../assets/3.11-4.png)

- 購読するソーステーブルを選択します。スキーマ移行後のターゲットのStarRocksテーブルは主キーテーブルなので、主キーのないソーステーブルは現在サポートされていません**

- 次のステップをクリックします

![image.png](../assets/3.11-5.png)

- カラムマッピングを構成します
- 次のステップをクリックします

![image.png](../assets/3.11-6.png)

- DataJobを作成します

![image.png](../assets/3.11-7.png)

- DataJobのステータスを確認します。DataJobは作成されると自動的にスキーマ移行、フルデータ、およびインクリメンタルの段階を経ていきます

![image.png](../assets/3.11-8.png)