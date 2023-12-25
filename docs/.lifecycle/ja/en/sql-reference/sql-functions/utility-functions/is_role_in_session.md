---
displayed_sidebar: English
---

# is_role_in_session

## 説明

現在のセッションでロール（またはネストされたロール）がアクティブかどうかを検証します。

この関数はv3.1.4以降でサポートされています。

## 構文

```Haskell
BOOLEAN is_role_in_session(VARCHAR role_name);
```

## パラメーター

`role_name`: 検証したいロール（ネストされたロールも含む）。サポートされるデータ型はVARCHARです。

## 戻り値

BOOLEAN値を返します。`1`はロールが現在のセッションでアクティブであることを示し、`0`はその逆を示します。

## 例

1. ロールとユーザーを作成します。

   ```sql
   -- Create three roles.
   create role r1;
   create role r2;
   create role r3;

   -- Create user u1.
   create user u1;

   -- Pass roles r2 and r3 to r1, and grant r1 to user u1. This way, user u1 has three roles: r1, r2, and r3.
   grant r3 to role r2;
   grant r2 to role r1;
   grant r1 to user u1;

   -- Switch to user u1 and perform operations as u1.
   execute as user u1 with no revert;
   ```

2. `r1`がアクティブかどうかを検証します。結果はこのロールがアクティブでないことを示しています。

   ```plaintext
   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        0 |
   +--------------------------+
   ```

3. [SET ROLE](../../sql-statements/account-management/SET_ROLE.md) コマンドを実行して `r1` をアクティブにし、`is_role_in_session` を使用してロールがアクティブかどうかを検証します。結果は `r1` がアクティブであり、`r1` にネストされている `r2` と `r3` もアクティブであることを示しています。

   ```sql
   set role "r1";

   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        1 |
   +--------------------------+

   select is_role_in_session("r2");
   +--------------------------+
   | is_role_in_session('r2') |
   +--------------------------+
   |                        1 |
   +--------------------------+

   select is_role_in_session("r3");
   +--------------------------+
   | is_role_in_session('r3') |
   +--------------------------+
   |                        1 |
   +--------------------------+
   ```
