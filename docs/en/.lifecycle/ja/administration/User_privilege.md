---
displayed_sidebar: "Japanese"
---

# ユーザー権限の管理

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

このトピックでは、StarRocksでユーザー、ロール、および権限を管理する方法について説明します。

StarRocksでは、ロールベースのアクセス制御（RBAC）とアイデンティティベースのアクセス制御（IBAC）の両方を使用して、StarRocksクラスター内の権限を管理し、クラスター管理者がクラスター内の異なる粒度レベルで権限を制限できるようにしています。

StarRocksクラスター内では、ユーザーまたはロールに権限を付与することができます。ロールは、クラスター内のユーザーまたは他のロールに割り当てることができる権限のコレクションです。ユーザーには1つ以上のロールが付与されることがあり、これによって異なるオブジェクトに対する権限が決まります。

## ユーザーおよびロール情報の表示

システム定義のロール `user_admin` を持つユーザーは、StarRocksクラスター内のすべてのユーザーおよびロール情報を表示できます。

### 権限情報の表示

[SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) を使用して、ユーザーまたはロールに付与された権限を表示することができます。

- 現在のユーザーの権限を表示する。

  ```SQL
  SHOW GRANTS;
  ```

  > **注意**
  >
  > ユーザーは、特別な権限を必要とせずに自分自身の権限を表示することができます。

- 特定のユーザーの権限を表示する。

  次の例は、ユーザー `jack` の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 特定のロールの権限を表示する。

  次の例は、ロール `example_role` の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### ユーザープロパティの表示

[SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md) を使用して、ユーザーのプロパティを表示することができます。

次の例は、ユーザー `jack` のプロパティを表示しています。

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### ロールの表示

[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) を使用して、StarRocksクラスター内のすべてのロールを表示することができます。

```SQL
SHOW ROLES;
```

### ユーザーの表示

SHOW USERS を使用して、StarRocksクラスター内のすべてのユーザーを表示することができます。

```SQL
SHOW USERS;
```

## ユーザーの管理

システム定義のロール `user_admin` を持つユーザーは、StarRocksでユーザーの作成、変更、および削除を行うことができます。

### ユーザーの作成

ユーザーの識別子、認証方法、およびデフォルトのロールを指定してユーザーを作成することができます。

StarRocksは、ログイン資格情報またはLDAP認証によるユーザー認証をサポートしています。StarRocksの認証についての詳細については、[認証](../administration/Authentication.md) を参照してください。ユーザーの作成に関する詳細な情報と高度な手順については、[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md) を参照してください。

次の例では、ユーザー `jack` を作成し、IPアドレス `172.10.1.10` からの接続のみを許可し、パスワードを `12345` に設定し、デフォルトのロールとして `example_role` を割り当てます。

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注意**
>
> - StarRocksは、ユーザーのパスワードを保存する前に暗号化します。パスワードを取得するには、password() 関数を使用できます。
> - ユーザー作成時にデフォルトのロールが指定されていない場合、システム定義のデフォルトロール `PUBLIC` がユーザーに割り当てられます。

### ユーザーの変更

ユーザーのパスワード、デフォルトのロール、またはプロパティを変更することができます。

ユーザーのデフォルトロールは、ユーザーがStarRocksに接続すると自動的にアクティブになります。接続後にユーザーのすべての（デフォルトおよび付与された）ロールを有効にする方法については、[すべてのロールを有効にする](#すべてのロールを有効にする) を参照してください。

#### ユーザーのデフォルトロールの変更

[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) または [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) を使用して、ユーザーのデフォルトロールを設定することができます。

次の例は、`jack` のデフォルトロールを `db1_admin` に設定しています。`db1_admin` は `jack` に割り当てられている必要があります。

- SET DEFAULT ROLE を使用してデフォルトロールを設定する場合:

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- ALTER USER を使用してデフォルトロールを設定する場合:

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### ユーザーのプロパティの変更

[SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md) を使用して、ユーザーのプロパティを設定することができます。

次の例では、ユーザー `jack` の最大接続数を `1000` に設定しています。同じユーザー名を持つユーザー識別子は同じプロパティを共有します。

したがって、`jack` のプロパティを設定するだけで、ユーザー名が `jack` のすべてのユーザー識別子にこの設定が適用されます。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### ユーザーのパスワードのリセット

[SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md) または [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) を使用して、ユーザーのパスワードをリセットすることができます。

> **注意**
>
> - ユーザーは、特別な権限を必要とせずに自分自身のパスワードをリセットすることができます。
> - `root` ユーザー自体のパスワードは、`root` ユーザーのみが設定できます。パスワードを失った場合やStarRocksに接続できない場合は、詳細な手順については [失われた root パスワードのリセット](#失われた-root-パスワードのリセット) を参照してください。

次の例では、`jack` のパスワードを `54321` にリセットしています。

- SET PASSWORD を使用してパスワードをリセットする場合:

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- ALTER USER を使用してパスワードをリセットする場合:

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 失われた root パスワードのリセット

`root` ユーザーのパスワードを失った場合やStarRocksに接続できない場合は、次の手順に従ってリセットすることができます。

1. **すべてのFEノード** の **fe/conf/fe.conf** の設定ファイルに、ユーザー認証を無効にするための次の設定項目を追加します。

   ```YAML
   enable_auth_check = false
   ```

2. **すべてのFEノード** を再起動して、設定が有効になるようにします。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. MySQLクライアントから `root` ユーザーを介してStarRocksに接続します。ユーザー認証が無効になっているため、パスワードを指定する必要はありません。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. `root` ユーザーのパスワードをリセットします。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. **すべてのFEノード** の **fe/conf/fe.conf** の設定ファイルで、設定項目 `enable_auth_check` を `true` に設定して、ユーザー認証を再度有効にします。

   ```YAML
   enable_auth_check = true
   ```

6. **すべてのFEノード** を再起動して、設定が有効になるようにします。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. `root` ユーザーと新しいパスワードを使用して、MySQLクライアントからStarRocksに接続して、パスワードが正常にリセットされたかどうかを確認します。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### ユーザーの削除

[DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md) を使用して、ユーザーを削除することができます。

次の例では、ユーザー `jack` を削除しています。

```SQL
DROP USER jack@'172.10.1.10';
```

## ロールの管理

システム定義のロール `user_admin` を持つユーザーは、StarRocksでロールの作成、付与、取り消し、または削除を行うことができます。

### ロールの作成

[CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md) を使用して、ロールを作成することができます。

次の例では、ロール `example_role` を作成しています。

```SQL
CREATE ROLE example_role;
```

### ロールの付与

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を使用して、ユーザーまたは他のロールにロールを付与することができます。

- ユーザーにロールを付与する。

  次の例では、ロール `example_role` をユーザー `jack` に付与しています。

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- ロールにロールを付与する。

  次の例では、ロール `example_role` をロール `test_role` に付与しています。

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### ロールの取り消し

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーまたは他のロールからロールを取り消すことができます。

> **注意**
>
> システム定義のデフォルトロール `PUBLIC` をユーザーから取り消すことはできません。

- ユーザーからロールを取り消す。

  次の例では、ユーザー `jack` からロール `example_role` を取り消しています。

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- ロールからロールを取り消す。

  次の例では、ロール `test_role` からロール `example_role` を取り消しています。

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### ロールの削除

[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md) を使用して、ロールを削除することができます。

次の例では、ロール `example_role` を削除しています。

```SQL
DROP ROLE example_role;
```

> **注意**
>
> システム定義のロールは削除することはできません。

### すべてのロールを有効にする

ユーザーのデフォルトロールは、ユーザーがStarRocksクラスターに接続するたびに自動的にアクティブになるロールです。

StarRocksユーザーがStarRocksクラスターに接続するときに、すべてのロール（デフォルトおよび付与されたロール）を有効にする場合は、次の操作を実行できます。

この操作には、システム権限 OPERATE が必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

また、SET ROLE を使用して、自分に割り当てられたロールをアクティブにすることもできます。たとえば、ユーザー `jack@'172.10.1.10'` には `db_admin` と `user_admin` のロールがありますが、これらはユーザーのデフォルトロールではなく、ユーザーがStarRocksに接続したときに自動的にアクティブになりません。jack@'172.10.1.10' が `db_admin` と `user_admin` をアクティブにする必要がある場合は、`SET ROLE db_admin, user_admin;` を実行します。SET ROLE は元のロールを上書きします。すべてのロールを有効にする場合は、SET ROLE ALL を実行します。

## 権限の管理

システム定義のロール `user_admin` を持つユーザーは、StarRocksで権限を付与または取り消すことができます。

### 権限の付与

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を使用して、ユーザーまたはロールに権限を付与することができます。

- ユーザーに権限を付与する。

  次の例では、ユーザー `jack` にテーブル `sr_member` の SELECT 権限を付与し、`jack` に他のユーザーやロールにこの権限を付与することを許可しています（SQLで WITH GRANT OPTION を指定）。

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- ロールに権限を付与する。

  次の例では、ロール `example_role` にテーブル `sr_member` の SELECT 権限を付与しています。

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 権限の取り消し

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザーまたはロールから権限を取り消すことができます。

- ユーザーから権限を取り消す。

  次の例では、ユーザー `jack` からテーブル `sr_member` の SELECT 権限を取り消し、`jack` に他のユーザーやロールにこの権限を付与することを禁止しています）。

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- ロールから権限を取り消す。

  次の例では、ロール `example_role` からテーブル `sr_member` の SELECT 権限を取り消しています。

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## ベストプラクティス

### マルチサービスアクセス制御

通常、企業が所有するStarRocksクラスターは、単一のサービスプロバイダーによって管理され、複数の事業部門（LOB）があります。各LOBは1つ以上のデータベースを使用します。

以下のように、StarRocksクラスターのユーザーには、サービスプロバイダーと2つのLOB（AとB）のメンバーが含まれます。各LOBは、アナリストと幹部の2つのロールで運営されます。アナリストはビジネスのステートメントを生成および分析し、幹部はステートメントをクエリします。

![ユーザー権限](../assets/user_privilege_1.png)

LOB Aはデータベース `DB_A` を独自に管理し、LOB Bはデータベース `DB_B` を管理します。LOB AとLOB Bは `DB_C` の異なるテーブルを使用します。`DB_PUBLIC` は両LOBのすべてのメンバーからアクセスできます。

![ユーザー権限](../assets/user_privilege_2.png)

異なるメンバーが異なるデータベースとテーブルで異なる操作を実行するため、各サービスとポジションに基づいてロールを作成し、各ロールに必要な権限のみを適用し、これらのロールを対応するメンバーに割り当てることをお勧めします。以下のように：

![ユーザー権限](../assets/user_privilege_3.png)

1. クラスターメンテナンス担当者には、システム定義のロール `db_admin`、`user_admin`、および `cluster_admin` を割り当て、デイリーメンテナンスには `db_admin` と `user_admin` をデフォルトのロールとして設定し、クラスターノードの操作が必要な場合にのみロール `cluster_admin` を手動でアクティブにします。

   例：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. 各LOB内の各メンバーに対してユーザーを作成し、各ユーザーに複雑なパスワードを設定します。
3. 各LOB内の各ポジションに対してロールを作成し、各ロールに対応する権限を適用します。

   各LOBのディレクターに対しては、LOBが必要とする最大の権限を持つロールを付与し、対応するGRANT権限（SQLで WITH GRANT OPTION を指定）を付与します。したがって、彼らはこれらの権限をLOBのメンバーに割り当てることができます。デイリーワークで必要な場合は、ロールをデフォルトのロールに設定します。

   例：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

   アナリストと幹部の場合、対応する権限を持つロールを割り当てます。

   例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. すべてのクラスターユーザーからアクセスできるデータベース `DB_PUBLIC` に対して、システム定義のロール `public` に SELECT 権限を付与します。

   例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

複雑なシナリオでは、他のユーザーにロールを割り当てて、ロールの継承を実現することができます。

たとえば、アナリストが `DB_PUBLIC` のテーブルに書き込みとクエリの権限を必要とし、幹部はこれらのテーブルをクエリするだけでよい場合、ロール `public_analysis` と `public_sales` を作成し、関連する権限をロールに適用し、それぞれアナリストと幹部の元のロールに割り当てます。

例：

```SQL
CREATE ROLE public_analysis;
CREATE ROLE public_sales;
GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public_analysis;
GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public_sales;
GRANT public_analysis TO ROLE linea_analysis;
GRANT public_analysis TO ROLE lineb_analysis;
GRANT public_sales TO ROLE linea_query;
GRANT public_sales TO ROLE lineb_query;
```

### シナリオに基づいたロールのカスタマイズ

<UserPrivilegeCase />
