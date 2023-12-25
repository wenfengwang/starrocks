---
displayed_sidebar: Chinese
---

# プロパティの設定

## 機能

ユーザー属性を設定します。現在、単一ユーザーの最大接続数のみを設定することができ、`user_admin` ロールを持つユーザーのみがこの属性を設定する権限を持っています。

> **説明**
>
> ここでの属性はユーザー (user) の属性を指し、ユーザー識別子 (user_identity) の属性ではありません。例えば、CREATE USER ステートメントで `'jack'@'%'` と `'jack'@'192.%'` を作成した場合、SET PROPERTY ステートメントで設定されるのは `jack` というユーザーの属性であり、`'jack'@'%'` や `'jack'@'192.%'` の属性ではありません。

## 文法

```SQL
SET PROPERTY [FOR 'user'] 'max_user_connections' = 'value'
```

## パラメータ説明

- `FOR 'user'`：指定ユーザー、オプションパラメータ。このパラメータを設定しない場合、デフォルトで現在のユーザーの属性が設定されます。

- `'max_user_connections' = 'value'`：単一ユーザーの最大接続数、必須パラメータ。有効な数値範囲は 1 から 10000 です。

## 例

例1：現在のユーザーの最大接続数を 1000 に設定します。

```SQL
SET PROPERTY 'max_user_connections' = '1000';
```

例2：ユーザー `jack` の最大接続数を 1000 に設定します。

```SQL
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
```

## 関連操作

- ユーザーを作成するには、[CREATE USER](./CREATE_USER.md) を参照してください。

- ユーザー属性を表示するには、[SHOW PROPERTY](./SHOW_PROPERTY.md) を参照してください。
