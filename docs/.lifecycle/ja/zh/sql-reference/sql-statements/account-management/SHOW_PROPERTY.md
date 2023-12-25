---
displayed_sidebar: Chinese
---


# SHOW PROPERTY

## 機能

単一ユーザーの最大接続数を表示します。

> **注意**
>
> 現在のユーザーは自分の property を照会できます。しかし、`user_admin` ロールを持つユーザーのみが他のユーザーの property を表示できます。

## 構文

```SQL
SHOW PROPERTY [FOR 'user_name'] [LIKE 'max_user_connections']
```

## パラメータ説明

| **パラメータ**        | **必須** | **説明**                                         |
| -------------------- | -------- | ------------------------------------------------ |
| user_name            | いいえ   | ユーザー名。指定しない場合は、現在のユーザーの最大接続数が表示されます。 |
| max_user_connections | いいえ   | ユーザーの最大接続数。                            |

## 例

例1：現在のユーザーの最大接続数を表示します。

```Plain
SHOW PROPERTY;

+----------------------+-------+
| Key                  | Value |
+----------------------+-------+
| max_user_connections | 10000 |
+----------------------+-------+
```

例2：ユーザー `jack` の最大接続数を表示します。

```SQL
SHOW PROPERTY FOR 'jack';
```

または

```SQL
SHOW PROPERTY FOR 'jack' LIKE 'max_user_connections';
```

返される情報は以下の通りです：

```Plain
+----------------------+-------+
| Key                  | Value |
+----------------------+-------+
| max_user_connections | 100   |
+----------------------+-------+
```

## 関連操作

ユーザーの最大接続数を設定するには、[SET PROPERTY](./SET_PROPERTY.md) を参照してください。
