---
displayed_sidebar: Chinese
---

# DROP ROLE

## 機能

ロールを削除します。ロールがユーザーに付与されていた場合でも、そのロールを削除しても、ユーザーは引き続きそのロールが持っていた関連権限を保持します。

> **注意**
>
> - `user_admin` のみがロールを削除できます。
> - StarRocks システムにプリセットされたロールは削除できません。

## 文法

```SQL
DROP ROLE <role_name>
```

## パラメータ説明

`role_name`：削除するロールの名前。

## 例

ロール `analyst` を削除します。

```SQL
  DROP ROLE analyst;
```
