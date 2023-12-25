---
displayed_sidebar: English
---

# デプロイ後のセットアップ

このトピックでは、StarRocks のデプロイ後に実行する必要があるタスクについて説明します。

新しい StarRocks クラスターを本番環境に移行する前に、初期アカウントを保護し、クラスターを適切に実行するために必要な変数とプロパティを設定する必要があります。

## 初期アカウントのセキュリティ

StarRocks クラスタが作成されると、クラスタの初期 `root` ユーザーが自動的に生成されます。`root` ユーザーには `root` 権限が付与され、これはクラスター内のすべての権限の集合です。誤用を防ぐために、このユーザーアカウントを保護し、本番環境での使用を避けることを推奨します。

StarRocks は、クラスタの作成時に `root` ユーザーに自動的に空のパスワードを割り当てます。以下の手順に従って、`root` ユーザーの新しいパスワードを設定してください：

1. ユーザー名 `root` と空のパスワードを使用して、MySQL クライアント経由で StarRocks に接続します。

   ```Bash
   # <fe_address> を接続する FE ノードの IP アドレス（priority_networks）または FQDN に置き換え、
   # <query_port> を fe.conf で指定した query_port（デフォルト：9030）に置き換えます。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次の SQL を実行して、`root` ユーザーのパスワードをリセットします。

   ```SQL
   -- <password> を root ユーザーに割り当てたいパスワードに置き換えます。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注記**
>
> - パスワードをリセットした後は、パスワードを適切に保管してください。パスワードを忘れた場合は、[紛失した root パスワードのリセット](../administration/User_privilege.md#reset-lost-root-password)で詳細な手順を参照してください。
> - デプロイ後のセットアップが完了したら、新しいユーザーとロールを作成して、チーム内の権限を管理できます。詳細な手順については、[ユーザー権限の管理](../administration/User_privilege.md)を参照してください。

## 必要なシステム変数の設定

StarRocks クラスタを本番環境で正しく動作させるためには、以下のシステム変数を設定する必要があります。

| **変数名**                          | **StarRocks バージョン** | **推奨値**                                        | **説明**                                                      |
| ----------------------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| is_report_success                   | v2.4 以前             | false                                                        | クエリのプロファイルを分析用に送信するかどうかを制御するブール型スイッチです。デフォルト値は `false` で、プロファイルは不要です。この変数を `true` に設定すると、StarRocks の同時実行性に影響を与える可能性があります。 |
| enable_profile                      | v2.5 以降             | false                                                        | クエリのプロファイルを分析用に送信するかどうかを制御するブール型スイッチです。デフォルト値は `false` で、プロファイルは不要です。この変数を `true` に設定すると、StarRocks の同時実行性に影響を与える可能性があります。 |
| enable_pipeline_engine              | v2.3 以降             | true                                                         | パイプライン実行エンジンを有効にするかどうかを制御するブール型スイッチです。`true` は有効、`false` は無効を示します。デフォルト値は `true` です。 |
| parallel_fragment_exec_instance_num | v2.3 以降             | パイプラインエンジンを有効にしている場合、この変数を `1` に設定できます。パイプラインエンジンを有効にしていない場合は、CPU コア数の半分に設定することを推奨します。 | 各 BE 上でノードをスキャンするために使用されるインスタンスの数です。デフォルト値は `1` です。 |
| pipeline_dop                        | v2.3、v2.4、および v2.5  | 0                                                            | パイプラインインスタンスの並列度を表す値で、クエリの同時実行性を調整するために使用されます。デフォルト値は `0` で、システムが各パイプラインインスタンスの並列度を自動的に調整します。<br />v3.0 以降、StarRocks はクエリの並列度に基づいてこのパラメータを適応的に調整します。 |

- `is_report_success` をグローバルに `false` に設定します。

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- `enable_profile` をグローバルに `false` に設定します。

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- `enable_pipeline_engine` をグローバルに `true` に設定します。

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- `parallel_fragment_exec_instance_num` をグローバルに `1` に設定します。

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- `pipeline_dop` をグローバルに `0` に設定します。

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

システム変数の詳細については、[システム変数](../reference/System_variable.md)を参照してください。

## ユーザープロパティの設定

クラスタに新しいユーザーを作成した場合は、そのユーザーの最大接続数を増やす必要があります（例：`1000`）。

```SQL
-- <username> を最大接続数を増やしたいユーザー名に置き換えます。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 次のステップ

StarRocks クラスターをデプロイして設定した後、シナリオに最適なテーブル設計に進むことができます。テーブル設計に関する詳細な手順については、[StarRocks テーブル設計の理解](../table_design/Table_design.md)を参照してください。
