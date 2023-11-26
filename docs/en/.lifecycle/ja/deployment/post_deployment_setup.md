---
displayed_sidebar: "Japanese"
---

# デプロイ後のセットアップ

このトピックでは、StarRocksをデプロイした後に実行する必要のあるタスクについて説明します。

新しいStarRocksクラスタを本番環境で使用する前に、初期アカウントをセキュアにし、クラスタが正常に動作するために必要な変数とプロパティを設定する必要があります。

## 初期アカウントのセキュア化

StarRocksクラスタが作成されると、クラスタの初期`root`ユーザが自動的に生成されます。`root`ユーザには、クラスタ内のすべての権限が含まれる`root`特権が付与されます。このユーザアカウントをセキュアにし、誤用を防ぐために本番環境で使用しないようにすることをお勧めします。

StarRocksは、クラスタが作成されると`root`ユーザに空のパスワードを自動的に割り当てます。`root`ユーザの新しいパスワードを設定するために、次の手順に従ってください。

1. MySQLクライアントを使用して、ユーザ名`root`と空のパスワードでStarRocksに接続します。

   ```Bash
   # <fe_address>を接続するFEノードのIPアドレス（priority_networks）またはFQDNに置き換え、<query_port>をfe.confで指定したquery_port（デフォルト：9030）に置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行して、`root`ユーザのパスワードをリセットします。

   ```SQL
   -- <password>をrootユーザに割り当てるパスワードに置き換えてください。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注意**
>
> - パスワードをリセットした後は、パスワードを適切に保管してください。パスワードを忘れた場合は、詳細な手順については[失われたrootパスワードのリセット](../administration/User_privilege.md#reset-lost-root-password)を参照してください。
> - デプロイ後のセットアップが完了した後、チーム内で特権を管理するために新しいユーザとロールを作成することができます。詳細な手順については、[ユーザ特権の管理](../administration/User_privilege.md)を参照してください。

## 必要なシステム変数の設定

StarRocksクラスタが正常に動作するためには、以下のシステム変数を設定する必要があります。

| **変数名**                           | **StarRocksバージョン** | **推奨値**                                                   | **説明**                                                     |
| ----------------------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| is_report_success                   | v2.4以前               | false                                                        | クエリのプロファイルを解析するかどうかを制御するブールスイッチです。デフォルト値は`false`で、プロファイルは必要ありません。この変数を`true`に設定すると、StarRocksの並行性に影響を与える場合があります。 |
| enable_profile                      | v2.5以降               | false                                                        | クエリのプロファイルを解析するかどうかを制御するブールスイッチです。デフォルト値は`false`で、プロファイルは必要ありません。この変数を`true`に設定すると、StarRocksの並行性に影響を与える場合があります。 |
| enable_pipeline_engine              | v2.3以降               | true                                                         | パイプライン実行エンジンを有効にするかどうかを制御するブールスイッチです。`true`は有効を示し、`false`は無効を示します。デフォルト値：`true`。 |
| parallel_fragment_exec_instance_num | v2.3以降               | パイプラインエンジンを有効にしている場合は`1`に設定します。パイプラインエンジンを有効にしていない場合は、CPUコア数の半分に設定する必要があります。 | 各BEでノードをスキャンするために使用されるインスタンスの数です。デフォルト値は`1`です。 |
| pipeline_dop                        | v2.3、v2.4、およびv2.5  | 0                                                            | パイプラインインスタンスの並列性であり、クエリの並行性を調整するために使用されます。デフォルト値：`0`で、システムは各パイプラインインスタンスの並列性を自動的に調整します。<br />v3.0以降、StarRocksはクエリの並行性に基づいてこのパラメータを自動的に調整します。 |

- `is_report_success`をグローバルに`false`に設定します：

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- `enable_profile`をグローバルに`false`に設定します：

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- `enable_pipeline_engine`をグローバルに`true`に設定します：

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- `parallel_fragment_exec_instance_num`をグローバルに`1`に設定します：

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- `pipeline_dop`をグローバルに`0`に設定します：

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

システム変数の詳細については、[システム変数](../reference/System_variable.md)を参照してください。

## ユーザプロパティの設定

クラスタで新しいユーザを作成した場合、そのユーザの最大接続数を拡大する必要があります（例：`1000`）。

```SQL
-- <username>を最大接続数を拡大するユーザ名に置き換えてください。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 次に行うこと

StarRocksクラスタをデプロイしてセットアップした後、シナリオに最適なテーブルを設計することができます。詳細な手順については、[StarRocksテーブル設計の理解](../table_design/Table_design.md)を参照してください。
