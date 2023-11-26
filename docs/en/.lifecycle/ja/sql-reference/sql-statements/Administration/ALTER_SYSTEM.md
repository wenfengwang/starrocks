---
displayed_sidebar: "Japanese"
---

# ALTER SYSTEM

## 説明

クラスタ内のFE、BE、CN、Brokerノード、およびメタデータスナップショットを管理します。

> **注意**
>
> `cluster_admin`ロールのみがこのSQLステートメントを実行する特権を持っています。

## 構文とパラメータ

### FE

- フォロワーFEを追加します。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいフォロワーFEのステータスは、`SHOW PROC '/frontends'\G`を実行して確認できます。

- フォロワーFEを削除します。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- オブザーバーFEを追加します。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいオブザーバーFEのステータスは、`SHOW PROC '/frontends'\G`を実行して確認できます。

- オブザーバーFEを削除します。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

| **パラメータ**    | **必須** | **説明**                                                                                                                                       |
| ---------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| fe_host          | Yes      | FEインスタンスのホスト名またはIPアドレスです。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用してください。 |
| edit_log_port    | Yes      | FEノードのBDB JE通信ポートです。デフォルト値は`9010`です。                                                                                     |

### BE

- BEノードを追加します。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいBEのステータスは、[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を実行して確認できます。

- BEノードを削除します。

  > **注意**
  >
  > シングルレプリカテーブルのタブレットを保持しているBEノードは削除できません。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- BEノードを廃止します。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  BEノードを削除するのとは異なり、BEノードを廃止すると、まずBE上のデータが他のBEに移行され、その後BEがクラスタから削除されます。データのロードやクエリはデータの移行中にも影響を受けません。[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を使用して操作が成功したかどうかを確認できます。操作が成功した場合、廃止されたBEは返されません。操作が失敗した場合、BEはオンラインのままです。[CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md)を使用して操作を手動でキャンセルできます。

| **パラメータ**          | **必須** | **説明**                                                                                                                                       |
| ---------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| be_host                | Yes      | BEインスタンスのホスト名またはIPアドレスです。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用してください。 |
| heartbeat_service_port | Yes      | BEハートビートサービスポートです。BEはこのポートを使用してFEからハートビートを受信します。デフォルト値は`9050`です。                           |

### CN

- CNノードを追加します。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいCNのステータスは、[SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md)を実行して確認できます。

- CNノードを削除します。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

| **パラメータ**          | **必須** | **説明**                                                                                                                                       |
| ---------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| cn_host                | Yes      | CNインスタンスのホスト名またはIPアドレスです。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用してください。 |
| heartbeat_service_port | Yes      | CNハートビートサービスポートです。CNはこのポートを使用してFEからハートビートを受信します。デフォルト値は`9050`です。                           |

### Broker

- Brokerノードを追加します。Brokerノードを使用して、HDFSやクラウドストレージからデータをStarRocksにロードすることができます。詳細については、[HDFSからデータをロードする](../../../loading/hdfs_load.md)または[クラウドストレージからデータをロードする](../../../loading/cloud_storage_load.md)を参照してください。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  1つのSQLで複数のBrokerノードを追加することができます。各`<broker_host>:<broker_ipc_port>`のペアは1つのBrokerノードを表し、共通の`broker_name`を共有します。新しいBrokerノードのステータスは、[SHOW BROKER](../Administration/SHOW_BROKER.md)を実行して確認できます。

- Brokerノードを削除します。

> **注意**
>
> Brokerノードを削除すると、現在実行中のタスクが終了します。

  - 同じ`broker_name`を持つ1つまたは複数のBrokerノードを削除します。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

  - 同じ`broker_name`を持つすべてのBrokerノードを削除します。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

| **パラメータ**   | **必須** | **説明**                                                                                                                                       |
| --------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| broker_name     | Yes      | Brokerノードの名前です。複数のBrokerノードで同じ名前を使用することができます。                                                                 |
| broker_host     | Yes      | Brokerインスタンスのホスト名またはIPアドレスです。インスタンスに複数のIPアドレスがある場合は、設定項目`priority_networks`の値を使用してください。 |
| broker_ipc_port | Yes      | Brokerノード上のThriftサーバーポートです。BrokerノードはFEまたはBEからのリクエストを受信するために使用します。デフォルト値は`8000`です。       |

### イメージの作成

イメージファイルを作成します。イメージファイルはFEメタデータのスナップショットです。

```SQL
ALTER SYSTEM CREATE IMAGE
```

イメージの作成は、リーダーFE上で非同期に行われます。操作の開始時間と終了時間は、FEログファイル**fe.log**で確認できます。`triggering a new checkpoint manually...`のようなログは、イメージの作成が開始されたことを示し、`finished save image...`のようなログは、イメージが作成されたことを示します。

## 使用上の注意

- FE、BE、CN、またはBrokerノードの追加および削除は同期的な操作です。ノードの削除操作をキャンセルすることはできません。
- シングルFEクラスタでは、FEノードを削除することはできません。
- マルチFEクラスタでは、リーダーFEノードを直接削除することはできません。削除するには、まずリーダーFEを再起動する必要があります。StarRocksが新しいリーダーFEを選出した後、前のリーダーFEを削除できます。
- BEノードの数がデータのレプリカ数よりも少ない場合、BEノードを削除することはできません。たとえば、クラスタに3つのBEノードがあり、データを3つのレプリカに保存している場合、BEノードをいずれも削除することはできません。また、4つのBEノードと3つのレプリカがある場合、1つのBEノードを削除することができます。
- BEノードを削除すると、StarRocksはBEノードをクラスタから強制的に削除し、削除後に削除されたタブレットを補完します。一方、BEノードを廃止すると、StarRocksはまず廃止されたBEノード上のタブレットを他のBEに移行し、その後ノードを削除します。

## 例

例1: フォロワーFEノードを追加します。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

例2: 2つのオブザーバーFEノードを同時に削除します。

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

例5: 2つのBEノードを同時に廃止します。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例6: 同じ`broker_name`で2つのBrokerノードを追加します - `hdfs`。

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
