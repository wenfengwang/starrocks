---
displayed_sidebar: "Japanese"
---

# ALTER SYSTEM (システムの変更)

## 説明

クラスター内のFE、BE、CN、ブローカーノード、およびメタデータのスナップショットを管理します。

> **注意**
>
> `cluster_admin`ロールのみがこのSQLステートメントを実行する特権を持っています。

## 構文とパラメータ

### FE

- フォロワーFEを追加します。

  ```SQL
  ALTER SYSTEM ADD FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいフォロワーFEの状態は、`SHOW PROC '/frontends'\G`を実行して確認できます。

- フォロワーFEを削除します。

  ```SQL
  ALTER SYSTEM DROP FOLLOWER "<fe_host>:<edit_log_port>"[, ...]
  ```

- オブザーバーFEを追加します。

  ```SQL
  ALTER SYSTEM ADD OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

  新しいオブザーバーFEの状態は、`SHOW PROC '/frontends'\G`を実行して確認できます。

- オブザーバーFEを削除します。

  ```SQL
  ALTER SYSTEM DROP OBSERVER "<fe_host>:<edit_log_port>"[, ...]
  ```

| **パラメータ**     | **必須** | **説明**                                                              |
| ------------------ | ------------ | ------------------------------------------------------------------- |
| fe_host            | Yes          | FEインスタンスのホスト名またはIPアドレス。複数のIPアドレスを持つ場合は、構成項目`priority_networks`の値を使用してください。 |
| edit_log_port      | Yes          | FEノードのBDB JE通信ポート。デフォルト：`9010`。                   |

### BE

- BEノードを追加します。

  ```SQL
  ALTER SYSTEM ADD BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいBEの状態は、[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を実行して確認できます。

- BEノードを削除します。

  > **注意**
  >
  > シングルレプリカテーブルのタブレットを保存しているBEノードは削除できません。

  ```SQL
  ALTER SYSTEM DROP BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

- BEノードを廃止します。

  ```SQL
  ALTER SYSTEM DECOMMISSION BACKEND "<be_host>:<heartbeat_service_port>"[, ...]
  ```

  BEを削除するのはクラスターから強制的に削除することを指し、BEを廃止することは安全に削除することを意味します。これは非同期操作です。BEが廃止されると、BE上のデータはまず他のBEに移行され、その後BEがクラスターから削除されます。データの読み込みやクエリはデータの移行中に影響を受けません。操作が成功したかどうかは[SHOW BACKENDS](../Administration/SHOW_BACKENDS.md)を使用して確認できます。操作が成功した場合、廃止されたBEは表示されません。操作が失敗した場合、BEはオンラインのままです。[CANCEL DECOMMISSION](../Administration/CANCEL_DECOMMISSION.md)を使用して操作を手動でキャンセルできます。

| **パラメータ**          | **必須** | **説明**                                                                            |
| ---------------------- | ------------ | ------------------------------------------------------------------------------------------ |
| be_host                | Yes          | BEインスタンスのホスト名またはIPアドレス。複数のIPアドレスを持つ場合は、構成項目`priority_networks`の値を使用してください。|
| heartbeat_service_port | Yes          | BEハートビートサービスポート。BEはこのポートをFEからのハートビート受信に使用します。デフォルト：`9050`。|

### CN

- CNノードを追加します。

  ```SQL
  ALTER SYSTEM ADD COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

  新しいCNの状態は、[SHOW COMPUTE NODES](../Administration/SHOW_COMPUTE_NODES.md)を実行して確認できます。

- CNノードを削除します。

  ```SQL
  ALTER SYSTEM DROP COMPUTE NODE "<cn_host>:<heartbeat_service_port>"[, ...]
  ```

> **注意**
>
> `ALTER SYSTEM DECOMMISSION`コマンドを使用してCNノードを廃止することはできません。

| **パラメータ**          | **必須** | **説明**                                                                            |
| ---------------------- | ------------ | ------------------------------------------------------------------------------------------ |
| cn_host                | Yes          | CNインスタンスのホスト名またはIPアドレス。複数のIPアドレスを持つ場合は、構成項目`priority_networks`の値を使用してください。|
| heartbeat_service_port | Yes          | CNハートビートサービスポート。CNはこのポートをFEからのハートビート受信に使用します。デフォルト：`9050`。|

### ブローカーノード

- ブローカーノードを追加します。ブローカーノードを使用してHDFSやクラウドストレージからデータをStarRocksにロードできます。詳細については、[HDFSからデータをロード](../../../loading/hdfs_load.md)または[クラウドストレージからデータをロード](../../../loading/cloud_storage_load.md)を参照してください。

  ```SQL
  ALTER SYSTEM ADD BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
  ```

  1つのSQLで複数のブローカーノードを追加できます。各`<broker_host>:<broker_ipc_port>`ペアは1つのブローカーノードを表し、共通の`broker_name`を共有します。新しいブローカーノードの状態は、[SHOW BROKER](../Administration/SHOW_BROKER.md)を実行して確認できます。

- ブローカーノードを削除します。

> **注意**
>
> ブローカーノードを削除すると、現在実行中のタスクが終了します。

  - 同じ`broker_name`の1つまたは複数のブローカーノードを削除します。

    ```SQL
    ALTER SYSTEM DROP BROKER <broker_name> "<broker_host>:<broker_ipc_port>"[, ...]
    ```

  - 同じ`broker_name`のすべてのブローカーノードを削除します。

    ```SQL
    ALTER SYSTEM DROP ALL BROKER <broker_name>
    ```

| **パラメータ**   | **必須** | **説明**                                                              |
| --------------- | ------------ | ---------------------------------------------------------------------------- |
| broker_name     | Yes          | ブローカーノードの名前。複数のブローカーノードが同じ名前を使用できます。 |
| broker_host     | Yes          | ブローカーインスタンスのホスト名またはIPアドレス。複数のIPアドレスを持つ場合は、構成項目`priority_networks`の値を使用してください。|
| broker_ipc_port | Yes          | ブローカーノード上のThriftサーバーポート。ブローカーノードは、FEまたはBEからのリクエストを受信するために使用します。デフォルト：`8000`。 |

### 画像の作成

画像ファイルを作成します。画像ファイルはFEメタデータのスナップショットです。

```SQL
ALTER SYSTEM CREATE IMAGE
```

画像の作成はリーダーFE上で非同期で実行されます。操作の開始時間と終了時間は、FEログファイル**fe.log**で確認できます。`triggering a new checkpoint manually...`というログは、画像が作成されたことを示し、`finished save image...`というログは画像が作成されたことを示します。

## 使用上の注意

- FE、BE、CN、またはブローカーノードの追加および削除は同期的な操作です。ノードの削除操作はキャンセルできません。
- シングルFEクラスターではFEノードを削除できません。
- マルチFEクラスターでは、直接リーダーFEノードを削除することはできません。削除するには、まず再起動する必要があります。StarRocksが新しいリーダーFEを選出した後、前のFEを削除できます。
- 残っているBEノードの数がデータレプリカの数よりも少ない場合、BEノードを直接削除できません。 たとえば、クラスターに3つのBEノードがあり、データを3つのレプリカに格納している場合、BEノードを削除できません。 4つのBEノードと3つのレプリカがある場合、1つのBEノードを削除できます。
- BEノードを削除するとBEをクラスターから強制的に削除し、削除後に削除されたタブレットを補完します。BEを廃止することとの違いは、BEノードを削除すると、StarRocksが直ちにクラスターから削除し、削除後のタブレットを補いますが、BEを廃止すると、StarRocksが最初に廃止されたBEノード上のタブレットを他のBEに移行し、その後ノードを削除します。

## 例

例1：フォロワーFEノードを追加します。

```SQL
ALTER SYSTEM ADD FOLLOWER "x.x.x.x:9010";
```

例2：2つのオブザーバーFEノードを同時に削除します。

```SQL
ALTER SYSTEM DROP OBSERVER "x.x.x.x:9010","x.x.x.x:9010";
```

例3：BEノードを追加します。

```SQL
ALTER SYSTEM ADD BACKEND "x.x.x.x:9050";
```

例4：2つのBEノードを同時に削除します。

```SQL
ALTER SYSTEM DROP BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例5：2つのBEノードを同時に廃止します。

```SQL
ALTER SYSTEM DECOMMISSION BACKEND "x.x.x.x:9050", "x.x.x.x:9050";
```

例6：`broker_name`が`hdfs`である2つのブローカーノードを追加します。

```SQL
ALTER SYSTEM ADD BROKER hdfs "x.x.x.x:8000", "x.x.x.x:8000";
```

例7：`amazon_s3`から2つのブローカーノードを削除します。

```SQL
ALTER SYSTEM DROP BROKER amazon_s3 "x.x.x.x:8000", "x.x.x.x:8000";
```

例8：`amazon_s3`のすべてのブローカーノードを削除します。

```SQL
ALTER SYSTEM DROP ALL BROKER amazon_s3;
```