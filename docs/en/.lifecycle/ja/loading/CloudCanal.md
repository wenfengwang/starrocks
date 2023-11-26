---
displayed_sidebar: "Japanese"
---

# CloudCanalを使用してデータをロードする

## はじめに

CloudCanal Community Editionは、[ClouGence株式会社](https://www.cloudcanalx.com)が公開している無料のデータ移行および同期プラットフォームであり、スキーマ移行、フルデータ移行、検証、修正、およびリアルタイムの増分同期を統合しています。
CloudCanalは、ユーザーが簡単な方法でモダンなデータスタックを構築するのに役立ちます。
![image.png](../assets/3.11-1.png)

## ダウンロード

[CloudCanalのダウンロードリンク](https://www.cloudcanalx.com)

[CloudCanalクイックスタート](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)

## 機能の説明

- StarRocksへの効率的なデータインポートには、CloudCanalバージョン2.2.5.0以上を使用することを強くお勧めします。
- CloudCanalを使用してStarRocksに**増分データ**をインポートする際には、取り込み頻度を制御することをお勧めします。CloudCanalからStarRocksにデータを書き込むためのデフォルトのインポート頻度は、デフォルトで10秒に設定されている`realFlushPauseSec`パラメータを調整することができます。
- 現在のコミュニティエディションでは、最大メモリ構成が2GBの場合、DataJobsがOOM例外や大きなGC停止を発生させる場合は、バッチサイズを減らしてメモリ使用量を最小限に抑えることをお勧めします。
  - フルデータタスクの場合、`fullBatchSize`および`fullRingBufferSize`パラメータを調整することができます。
  - 増分データタスクの場合、`increBatchSize`および`increRingBufferSize`パラメータを適切に調整することができます。
- サポートされているソースエンドポイントと機能：

  | ソースエンドポイント \ 機能 | スキーマ移行 | フルデータ | 増分データ | 検証 |
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

CloudCanalを使用すると、ユーザーはビジュアルインターフェースで操作を行い、ビジュアルインターフェースを介してDataSourcesをシームレスに追加し、DataJobsを作成することができます。これにより、自動スキーマ移行、フルデータ移行、およびリアルタイムの増分同期が可能になります。以下の例では、MySQLからStarRocksへのデータの移行と同期方法を示しています。手順は、他のデータソースとStarRocks間のデータ同期にも同様です。

### 前提条件

まず、[CloudCanalクイックスタート](https://www.cloudcanalx.com/us/cc-doc/quick/quick_start)を参照して、CloudCanal Community Editionのインストールと展開を完了してください。

### DataSourceの追加

- CloudCanalプラットフォームにログインします
- **DataSource Management** -> **Add DataSource**に移動します
- オプションの中から**StarRocks**を選択します

![image.png](../assets/3.11-2.png)

> ヒント：
>
> - クライアントアドレス：StarRocksサーバーのMySQLクライアントサービスポートのアドレスです。CloudCanalは、主にこのアドレスを使用してデータベーステーブルのメタデータ情報をクエリします。
>
> - HTTPアドレス：HTTPアドレスは、CloudCanalからのデータインポートリクエストを受け取るために主に使用されます。

### DataJobの作成

DataSourceが正常に追加されたら、次の手順に従ってデータ移行および同期のDataJobを作成できます。

- CloudCanalで**DataJob Management** -> **Create DataJob**に移動します
- DataJobのソースとターゲットデータベースを選択します
- 次のステップをクリックします

![image.png](../assets/3.11-3.png)

- **増分データ**を選択し、**フルデータ**を有効にします
- DDL Syncを選択します
- 次のステップをクリックします

![image.png](../assets/3.11-4.png)

- 購読するソーステーブルを選択します。スキーマ移行後に自動的に作成されるStarRocksテーブルは、主キーテーブルですので、主キーのないソーステーブルは現在サポートされていません**

- 次のステップをクリックします

![image.png](../assets/3.11-5.png)

- カラムマッピングを設定します
- 次のステップをクリックします

![image.png](../assets/3.11-6.png)

- DataJobを作成します

![image.png](../assets/3.11-7.png)

- DataJobのステータスを確認します。DataJobは作成された後、自動的にスキーマ移行、フルデータ、および増分のステージを経ます

![image.png](../assets/3.11-8.png)
