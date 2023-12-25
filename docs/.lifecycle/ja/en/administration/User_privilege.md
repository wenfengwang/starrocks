---
displayed_sidebar: English
---

# ユーザー権限の管理

import UserPrivilegeCase from '../assets/commonMarkdown/userPrivilegeCase.md'

このトピックでは、StarRocksでのユーザー、ロール、および権限の管理方法について説明します。

StarRocksは、ロールベースのアクセス制御（RBAC）とアイデンティティベースのアクセス制御（IBAC）を採用しており、クラスタ管理者がStarRocksクラスタ内の権限を異なる粒度レベルで簡単に制限できるようにしています。

StarRocksクラスタ内では、ユーザーやロールに権限を付与することができます。ロールは、必要に応じてクラスタ内のユーザーや他のロールに割り当てられる権限の集合です。ユーザーは一つ以上のロールを付与され、それによって異なるオブジェクトに対する権限が決定されます。

## ユーザーとロール情報の表示

システム定義のロール`user_admin`を持つユーザーは、StarRocksクラスタ内のすべてのユーザーとロール情報を表示できます。

### 権限情報の表示

[SHOW GRANTS](../sql-reference/sql-statements/account-management/SHOW_GRANTS.md)を使用して、ユーザーやロールに付与された権限を表示できます。

- 現在のユーザーの権限を表示します。

  ```SQL
  SHOW GRANTS;
  ```

  > **注記**
  >
  > どのユーザーも、特別な権限を必要とせずに自分自身の権限を表示できます。

- 特定のユーザーの権限を表示します。

  以下は、ユーザー`jack`の権限を表示する例です：

  ```SQL
  SHOW GRANTS FOR jack@'172.10.1.10';
  ```

- 特定のロールの権限を表示します。

  以下は、ロール`example_role`の権限を表示する例です：

  ```SQL
  SHOW GRANTS FOR ROLE example_role;
  ```

### ユーザープロパティの表示

[SHOW PROPERTY](../sql-reference/sql-statements/account-management/SHOW_PROPERTY.md)を使用して、ユーザーのプロパティを表示できます。

以下は、ユーザー`jack`のプロパティを表示する例です：

```SQL
SHOW PROPERTY FOR jack@'172.10.1.10';
```

### ロールの表示

[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)を使用して、StarRocksクラスタ内のすべてのロールを表示できます。

```SQL
SHOW ROLES;
```

### ユーザーの表示

SHOW USERSを使用して、StarRocksクラスタ内のすべてのユーザーを表示できます。

```SQL
SHOW USERS;
```

## ユーザーの管理

システム定義のロール`user_admin`を持つユーザーは、StarRocksでユーザーを作成、変更、削除することができます。

### ユーザーの作成

ユーザーID、認証方法、およびデフォルトロールを指定してユーザーを作成できます。

StarRocksは、ログイン資格情報またはLDAP認証を使用したユーザー認証をサポートしています。StarRocksの認証に関する詳細は、[Authentication](../administration/Authentication.md)を参照してください。ユーザーを作成するための詳細と高度な指示については、[CREATE USER](../sql-reference/sql-statements/account-management/CREATE_USER.md)を参照してください。

以下の例では、ユーザー`jack`を作成し、IPアドレス`172.10.1.10`からのみ接続を許可し、パスワードを`12345`に設定し、デフォルトロールとして`example_role`を割り当てます：

```SQL
CREATE USER jack@'172.10.1.10' IDENTIFIED BY '12345' DEFAULT ROLE 'example_role';
```

> **注記**
>
> - StarRocksは、ユーザーのパスワードを保存する前に暗号化します。password()関数を使用して暗号化されたパスワードを取得できます。
> - システム定義のデフォルトロール`PUBLIC`は、ユーザー作成時にデフォルトロールが指定されていない場合にユーザーに割り当てられます。

### ユーザーの変更

ユーザーのパスワード、デフォルトロール、またはプロパティを変更できます。

ユーザーのデフォルトロールは、StarRocksに接続すると自動的にアクティブになります。接続後にユーザーのすべてのロール（デフォルトおよび付与された）を有効にする方法については、[すべてのロールを有効にする](#enable-all-roles)を参照してください。

#### ユーザーのデフォルトロールを変更する

[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)または[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)を使用して、ユーザーのデフォルトロールを設定できます。

以下の例では、ユーザー`jack`のデフォルトロールを`db1_admin`に設定します。`db1_admin`は`jack`に割り当てられている必要があります。

- SET DEFAULT ROLEを使用してデフォルトロールを設定する：

  ```SQL
  SET DEFAULT ROLE 'db1_admin' TO jack@'172.10.1.10';
  ```

- ALTER USERを使用してデフォルトロールを設定する：

  ```SQL
  ALTER USER jack@'172.10.1.10' DEFAULT ROLE 'db1_admin';
  ```

#### ユーザーのプロパティを変更する

[SET PROPERTY](../sql-reference/sql-statements/account-management/SET_PROPERTY.md)を使用して、ユーザーのプロパティを設定できます。

以下の例では、ユーザー`jack`の最大接続数を`1000`に設定します。同じユーザー名を持つユーザーIDは同じプロパティを共有するため、`jack`のプロパティを設定するだけで、ユーザー名`jack`を持つすべてのユーザーIDに対して設定が有効になります。

```SQL
SET PROPERTY FOR jack 'max_user_connections' = '1000';
```

#### ユーザーのパスワードをリセットする

[SET PASSWORD](../sql-reference/sql-statements/account-management/SET_PASSWORD.md)または[ALTER USER](../sql-reference/sql-statements/account-management/ALTER_USER.md)を使用して、ユーザーのパスワードをリセットできます。

> **注記**
>
> - どのユーザーも、特別な権限を必要とせずに自分自身のパスワードをリセットできます。
> - `root`ユーザーのパスワードは、`root`ユーザー自身しか設定できません。パスワードを紛失してStarRocksに接続できない場合は、[紛失したrootパスワードをリセットする](#reset-lost-root-password)に関する指示を参照してください。

以下の例では、ユーザー`jack`のパスワードを`54321`にリセットします：

- SET PASSWORDを使用してパスワードをリセットする：

  ```SQL
  SET PASSWORD FOR jack@'172.10.1.10' = PASSWORD('54321');
  ```

- ALTER USERを使用してパスワードをリセットする：

  ```SQL
  ALTER USER jack@'172.10.1.10' IDENTIFIED BY '54321';
  ```

#### 紛失したrootパスワードをリセットする

`root`ユーザーのパスワードを紛失してStarRocksに接続できない場合は、以下の手順に従ってリセットできます：

1. すべてのFEノードの設定ファイル**fe/conf/fe.conf**に以下の設定項目を追加して、ユーザー認証を無効にします：

   ```YAML
   enable_auth_check = false
   ```

2. すべてのFEノードを再起動して、設定を有効にします：

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

3. MySQLクライアントを使用して、ユーザー認証が無効の状態で`root`ユーザーとしてStarRocksに接続します。パスワードは必要ありません：

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot
   ```

4. `root`ユーザーのパスワードをリセットします：

   ```SQL
   SET PASSWORD for root = PASSWORD('xxxxxx');
   ```

5. すべてのFEノードの設定ファイル**fe/conf/fe.conf**で`enable_auth_check`を`true`に設定して、ユーザー認証を再度有効にします：

   ```YAML
   enable_auth_check = true
   ```

6. すべてのFEノードを再起動して、設定を有効にします：

   ```Bash
   ./fe/bin/stop_fe.sh
   ./fe/bin/start_fe.sh
   ```

7. 新しいパスワードを使用して`root`ユーザーとしてMySQLクライアントからStarRocksに接続し、パスワードが正常にリセットされたかどうかを確認します：

   ```Bash
   mysql -h <fe_ip_or_fqdn> -P<fe_query_port> -uroot -p<xxxxxx>
   ```

### ユーザーの削除

[DROP USER](../sql-reference/sql-statements/account-management/DROP_USER.md)を使用してユーザーを削除できます。

以下は、ユーザー`jack`を削除する例です：

```SQL
DROP USER jack@'172.10.1.10';
```

## ロールの管理


システム定義のロール`user_admin`を持つユーザーは、StarRocksでロールを作成、付与、取り消し、または削除することができます。

### ロールを作成する

[CREATE ROLE](../sql-reference/sql-statements/account-management/CREATE_ROLE.md)を使用してロールを作成できます。

次の例では、ロール`example_role`を作成します：

```SQL
CREATE ROLE example_role;
```

### ロールを付与する

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を使用して、ユーザーまたは別のロールにロールを付与できます。

- ユーザーにロールを付与する。

  次の例では、ロール`example_role`をユーザー`jack`に付与します：

  ```SQL
  GRANT example_role TO USER jack@'172.10.1.10';
  ```

- 別のロールにロールを付与する。

  次の例では、ロール`example_role`をロール`test_role`に付与します：

  ```SQL
  GRANT example_role TO ROLE test_role;
  ```

### ロールを取り消す

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーまたは別のロールからロールを取り消すことができます。

> **注記**
>
> システム定義のデフォルトロール`PUBLIC`はユーザーから取り消すことはできません。

- ユーザーからロールを取り消す。

  次の例では、ユーザー`jack`からロール`example_role`を取り消します：

  ```SQL
  REVOKE example_role FROM USER jack@'172.10.1.10';
  ```

- 別のロールからロールを取り消す。

  次の例では、ロール`test_role`からロール`example_role`を取り消します：

  ```SQL
  REVOKE example_role FROM ROLE test_role;
  ```

### ロールを削除する

[DROP ROLE](../sql-reference/sql-statements/account-management/DROP_ROLE.md)を使用してロールを削除できます。

次の例では、ロール`example_role`を削除します：

```SQL
DROP ROLE example_role;
```

> **警告**
>
> システム定義のロールは削除できません。

### すべてのロールを有効にする

ユーザーのデフォルトロールは、StarRocksクラスタに接続するたびに自動的にアクティブ化されるロールです。

StarRocksクラスタに接続する際に、すべてのStarRocksユーザーのすべてのロール（デフォルトロールと付与されたロール）を有効にしたい場合は、次の操作を実行します。

この操作にはシステム権限OPERATEが必要です。

```SQL
SET GLOBAL activate_all_roles_on_login = TRUE;
```

また、SET ROLEを使用して自分に割り当てられたロールをアクティブ化することもできます。例えば、ユーザー`jack@'172.10.1.10'`はロール`db_admin`と`user_admin`を持っていますが、これらはユーザーのデフォルトロールではなく、StarRocksに接続したときに自動的にアクティブ化されません。`jack@'172.10.1.10'`が`db_admin`と`user_admin`をアクティブ化する必要がある場合は、`SET ROLE db_admin, user_admin;`を実行できます。SET ROLEは元のロールを上書きするので、すべてのロールを有効にするにはSET ROLE ALLを実行します。

## 権限の管理

システム定義のロール`user_admin`を持つユーザーは、StarRocksで権限を付与または取り消すことができます。

### 権限を付与する

[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を使用して、ユーザーまたはロールに権限を付与できます。

- ユーザーに権限を付与する。

  次の例では、テーブル`sr_member`に対するSELECT権限をユーザー`jack`に付与し、`jack`がこの権限を他のユーザーやロールに付与できるようにします（SQLでWITH GRANT OPTIONを指定することにより）：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO USER jack@'172.10.1.10' WITH GRANT OPTION;
  ```

- ロールに権限を付与する。

  次の例では、テーブル`sr_member`に対するSELECT権限をロール`example_role`に付与します：

  ```SQL
  GRANT SELECT ON TABLE sr_member TO ROLE example_role;
  ```

### 権限を取り消す

[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)を使用して、ユーザーやロールから権限を取り消すことができます。

- ユーザーから権限を取り消す。

  次の例では、テーブル`sr_member`に対するSELECT権限をユーザー`jack`から取り消し、`jack`がこの権限を他のユーザーやロールに付与することを禁止します：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM USER jack@'172.10.1.10';
  ```

- ロールから権限を取り消す。

  次の例では、テーブル`sr_member`に対するSELECT権限をロール`example_role`から取り消します：

  ```SQL
  REVOKE SELECT ON TABLE sr_member FROM ROLE example_role;
  ```

## ベストプラクティス

### マルチサービスアクセスコントロール

通常、企業所有のStarRocksクラスタは単一のサービスプロバイダーによって管理され、複数のビジネスライン（LOB）がそれぞれ1つ以上のデータベースを使用しています。

以下に示すように、StarRocksクラスタのユーザーには、サービスプロバイダーのメンバーと2つのLOB（AおよびB）が含まれます。各LOBは、アナリストとエグゼクティブの2つのロールによって運営されます。アナリストはビジネスステートメントを生成して分析し、エグゼクティブはステートメントを照会します。

![ユーザー権限](../assets/user_privilege_1.png)

LOB Aはデータベース`DB_A`を独立して管理し、LOB Bはデータベース`DB_B`を独立して管理します。LOB AとLOB Bは、データベース`DB_C`で異なるテーブルを使用します。データベース`DB_PUBLIC`は両方のLOBのすべてのメンバーがアクセスできます。

![ユーザー権限](../assets/user_privilege_2.png)

異なるメンバーが異なるデータベースやテーブルで異なる操作を行うため、サービスや役職に応じてロールを作成し、各ロールに必要な権限のみを適用し、これらのロールを対応するメンバーに割り当てることを推奨します。以下に示すように：

![ユーザー権限](../assets/user_privilege_3.png)

1. システム定義のロール`db_admin`、`user_admin`、`cluster_admin`をクラスタメンテナーに割り当て、`db_admin`と`user_admin`を日常メンテナンスのデフォルトロールとして設定し、クラスタのノードを操作する必要があるときに`cluster_admin`を手動でアクティブ化します。

   例：

   ```SQL
   GRANT db_admin, user_admin, cluster_admin TO USER user_platform;
   ALTER USER user_platform DEFAULT ROLE db_admin, user_admin;
   ```

2. LOB内の各メンバーにユーザーを作成し、各ユーザーに複雑なパスワードを設定します。
3. LOB内の各職位にロールを作成し、それぞれのロールに対応する権限を適用します。

   各LOBのディレクターには、そのLOBが必要とする権限の最大セットと、対応するGRANT権限を付与します（SQL文でWITH GRANT OPTIONを指定することにより）。これにより、彼らはこれらの権限をLOBのメンバーに割り当てることができます。日常業務で必要な場合は、そのロールをデフォルトロールとして設定します。

   例：

   ```SQL
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN DATABASE DB_A TO ROLE linea_admin WITH GRANT OPTION;
   GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_admin WITH GRANT OPTION;
   GRANT linea_admin TO USER user_linea_admin;
   ALTER USER user_linea_admin DEFAULT ROLE linea_admin;
   ```

   アナリストとエグゼクティブには、それぞれの権限を持つロールを割り当てます。

   例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_A TO ROLE linea_query;
   GRANT SELECT ON TABLE TABLE_C1, TABLE_C2, TABLE_C3 TO ROLE linea_query;
   GRANT linea_query TO USER user_linea_salesa;
   GRANT linea_query TO USER user_linea_salesb;
   ALTER USER user_linea_salesa DEFAULT ROLE linea_query;
   ALTER USER user_linea_salesb DEFAULT ROLE linea_query;
   ```

4. すべてのクラスタユーザーがアクセスできるデータベース`DB_PUBLIC`については、システム定義のロール`public`にSELECT権限を付与します。

   例：

   ```SQL
   GRANT SELECT ON ALL TABLES IN DATABASE DB_PUBLIC TO ROLE public;
   ```

より複雑なシナリオでロールの継承を実現するために、他のユーザーにロールを割り当てることができます。

たとえば、アナリストが`DB_PUBLIC`内のテーブルに書き込みやクエリを行う権限が必要で、エグゼクティブはこれらのテーブルに対してクエリのみを実行できる場合、ロール`public_analysis`と`public_sales`を作成し、関連する権限をそれぞれのロールに適用して、アナリストとエグゼクティブの元のロールに割り当てることができます。

例：

```SQL
CREATE ROLE public_analysis;
CREATE ROLE public_sales;
GRANT SELECT, ALTER, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA DB_PUBLIC TO ROLE public_analysis;
GRANT SELECT ON ALL TABLES IN SCHEMA DB_PUBLIC TO ROLE public_sales;
GRANT ROLE public_analysis TO linea_analysis;
GRANT ROLE public_analysis TO lineb_analysis;
GRANT ROLE public_sales TO linea_query;
GRANT ROLE public_sales TO lineb_query;
```

### シナリオに基づいてロールをカスタマイズする

<UserPrivilegeCase />
