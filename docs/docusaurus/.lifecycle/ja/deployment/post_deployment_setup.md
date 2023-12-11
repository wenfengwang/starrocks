---
displayed_sidebar: "Japanese"
---

# デプロイ後のセットアップ

このトピックでは、StarRocksをデプロイした後に実行する必要があるタスクについて説明します。

新しいStarRocksクラスタを本番環境で使用する前に、初期アカウントをセキュアにし、クラスタが正常に実行されるように必要な変数やプロパティを設定する必要があります。

## 初期アカウントをセキュアにする

StarRocksクラスタを作成すると、クラスタの初期`root`ユーザが自動的に生成されます。`root`ユーザには、クラスタ内のすべての権限が含まれる`root`権限が付与されます。このユーザアカウントをセキュアにし、誤用を防止するために本番環境で使用しないことをお勧めします。

StarRocksは、クラスタが作成されると`root`ユーザに空のパスワードが自動的に割り当てられます。`root`ユーザの新しいパスワードを設定するために、次の手順に従ってください。

1. MySQLクライアントを使用して`root`ユーザと空のパスワードでStarRocksに接続します。

   ```Bash
   # <fe_address>を接続するFEノードのIPアドレス（priority_networks）またはFQDNに、<query_port>をfe.confで指定したquery_port（デフォルト：9030）に置き換えてください。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行して、`root`ユーザのパスワードをリセットします。

   ```SQL
   -- <password>にrootユーザに割り当てたいパスワードを置き換えてください。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注意**
>
> - パスワードをリセットした後もパスワードを適切に管理してください。パスワードを忘れた場合は、詳しい手順については[失われたrootパスワードをリセットする](../administration/User_privilege.md#reset-lost-root-password)を参照してください。
> - デプロイ後のセットアップが完了した後、新しいユーザやロールを作成して、チーム内で権限を管理できます。テーブル内の権限を管理する詳しい手順については、[ユーザ権限の管理](../administration/User_privilege.md)を参照してください。

## 必要なシステム変数を設定する

本番環境でStarRocksクラスタが正常に動作するために、以下のシステム変数を設定する必要があります。

| **変数名**                           | **StarRocksバージョン** | **推奨値**                                                    | **説明**                                                      |
| ----------------------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| is_report_success                   | v2.4 以前             | false                                                        | クエリのプロファイルを解析するかどうかを制御するブールスイッチ。デフォルト値は `false` で、プロファイルは不要です。この変数を `true` に設定すると、StarRocksの並列性に影響を与える可能性があります。 |
| enable_profile                      | v2.5 以降             | false                                                        | クエリのプロファイルを解析するかどうかを制御するブールスイッチ。デフォルト値は `false` で、プロファイルは不要です。この変数を `true` に設定すると、StarRocksの並列性に影響を与える可能性があります。 |
| enable_pipeline_engine              | v2.3 以降             | true                                                         | パイプライン実行エンジンを有効にするかどうかを制御するブールスイッチ。`true` は有効、`false` は無効を示します。デフォルト値：`true`。 |
| parallel_fragment_exec_instance_num | v2.3 以降             | パイプラインエンジンを有効にした場合、この変数を `1` に設定できます。パイプラインエンジンを有効にしていない場合は、CPUコア数の半分に設定する必要があります。 | 各BEでノードをスキャンするために使用されるインスタンスの数。デフォルト値は `1` です。 |
| pipeline_dop                        | v2.3、v2.4、およびv2.5  | 0                                                            | パイプラインインスタンスの並列性。この変数はクエリの並列性を調整するために使用されます。デフォルト値：`0`、これはシステムが各パイプラインインスタンスの並列性を自動的に調整することを示します。<br />v3.0以降、StarRocksはこのパラメータをクエリの並列性に基づいて適応的に調整します。 |

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

システム変数についての詳細情報は、[システム変数](../reference/System_variable.md)を参照してください。

## ユーザプロパティを設定する

クラスタに新しいユーザを作成した場合は、そのユーザの最大接続数を拡大する必要があります（例： `1000`）：

```SQL
-- <username>に最大接続数を拡大したいユーザ名を置き換えてください。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 次に何をするか

StarRocksクラスタをデプロイしてセットアップした後、シナリオに最適なテーブルを設計するために進むことができます。テーブルの設計についての詳しい手順については、[StarRocksテーブル設計の理解](../table_design/Table_design.md)を参照してください。