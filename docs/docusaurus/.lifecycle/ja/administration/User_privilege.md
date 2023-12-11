---
displayed_sidebar: "Japanese"
---

# ユーザ権限の管理

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

このトピックでは、StarRocksでのユーザ、ロール、権限の管理方法について説明します。

StarRocksは、StarRocksクラスタ内での権限を簡単に制限できるようにするために、ロールベースのアクセス制御（RBAC）とアイデンティティベースのアクセス制御（IBAC）の両方を利用しています。

StarRocksクラスタ内では、特権はユーザまたはロールに付与することができます。ロールは、必要に応じてクラスタ内の他のロールやユーザに割り当てることができる特権のコレクションです。ユーザには1つ以上のロールが付与されることがあり、これによって異なるオブジェクトに対するアクセス権限が決まります。

## ユーザとロール情報の表示

システム定義ロール `user_admin` を持つユーザは、StarRocksクラスタ内のすべてのユーザとロール情報を表示できます。

### 権限情報の表示

[SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md) を使用して、ユーザまたはロールに付与された権限を表示できます。

- 現在のユーザの権限を表示します。

  ```SQL
  SHOW GRANTS;
  ```

  > **注記**
  >
  > 任意のユーザは、特権を必要とせずに自分自身の権限を表示できます。

- 特定のユーザの権限を表示します。

  次の例では、ユーザ `jack` の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 特定のロールの権限を表示します。

  次の例では、ロール `example_role` の権限を表示しています。

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### ユーザのプロパティの表示

[SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md) を使用して、ユーザのプロパティを表示できます。

次の例では、ユーザ `jack` のプロパティを表示しています。

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### ロールの表示

[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md) を使用して、StarRocksクラスタ内のすべてのロールを表示できます。

```SQL
SHOW ROLES;
```

### ユーザの表示

SHOW USERS を使用して、StarRocksクラスタ内のすべてのユーザを表示できます。

```SQL
SHOW USERS;
```

## ユーザの管理

システム定義ロール `user_admin` を持つユーザは、StarRocksでユーザの作成、変更、削除を行うことができます。

### ユーザの作成

ユーザのアイデンティティ、認証方法、デフォルトロールを指定してユーザを作成できます。

StarRocksは、ログイン資格情報またはLDAP認証を使用したユーザ認証をサポートしています。StarRocksの認証に関する詳細情報については、[認証](../administration/Authentication.md) を参照してください。ユーザの作成に関する詳細情報や高度な手順については、[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md) を参照してください。

次の例では、ユーザ `jack` を作成し、IPアドレス `172.10.1.10` からのみの接続を許可し、パスワードを `12345` に設定し、デフォルトロールとして `example_role` を割り当てています:

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注記**
>
> - StarRocksは、ユーザのパスワードを保存する前に暗号化します。暗号化されたパスワードは、password() 関数を使用して取得できます。
> - ユーザ作成時にデフォルトロールが指定されていない場合、システム定義のデフォルトロール `PUBLIC` がユーザに割り当てられます。

### ユーザの変更

ユーザのパスワード、デフォルトロール、またはプロパティを変更できます。

ユーザがStarRocksに接続するとき、デフォルトロールが自動的にアクティブ化されます。接続後にユーザにすべての（デフォルトと付与された）ロールを有効にする方法については、[すべてのロールを有効にする](#enable-all-roles) を参照してください。

#### ユーザのデフォルトロールの変更

[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) または [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) を使用して、ユーザのデフォルトロールを設定できます。

次の例は、いずれも `jack` のデフォルトロールを `db1_admin` に設定しています。なお、`db1_admin` は `jack` に割り当てられている必要があります。

- SET DEFAULT ROLE を使用してデフォルトロールを設定:

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- ALTER USER を使用してデフォルトロールを設定:

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### ユーザのプロパティの変更

[SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md) を使用して、ユーザのプロパティを設定できます。

次の例は、ユーザ `jack` の最大接続数を `1000` に設定しています。同じユーザ名を持つユーザアイデンティティは、同じプロパティを共有します。

したがって、`jack` のプロパティを設定するだけで、ユーザ名が `jack` のすべてのユーザアイデンティティに設定が適用されます。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### ユーザのパスワードのリセット

[SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md) または [ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md) を使用して、ユーザのパスワードをリセットできます。

> **注記**
>
> - 任意のユーザは、特権を必要とせずに自分自身のパスワードをリセットできます。
> - パスワードをリセットするには `root` ユーザ自身のみができます。パスワードを紛失し、StarRocksに接続できない場合は、[リセットされた root パスワードをリセットする](#reset-lost-root-password) を参照してください。

次の例は、いずれも `jack` のパスワードを `54321` にリセットしています。

- SET PASSWORD を使用してパスワードをリセット:

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- ALTER USER を使用してパスワードをリセット:

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### リセットされた root パスワードをリセットする

`root` ユーザのパスワードを紛失し、StarRocksに接続できない場合は、以下の手順に従ってパスワードをリセットすることができます:

1. すべてのFEノードの**fe/conf/fe.conf**の設定ファイルに、ユーザ認証を無効にするための以下の構成項目を追加します:

   ```YAML
   enable_auth_check = false
   ```

2. 設定の有効化にするために、**すべてのFEノード**を再起動します。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. MySQLクライアントを使用して、ユーザ認証が無効の状態でStarRocksに`root`ユーザとして接続します。ユーザ認証が無効の場合、パスワードを指定する必要はありません。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. `root`ユーザのパスワードをリセットします。

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. **すべてのFEノード**の設定ファイル**fe/conf/fe.conf**に、構成項目 `enable_auth_check` を `true` に設定して、ユーザ認証を再び有効にします。

   ```YAML
   enable_auth_check = true
   ```

6. 設定の有効化にするために、**すべてのFEノード**を再起動します。

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. 新しいパスワードを使用して`root`ユーザとしてStarRocksに正常に接続するかどうかを検証するために、MySQLクライアントを使用して、**すべてのFEノード**に再接続します。

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### ユーザの削除

[DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md) を使用して、ユーザを削除できます。

次の例では、ユーザ `jack` を削除しています。

```SQL
DROP USER jack@'172.10.1.10';
```

## ロールの管理

システム定義ロール `user_admin` を持つユーザは、StarRocksでロールの作成、付与、取り消し、削除を行うことができます。

### ロールの作成

[CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md) を使用して、ロールを作成できます。

次の例では、ロール `example_role` を作成しています。

```SQL
CREATE ROLE example_role;
```

### ロールの付与

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を使用して、ユーザまたは他のロールにロールを付与することができます。

- ユーザにロールを付与します。

  次の例では、ユーザ `jack` にロール `example_role` を付与しています。

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 他のロールにロールを付与します。

  次の例では、ロール `example_role` をロール `test_role` に付与しています。

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### ロールの取り消し

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) を使用して、ユーザまたは他のロールからロールを取り消すことができます。

> **注記**
>
> システムで定義されたデフォルトの役割`PUBLIC`は、ユーザーから取り消すことはできません。

- ユーザーから役割を取り消す。

  次の例は、ユーザー`jack`から役割`example_role`を取り消すものです：

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 別の役割から役割を取り消す。

  次の例は、役割`test_role`から役割`example_role`を取り消すものです：

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### 役割を削除する

[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md)を使用して、役割を削除できます。

次の例は、役割`example_role`を削除するものです：

```SQL
DROP ROLE example_role;
```

> **注意**
>
> システムで定義された役割は削除できません。

### すべての役割を有効にする

ユーザーのデフォルト役割は、ユーザーがStarRocksクラスタに接続するたびに自動的に有効になる役割です。

StarRocksユーザーがStarRocksクラスタに接続する際に、すべての役割（デフォルトの役割および付与された役割）を有効にしたい場合は、次の操作を実行できます。

この操作には、システム権限OPERATEが必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

また、SET ROLEを使用して、自分に割り当てられた役割を有効にすることもできます。たとえば、ユーザー`jack@'172.10.1.10'`は`db_admin`および`user_admin`の役割を持っていますが、これらはユーザーのデフォルトの役割ではなく、StarRocksに接続する際に自動的に有効になるものではありません。jack@'172.10.1.10'が`db_admin`および`user_admin`を有効にする必要がある場合は、`SET ROLE db_admin, user_admin;`を実行できます。SET ROLEは元の役割を上書きします。すべての役割を有効にしたい場合は、`SET ROLE ALL`を実行してください。

## 権限の管理

システムで定義された役割`user_admin`を持つユーザーは、StarRocksで権限を付与または取り消すことができます。

### 権限を付与する

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を使用して、ユーザーまたは役割に権限を付与できます。

- ユーザーに権限を付与する。

  次の例は、テーブル`sr_member`に対するSELECT権限をユーザー`jack`に付与し、`jack`に他のユーザーや役割にこの権限を付与することを許可するものです（SQLでWITH GRANT OPTIONを指定）：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- 役割に権限を付与する。

  次の例は、テーブル`sr_member`に対するSELECT権限を役割`example_role`に付与するものです：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 権限を取り消す

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーまたは役割から権限を取り消すことができます。

- ユーザーから権限を取り消す。

  次の例は、テーブル`sr_member`に対するSELECT権限をユーザー`jack`から取り消し、`jack`に他のユーザーや役割にこの権限を付与することを許可しません：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- 役割から権限を取り消す。

  次の例は、テーブル`sr_member`に対するSELECT権限を役割`example_role`から取り消すものです：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## ベストプラクティス

### マルチサービスアクセス制御

通常、企業所有のStarRocksクラスタは単一のサービスプロバイダによって管理され、複数の事業部門（LOB）があり、それぞれが1つ以上のデータベースを使用しています。

以下に示すように、StarRocksクラスタのユーザーには、サービスプロバイダとLOB（AおよびB）のメンバーが含まれます。各LOBはアナリストとエグゼクティブという2つの役割で運営されます。アナリストはビジネスの記録を作成し分析し、エグゼクティブはこれらの記録をクエリします。

![User Privileges](../assets/user_privilege_1.png)

LOB Aはデータベース`DB_A`を独自に管理し、LOB Bはデータベース`DB_B`を管理しています。LOB AおよびLOB Bは`DB_C`の異なるテーブルを使用しています。`DB_PUBLIC`は、両方のLOBのすべてのメンバーがアクセスできるようになっています。

![User Privileges](../assets/user_privilege_2.png)

異なるメンバーが異なるデータベースやテーブルで異なる操作を行うため、それぞれのサービスやポジションに応じた役割を作成し、各役割に必要な権限のみを適用し、これらの役割を対応するメンバーに割り当てることをお勧めします。以下に示すように：

1. クラスタメンテナンス担当者には、システムで定義された役割`db_admin`、`user_admin`、`cluster_admin`を付与し、デイリーメンテナンスにはデフォルトの役割として`db_admin`と`user_admin`を設定し、クラスタのノードを操作する必要があるときにのみ役割`cluster_admin`を手動でアクティブ化します。

   例：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. 各LOB内の各メンバー用のユーザーを作成し、それぞれのユーザーに複雑なパスワードを設定します。
3. 各LOB内の各ポジション用の役割を作成し、それぞれの役割に対応する権限を適用します。

   各LOBのディレクターには、LOBが必要とする権限の最大コレクションを付与し、ステートメントでWITH GRANT OPTIONを指定することで対応するGRANT権限を付与します。したがって、LOBのメンバーにこれらの権限を割り当てることができます。デイリーの作業に必要な場合は、その役割をデフォルトの役割に設定します。

   例：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

   アナリストとエグゼクティブには、それぞれの権限を持つ役割を割り当てます。

   例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. すべてのクラスタユーザーがアクセスできる`DB_PUBLIC`に対し、システムで定義された役割`public`にSELECT権限を付与します。

   例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

複雑なシナリオでの役割の継承を実現するために、他の人に役割を割り当てることができます。

例えば、アナリストが`DB_PUBLIC`のテーブルに書き込みおよびクエリする権限を必要とし、エグゼクティブはこれらのテーブルをクエリするだけである場合、`public_analysis`と`public_sales`の役割を作成し、それぞれの役割に関連する権限を適用し、これらを元のアナリストおよびエグゼクティブの役割に割り当てます。

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

### シナリオに基づいたカスタマイズされた役割

<UserPrivilegeCase />