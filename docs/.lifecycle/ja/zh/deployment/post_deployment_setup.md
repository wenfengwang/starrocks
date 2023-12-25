---
displayed_sidebar: Chinese
---

# 部署後の設定

この文書では、StarRocks をデプロイした後に実行する必要があるタスクについて説明します。

新しい StarRocks クラスターを本番環境に投入する前に、初期アカウントを管理し、クラスターが正常に動作するために必要な変数と属性を設定する必要があります。

## 初期アカウントの管理

StarRocks クラスターを作成すると、システムは自動的にクラスターの初期 `root` ユーザーを生成します。`root` ユーザーは `root` 権限を持っており、これはクラスター内のすべての権限の集合です。`root` ユーザーのパスワードを変更し、本番環境での使用を避けることをお勧めします。

1. ユーザー名 `root` と空のパスワードを使用して、MySQL クライアントを介して StarRocks に接続します。

   ```Bash
   # <fe_address> を接続する FE ノードの IP アドレス（priority_networks）
   # または FQDN に置き換え、<query_port> を fe.conf で指定した query_port に置き換えます（デフォルト：9030）。
   mysql -h <fe_address> -P<query_port> -uroot
   ```

2. 以下の SQL を実行して `root` ユーザーのパスワードをリセットします：

   ```SQL
   -- <password> を root ユーザーに設定したいパスワードに置き換えます。
   SET PASSWORD = PASSWORD('<password>')
   ```

> **説明**
>
> - パスワードをリセットした後は、必ず安全に保管してください。パスワードを忘れた場合は、[失われた root パスワードのリセット](../administration/User_privilege.md#失われた-root-パスワードのリセット) を参照して詳細な説明をご覧ください。
> - デプロイ後の設定を完了した後、新しいユーザーやロールを作成してチーム内の権限を管理できます。詳細は [ユーザー権限の管理](../administration/User_privilege.md) を参照してください。

## 必要なシステム変数の設定

StarRocks クラスターが本番環境で正常に動作するためには、以下のシステム変数を設定する必要があります：

| **変数名**                          | **StarRocks バージョン** | **推奨値**                                                   | **説明**                                                     |
| ----------------------------------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| is_report_success                   | v2.4 以前           | false                                                        | クエリの Profile を分析用に送信するかどうか。デフォルトは `false` で、送信しない。この変数を `true` に設定すると StarRocks の並行性能に影響します。 |
| enable_profile                      | v2.5 以降           | false                                                        | クエリの Profile を分析用に送信するかどうか。デフォルトは `false` で、送信しない。この変数を `true` に設定すると StarRocks の並行性能に影響します。 |
| enable_pipeline_engine              | v2.3 以降           | true                                                         | Pipeline Engine を有効にするかどうか。`true` は有効、`false` は無効。デフォルトは `true` です。 |
| parallel_fragment_exec_instance_num | v2.3 以降           | Pipeline Engine を有効にした場合は `1` に設定できます。無効にした場合は CPU コア数の半分に設定できます。 | 各 BE でスキャンノードのインスタンス数。デフォルトは `1` です。               |
| pipeline_dop                        | v2.3、v2.4、v2.5    | 0                                                            | Pipeline インスタンスの並行度で、クエリの並行度を調整します。デフォルトは 0 で、システムが各 Pipeline インスタンスの並行度を自動調整します。<br />v3.0 からは、StarRocks がクエリの並行度に基づいてこのパラメータを自動調整します。 |

- `is_report_success` をグローバルに `false` に設定します：

  ```SQL
  SET GLOBAL is_report_success = false;
  ```

- `enable_profile` をグローバルに `false` に設定します：

  ```SQL
  SET GLOBAL enable_profile = false;
  ```

- `enable_pipeline_engine` をグローバルに `true` に設定します：

  ```SQL
  SET GLOBAL enable_pipeline_engine = true;
  ```

- `parallel_fragment_exec_instance_num` をグローバルに `1` に設定します：

  ```SQL
  SET GLOBAL parallel_fragment_exec_instance_num = 1;
  ```

- `pipeline_dop` をグローバルに `0` に設定します：

  ```SQL
  SET GLOBAL pipeline_dop = 0;
  ```

システム変数の詳細については、[システム変数](../reference/System_variable.md) を参照してください。

## ユーザー属性の設定

クラスターに新しいユーザーを作成した場合は、そのユーザーの最大接続数を増やす必要があります（例：`1000` まで）：

```SQL
-- <username> を最大接続数を増やしたいユーザー名に置き換えます。
SET PROPERTY FOR '<username>' 'max_user_connections' = '1000';
```

## 次のステップ

StarRocks クラスターのデプロイと設定が成功した後、ビジネスシナリオに最適なテーブル設計を開始できます。テーブル設計の詳細については、[テーブル設計の理解](../table_design/StarRocks_table_design.md) を参照してください。
