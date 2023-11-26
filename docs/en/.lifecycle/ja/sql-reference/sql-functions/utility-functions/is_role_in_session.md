---
displayed_sidebar: "Japanese"
---

# is_role_in_session

## 説明

現在のセッションで、ロール（またはネストされたロール）がアクティブかどうかを確認します。

この関数は、v3.1.4以降でサポートされています。

## 構文

```Haskell
BOOLEAN is_role_in_session(VARCHAR role_name);
```

## パラメーター

`role_name`: 確認したいロール（ネストされたロールも可能）です。サポートされるデータ型はVARCHARです。

## 戻り値

BOOLEAN値を返します。 `1`は、ロールが現在のセッションでアクティブであることを示します。 `0`はその逆を示します。

## 例

1. ロールとユーザーを作成します。

   ```sql
   -- 3つのロールを作成します。
   create role r1;
   create role r2;
   create role r3;

   -- ユーザーu1を作成します。
   create user u1;

   -- ロールr2とr3をr1に渡し、r1をユーザーu1に付与します。これにより、ユーザーu1はr1、r2、r3の3つのロールを持ちます。
   grant r3 to role r2;
   grant r2 to role r1;
   grant r1 to user u1;

   -- ユーザーu1に切り替えて、u1として操作を実行します。
   execute as u1 with no revert;
   ```

2. `r1`がアクティブかどうかを確認します。結果は、このロールがアクティブでないことを示しています。

   ```plaintext
   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        0 |
   +--------------------------+
   ```

3. [SET ROLE](../../sql-statements/account-management/SET_ROLE.md)コマンドを実行して`r1`をアクティブにし、`is_role_in_session`を使用してロールがアクティブかどうかを確認します。結果は、`r1`がアクティブであり、`r1`にネストされた`r2`と`r3`もアクティブであることを示しています。

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
