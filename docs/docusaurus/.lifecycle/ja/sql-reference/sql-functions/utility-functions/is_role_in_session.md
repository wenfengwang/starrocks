---
displayed_sidebar: "Japanese"
---

# is_role_in_session

## 説明

現在のセッションでロール（またはネストされたロール）がアクティブかどうかを検証します。

この関数はバージョン3.1.4以降でサポートされています。

## 構文

```Haskell
BOOLEAN is_role_in_session(VARCHAR role_name);
```

## パラメータ

`role_name`: 検証したいロール（ネストされたロールでも可能）です。サポートされているデータ型はVARCHARです。

## 戻り値

BOOLEAN値を返します。`1` は現在のセッションでロールがアクティブであることを示します。`0` はその逆を示します。

## 例

1. ロールとユーザーを作成する。

   ```sql
   -- 三つのロールを作成する。
   create role r1;
   create role r2;
   create role r3;

   -- ユーザーu1を作成する。
   create user u1;

   -- ロールr2とr3をr1に渡し、r1をユーザーu1に付与する。これにより、ユーザーu1はr1、r2、r3の三つのロールを持つことになる。
   grant r3 to role r2;
   grant r2 to role r1;
   grant r1 to user u1;

   -- ユーザーu1に切り替えて、u1として操作を行う。
   execute as u1 with no revert;
   ```

2. `r1` がアクティブかどうかを検証する。その結果、このロールはアクティブではないことが表示されます。

   ```plaintext
   select is_role_in_session('r1');
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        0 |
   +--------------------------+
   ```

3. [SET ROLE](../../sql-statements/account-management/SET_ROLE.md) コマンドを実行して `r1` をアクティブにし、`is_role_in_session` を使用してロールがアクティブかどうかを検証します。その結果、 `r1` がアクティブであり、`r1` にネストされた `r2` や `r3` もアクティブであることが示されます。

   ```sql
   set role 'r1';

   select is_role_in_session('r1');
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        1 |
   +--------------------------+

   select is_role_in_session('r2');
   +--------------------------+
   | is_role_in_session('r2') |
   +--------------------------+
   |                        1 |
   +--------------------------+

   select is_role_in_session('r3');
   +--------------------------+
   | is_role_in_session('r3') |
   +--------------------------+
   |                        1 |
   +--------------------------+
   ```