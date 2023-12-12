---
displayed_sidebar: "Japanese"
---

# デプロイ後のセットアップ

このトピックでは、StarRocksをデプロイした後に実行する必要があるタスクについて説明します。

新しいStarRocksクラスターを本番環境に導入する前に、初期アカウントをセキュアにし、クラスターが正しく実行されるように必要な変数とプロパティを設定する必要があります。

## 初期アカウントのセキュリティ

StarRocksクラスターを作成すると、クラスターの初期`root`ユーザーが自動的に生成されます。`root`ユーザーには、クラスター内のすべての権限が含まれる`root`特権が付与されます。このユーザーアカウントをセキュアにし、誤用を防ぐために本番環境で使用しないようお勧めします。

StarRocksは、クラスターが作成されると`root`ユーザーに空のパスワードを自動的に割り当てます。次の手順に従って、`root`ユーザーの新しいパスワードを設定してください。

1. MySQLクライアントを使用して`root`ユーザーと空のパスワードでStarRocksに接続します。

   ```Bash
   # <fe_address>を接続するFEノードのIPアドレス（priority_networks）またはFQDNに、<query_port>をfe.confで指定したquery_port（デフォルト：9030）に置き換えます。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 次のSQLを実行して、`root`ユーザーのパスワードをリセットします。

   ```SQL
   -- <password>に`root`ユーザーに割り当てたいパスワードを置き換えてください。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **注意**
>
> - パスワードをリセットした後は、適切に保管してください。パスワードを忘れた場合は、[失われたrootパスワードをリセット](../administration/User_privilege.md#reset-lost-root-password)を参照して、詳細な手順をご覧ください。
> - デプロイ後のセットアップが完了したら、新しいユーザーと役割を作成し、チーム内で権限を管理することができます。詳細な手順については、[ユーザー権限の管理](../administration/User_privilege.md)を参照してください。

## 必要なシステム変数の設定

本番環境でStarRocksクラスターを正しく動作させるために、以下のシステム変数を設定する必要があります。

| **変数名**                           | **StarRocksバージョン** | **推奨値**                                                  | **説明**                                                    |
| ----------------------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| is_report_success                   | v2.4 以前            | false                                                        | クエリのプロファイルを解析するかどうかを制御するブールスイッチ。デフォルト値は`false`で、プロファイルは不要という意味です。この変数を`true`に設定すると、StarRocksの同時実行に影響を与える可能性があります。 |
| enable_profile                      | v2.5 以降            | false                                                        | クエリのプロファイルを解析するかどうかを制御するブールスイッチ。デフォルト値は`false`で、プロファイルは不要という意味です。この変数を`true`に設定すると、StarRocksの同時実行に影響を与える可能性があります。 |
| enable_pipeline_engine              | v2.3 以降            | true                                                         | パイプライン実行エンジンを有効にするかどうかを制御するブールスイッチ。`true`は有効を示し、`false`は逆を示します。デフォルト値：`true`。 |
| parallel_fragment_exec_instance_num | v2.3 以降            | パイプラインエンジンを有効にした場合、この変数を`1`に設定できます。パイプラインエンジンを有効にしていない場合は、CPUコア数の半分に設定する必要があります。| 各BEでノードをスキャンするために使用されるインスタンスの数。デフォルト値は`1`です。 |
| pipeline_dop                        | v2.3、v2.4、v2.5     | 0                                                            | パイプラインインスタンスの並列処理。クエリの同時実行を調整するために使用されます。デフォルト値：`0`は、システムが各パイプラインインスタンスの並列処理を自動的に調整することを示します。<br />v3.0以降、StarRocksはこのパラメータをクエリの並列処理に基づいて適応的に調整します。 |

- `is_report_success`をグローバルで`false`に設定します:

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- `enable_profile`をグローバルで`false`に設定します:

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- `enable_pipeline_engine`をグローバルで`true`に設定します:

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- `parallel_fragment_exec_instance_num`をグローバルで`1`に設定します:

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- `pipeline_dop`をグローバルで`0`に設定します:

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

システム変数に関する詳細な情報については、[システム変数](../reference/System_variable.md)を参照してください。

## ユーザープロパティの設定

クラスターに新しいユーザーを作成した場合、それらの最大接続数を拡大する必要があります（たとえば、`1000`に）：

```SQL
-- <username>を最大接続数を拡大したいユーザー名に置き換えてください。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 次の手順

StarRocksクラスターをデプロイしてセットアップした後は、シナリオに最適なテーブルを設計できます。詳細なテーブル設計手順については、[StarRocksテーブル設計の理解](../table_design/Table_design.md)を参照してください。
