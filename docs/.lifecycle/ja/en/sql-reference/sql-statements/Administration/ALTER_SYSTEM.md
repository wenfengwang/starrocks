---
displayed_sidebar: English
---

# ALTER SYSTEM

## 説明

クラスター内のFE、BE、CN、Brokerノード、およびメタデータスナップショットを管理します。

> **注記**
>
> この操作を実行できるのは`cluster_admin`ロールのみです。

## 構文とパラメータ

### FE

- Follower FEを追加します。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいFollower FEの状態を`SHOW PROC '/frontends'\G`を実行して確認できます。

- Follower FEを削除します。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- Observer FEを追加します。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいObserver FEのステータスは`SHOW PROC '/frontends'\G`を実行することで確認できます。

- Observer FEを削除します。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

| **パラメータ**      | **必須** | **説明**                                                     |
| ------------------ | ------------ | ------------------------------------------------------------------- |
| fe_host            | はい          | FEインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用します。|
| edit_log_port      | はい          | FEノードのBDB JE通信ポート。デフォルト: `9010`。          |

### BE

- BEノードを追加します。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいBEの状態は[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を実行して確認できます。

- BEノードを削除します。

  > **注記**
  >
  > 単一レプリカテーブルのタブレットを保持するBEノードは削除できません。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- BEノードをデコミッションします。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  BEノードを削除することはクラスタから強制的に除去することですが、デコミッションすることは安全に除去することを意味します。これは非同期操作です。BEがデコミッションされると、まずそのBE上のデータが他のBEに移行され、その後BEがクラスタから除去されます。データのロードとクエリはデータ移行中に影響を受けません。操作が成功したかどうかは[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を使用して確認できます。操作が成功すれば、デコミッションされたBEは戻りません。操作が失敗した場合、BEはオンラインのままです。[CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md)を使用して操作を手動でキャンセルできます。

| **パラメータ**          | **必須** | **説明**                                                                            |
| ---------------------- | ------------ | ------------------------------------------------------------------------------------------ |
| be_host                | はい          | BEインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用します。|
| heartbeat_service_port | はい          | BEハートビートサービスポート。BEはこのポートを使用してFEからハートビートを受信します。デフォルト: `9050`。|

### CN

- CNノードを追加します。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいCNの状態は[SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md)を実行して確認できます。

- CNノードを削除します。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

> **注記**
>
> `ALTER SYSTEM DECOMMISSION`コマンドを使用してCNノードをデコミッションすることはできません。

| **パラメータ**          | **必須** | **説明**                                                                            |
| ---------------------- | ------------ | ------------------------------------------------------------------------------------------ |
| cn_host                | はい          | CNインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用します。|
| heartbeat_service_port | はい          | CNハートビートサービスポート。CNはこのポートを使用してFEからハートビートを受信します。デフォルト: `9050`。|

### Broker

- Brokerノードを追加します。Brokerノードを使用して、HDFSまたはクラウドストレージからStarRocksにデータをロードすることができます。詳細は[HDFSからデータをロードする](../../../loading/hdfs_load.md)または[クラウドストレージからデータをロードする](../../../loading/cloud_storage_load.md)を参照してください。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  1つのSQLで複数のBrokerノードを追加できます。各`<broker_host>:<broker_ipc_port>`ペアは1つのBrokerノードを表し、共通の`broker_name`を共有します。新しいBrokerノードの状態は[SHOW BROKER](../Administration/SHOW_BROKER.md)を実行して確認できます。

- Brokerノードを削除します。

> **注意**
>
> Brokerノードを削除すると、そのノードで現在実行中のタスクが終了します。

  - 同じ`broker_name`を持つ1つまたは複数のBrokerノードを削除します。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

  - 同じ`broker_name`を持つすべてのBrokerノードを削除します。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

| **パラメータ**   | **必須** | **説明**                                                              |
| --------------- | ------------ | ---------------------------------------------------------------------------- |
| broker_name     | はい          | Brokerノードの名前。複数のBrokerノードが同じ名前を使用できます。 |
| broker_host     | はい          | Brokerインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用します。|
| broker_ipc_port | はい          | BrokerノードのThriftサーバーポート。Brokerノードはこれを使用してFEまたはBEからのリクエストを受信します。デフォルト: `8000`。 |

### イメージの作成

イメージファイルを作成します。イメージファイルはFEメタデータのスナップショットです。

```SQL
ALTER SYSTEM CREATE IMAGE
```

イメージの作成はLeader FE上での非同期操作です。操作の開始時刻と終了時刻はFEログファイル**fe.log**で確認できます。`triggering a new checkpoint manually...`というログはイメージ作成が開始されたことを示し、`finished save image...`というログはイメージが作成されたことを示します。

## 使用上の注意

- FE、BE、CN、Brokerノードの追加と削除は同期操作です。ノードの削除操作をキャンセルすることはできません。
- 単一FEクラスタのFEノードを削除することはできません。
- マルチFEクラスタでLeader FEノードを直接削除することはできません。削除するには、まず再起動する必要があります。StarRocksが新しいLeader FEを選出した後、前のLeader FEを削除できます。
- データレプリカの数よりも残りのBEノードの数が少ない場合、BEノードを削除することはできません。例えば、クラスタに3つのBEノードがあり、データを3つのレプリカで保持している場合、どのBEノードも削除できません。4つのBEノードと3つのレプリカがある場合は、1つのBEノードを削除できます。
- BEノードを削除すると、StarRocksはそれをクラスタから強制的に除去し、削除後にドロップされたタブレットを補います。BEノードをデコミッションすると、StarRocksはまずデコミッションされたBEノード上のタブレットを他のノードに移行し、その後ノードを除去します。

## 例

例1: Follower FEノードを追加します。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

例2: 2つのObserver FEノードを同時に削除します。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010","x.x.x.x:9010";
```

例3: BEノードを追加します。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

例4: 2つのBEノードを同時に削除します。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例5: 2つのBEノードを同時にデコミッションします。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例6: 同じ`broker_name`を持つ2つのBrokerノードを追加します - `hdfs`。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

例7: `amazon_s3`から2つのBrokerノードを削除します。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

例8: `amazon_s3`のすべてのBrokerノードを削除します。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```
