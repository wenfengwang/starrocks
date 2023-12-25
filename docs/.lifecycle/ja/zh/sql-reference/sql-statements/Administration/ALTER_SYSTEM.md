---
displayed_sidebar: Chinese
---

# ALTER SYSTEM

## 機能

StarRocks クラスター内の FE、BE、CN、Broker を管理します。

> **注意**
>
> `cluster_admin` ロールを持つユーザーのみがクラスター管理関連の操作を実行できます。

## 文法とパラメータ説明

### FE

- Follower FE を追加します。追加後、`SHOW PROC '/frontends'\G` コマンドで新しい FE の状態を確認できます。

   ```SQL
    ALTER SYSTEM ADD FOLLOWER "host:edit_log_port"[, ...]
    ```

- Follower FE を削除します。

    ```SQL
    ALTER SYSTEM DROP FOLLOWER "host:edit_log_port"[, ...]
    ```

- Observer FE を追加します。追加後、`SHOW PROC '/frontends'\G` コマンドで新しい FE の状態を確認できます。

    ```SQL
    ALTER SYSTEM ADD OBSERVER "host:edit_log_port"[, ...]
    ```

- Observer FE を削除します。

    ```SQL
    ALTER SYSTEM DROP OBSERVER "host:edit_log_port"[, ...]
    ```

     パラメータの説明は以下の通りです：

    | **パラメータ**       | **必須** | **説明**                                                     |
    | ------------------ | -------- | ------------------------------------------------------------ |
    | host:edit_log_port | はい       | <ul><li>`host`：FE マシンの FQDN または IP アドレス。マシンに複数の IP アドレスがある場合、このパラメータの値は `priority_networks` 設定項目で指定された唯一の通信 IP アドレスである必要があります。</li><li>`edit_log_port`：FE の BDBJE 通信ポートで、デフォルトは `9010` です。</li></ul> |

- image を作成します。

    ```SQL
    ALTER SYSTEM CREATE IMAGE
    ```

    このステートメントを実行すると、Leader FE が新しい Image（メタデータスナップショット）ファイルの作成をトリガーします。この操作は非同期で実行され、FE のログファイル **fe.log** を確認することで、いつ操作が開始され終了したかを確認できます。`triggering a new checkpoint manually...` は操作が開始されたことを示し、`finished save image...` は Image の作成が完了したことを示します。

### BE

- BE を追加します。追加後、[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md) で新しい BE の状態を確認できます。

    ```SQL
    ALTER SYSTEM ADD BACKEND "host:heartbeat_service_port"[, ...]
    ```

- BE を削除します。単一レプリカのテーブルがあり、そのテーブルの一部のタブレットが削除される BE 上に分布している場合、その BE を削除することはできません。

    ```SQL
    ALTER SYSTEM DROP BACKEND "host:heartbeat_service_port"[, ...]
    ```

- BE をデコミッションします。

    ```SQL
    ALTER SYSTEM DECOMMISSION BACKEND "host:heartbeat_service_port"[, ...]
    ```

    デコミッションする前に、その BE 上のデータは他の BE に移行され、データのインポートとクエリに影響を与えません。BE のデコミッションは非同期操作であり、[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md) ステートメントでデコミッションが成功したかどうかを確認できます。成功した場合、その BE は SHOW BACKENDS の返す情報には表示されません。デコミッション操作を手動でキャンセルすることができます。詳細は [CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md) を参照してください。

    パラメータの説明は以下の通りです：

    | **パラメータ**                | **必須** | **説明**                                                     |
    | --------------------------- | -------- | ------------------------------------------------------------ |
    | host:heartbeat_service_port | はい       |<ul><li> `host`：BE マシンの FQDN または IP アドレス。マシンに複数の IP アドレスがある場合、このパラメータの値は `priority_networks` 設定項目で指定された唯一の通信 IP アドレスである必要があります。</li><li>`heartbeat_service_port`：BE のハートビートポートで、FE からのハートビートを受信するために使用され、デフォルトは `9050` です。</li></ul> |

### CN

- CN を追加します。追加後、`SHOW PROC '/compute_nodes'\G` で新しい CN の状態を確認できます。

    ```SQL
    ALTER SYSTEM ADD COMPUTE NODE "host:heartbeat_service_port"[, ...]
    ```

- CN を削除します。

    ```SQL
    ALTER SYSTEM DROP COMPUTE NODE "host:heartbeat_service_port"[, ...]
    ```

> **説明**
>
> CN は `ALTER SYSTEM DECOMMISSION` コマンドを使用してデコミッションすることはできません。

    パラメータの説明は以下の通りです：

    | **パラメータ**                | **必須** | **説明**                                                     |
    | --------------------------- | -------- | ------------------------------------------------------------ |
    | host:heartbeat_service_port | はい       |<ul><li> `host`：CN マシンの FQDN または IP アドレス。マシンに複数の IP アドレスがある場合、このパラメータの値は `priority_networks` 設定項目で指定された唯一の通信 IP アドレスである必要があります。</li><li>`heartbeat_service_port`：CN のハートビートポートで、FE からのハートビートを受信するために使用され、デフォルトは `9050` です。</li></ul> |

### Broker

- Broker を追加します。追加後、HDFS や外部クラウドストレージシステムから StarRocks へのデータのインポートに Broker Load を使用できます。詳細は [HDFS からのインポート](../../../loading/hdfs_load.md) または [クラウドストレージからのインポート](../../../loading/cloud_storage_load.md) を参照してください。

    ```SQL
    ALTER SYSTEM ADD BROKER broker_name "host:port"[, ...]
    ```

    一つの SQL ステートメントで複数の Broker（一つの `host:port` が一つの Broker）を同時に追加する場合、これらの Broker は同じ `broker_name` を共有します。追加後、[SHOW BROKER](../Administration/SHOW_BROKER.md) ステートメントで Broker の詳細情報を確認できます。

- Broker を削除します。Broker 上で実行中のインポートタスクがある場合、その Broker を削除するとタスクが中断されることに注意してください。

  - `broker_name` の下の一つまたは複数の Broker を削除します。

      ```SQL
      ALTER SYSTEM DROP BROKER broker_name "host:broker_ipc_port"[, ...]
      ```

  - `broker_name` の下のすべての Broker を削除します。

      ```SQL
      ALTER SYSTEM DROP ALL BROKER broker_name
      ```

     パラメータの説明は以下の通りです：

    | **パラメータ**             | **必須** | **説明**                                                     |
    | -------------------- | -------- | ------------------------------------------------------------ |
    | broker_name          | はい       | 一つの Broker の名称、または複数の Broker が共有する名称。                 |
    | host:broker_ipc_port | はい       | <ul><li>`host`：Broker マシンの FQDN または IP アドレス。</li><li>`broker_ipc_port`：Broker の thrift server ポートで、FE または BE からのリクエストを受けるために使用され、デフォルトは `8000` です。</li></ul> |

## 使用説明

FE、BE、Broker の追加と削除は同期操作です。削除ステートメントを実行すると、FE、BE、Broker は直ちに削除され、その操作は手動で取り消すことはできません。

## 例

例1：Follower FE を追加します。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

例2：同時に二つの Observer FE を削除します。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010","x.x.x.x:9010";
```

例3：BE を追加します。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

例4：同時に二つの BE を削除します。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例5：同時に二つの BE をデコミッションします。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例6：同時に二つの `hdfs` という名前の Broker を追加します。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

例7：`amazon_s3` の下の二つの Broker を削除します。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

例8：`amazon_s3` の下のすべての Broker を削除します。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```
