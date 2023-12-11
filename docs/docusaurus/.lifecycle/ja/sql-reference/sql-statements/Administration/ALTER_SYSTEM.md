---
displayed_sidebar: "Japanese"
---

# ALTER SYSTEM（システム変更）

## 説明

クラスタ内のFE、BE、CN、ブローカーノード、およびメタデータスナップショットを管理します。

> **注意**
>
> `cluster_admin` ロールのみがこのSQLステートメントを実行する特権を持っています。

## 構文とパラメータ

### FE

- フォロワーFEを追加します。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいフォロワーFEの状態は、`SHOW PROC '/frontends'\G`を実行することで確認できます。

- フォロワーFEを削除します。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- オブザーバーFEを追加します。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいオブザーバーFEの状態は、`SHOW PROC '/frontends'\G`を実行することで確認できます。

- オブザーバーFEを削除します。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

| **パラメータ**      | **必須** | **説明**                                                                             |
| ------------------ | ------------ | ------------------------------------------------------------------------------------------ |
| fe_host            | Yes          | FEインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目 `priority_networks` の値を使用してください。 |
| edit_log_port      | Yes          | FEノードのBDB JE通信ポート。デフォルト: `9010`。          |

### BE

- BEノードを追加します。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいBEの状態は、[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を実行することで確認できます。

- BEノードを削除します。

  > **注意**
  >
  > 単一レプリカテーブルのタブレットを格納するBEノードは削除できません。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- BEノードを運用停止します。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  BEノードを削除するのとは異なり、BEを運用停止させてから削除します。非同期操作です。BEが運用停止になると、BE上のデータが他のBEに移行され、その後にBEがクラスタから削除されます。データのロードやクエリはデータ移行中に影響を受けません。操作が成功したかを[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を使用して確認できます。操作が成功すると、運用停止されたBEは返されません。操作に失敗した場合、BEはオンラインのままです。操作を手動でキャンセルするには[CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md)を使用できます。

| **パラメータ**          | **必須** | **説明**                                                                            |
| ---------------------- | ------------ | ------------------------------------------------------------------------------------------ |
| be_host                | Yes          | BEインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目 `priority_networks` の値を使用してください。|
| heartbeat_service_port | Yes          | BEハートビートサービスポート。BEはこれを使用してFEからハートビートを受信します。デフォルト: `9050`。|

### CN

- CNノードを追加します。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいCNの状態は、[SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md)を実行することで確認できます。

- CNノードを削除します。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

> **注意**
>
> `ALTER SYSTEM DECOMMISSION` コマンドを使用してCNノードを運用停止することはできません。

| **パラメータ**          | **必須** | **説明**                                                                            |
| ---------------------- | ------------ | ------------------------------------------------------------------------------------------ |
| cn_host                | Yes          | CNインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目 `priority_networks` の値を使用してください。|
| heartbeat_service_port | Yes          | CNハートビートサービスポート。CNはこれを使用してFEからハートビートを受信します。デフォルト: `9050`。|

### ブローカー

- ブローカーノードを追加します。ブローカーノードは、HDFSやクラウドストレージからStarRocksにデータをロードするために使用できます。詳細については、[HDFSからデータをロード](../../../loading/hdfs_load.md)または[クラウドストレージからデータをロード](../../../loading/cloud_storage_load.md)を参照してください。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  1つのSQLで複数のブローカーノードを追加できます。各 `<broker_host>:<broker_ipc_port>` ペアは1つのブローカーノードを表し、共通の `broker_name` を共有します。新しいブローカーノードの状態は、[SHOW BROKER](../Administration/SHOW_BROKER.md)を実行することで確認できます。

- ブローカーノードを削除します。

> **注意**
>
> ブローカーノードを削除すると、現在実行中のタスクが終了します。

  - 同じ `broker_name` の1つまたは複数のブローカーノードを削除します。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

  - 同じ `broker_name` のすべてのブローカーノードを削除します。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

| **パラメータ**   | **必須** | **説明**                                                              |
| --------------- | ------------ | ---------------------------------------------------------------------------- |
| broker_name     | Yes          | ブローカーノードの名前。複数のブローカーノードで同じ名前を使用できます。 |
| broker_host     | Yes          | ブローカーインスタンスのホスト名またはIPアドレス。インスタンスに複数のIPアドレスがある場合は、設定項目 `priority_networks` の値を使用してください。|
| broker_ipc_port | Yes          | ブローカーノード上のthriftサーバーポート。ブローカーノードは、これを使用してFEまたはBEからのリクエストを受信します。デフォルト: `8000`。 |

### 画像を作成

画像ファイルを作成します。画像ファイルはFEメタデータのスナップショットです。

```SQL
ALTER SYSTEM CREATE IMAGE
```

画像を作成することはLeader FE上で非同期操作です。操作の開始時間と終了時間はFEログファイル **fe.log** で確認できます。`triggering a new checkpoint manually...`のようなログは、画像の作成が開始されたことを示し、`finished save image...`のようなログは、画像が作成されたことを示します。

## 使用上の注意

- FE、BE、CN、またはブローカーノードの追加および削除は同期操作です。ノードの削除操作を取り消すことはできません。
- 単一のFEクラスタでは、FEノードを削除することはできません。
- マルチFEクラスタのリーダーFEノードは直接削除できません。削除するには、まずリーダーFEを再起動する必要があります。StarRocksが新しいリーダーFEを選出すると、以前のリーダーFEを削除できます。
- 残されたBEノードの数がデータのレプリカ数よりも少ない場合、BEノードを直接削除することはできません。例えば、クラスタに3つのBEノードがあり、データを3つのレプリカで保存している場合、BEノードを1つも削除できません。そして、4つのBEノードがあり、3つのレプリカでデータを保存している場合、1つのBEノードを削除できます。
- BEノードを削除および運用停止する違いは、BEノードを削除する場合、StarRocksがそれをクラスタから強制的に削除し、削除後に削除されたタブレットを補填することであり、BEノードを運用停止する場合、StarRocksがまず運用停止されたBEノードのタブレットを他のBEに移行し、その後ノードを削除します。

## 例

例1：フォロワーFEノードを追加する。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

例2：オブザーバーFEノードを2つ同時に削除する。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010","x.x.x.x:9010";
```

例3：BEノードを追加する。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

例4：BEノードを2つ同時に削除する。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例5：BEノードを2つ同時に運用停止する。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例6：同じ `broker_name` - `hdfs` のブローカーノード2つを追加する。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

例7：`amazon_s3` から2つのブローカーノードを削除する。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

例8：`amazon_s3` のすべてのブローカーノードを削除する。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```