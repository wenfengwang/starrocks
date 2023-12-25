---
displayed_sidebar: Chinese
---

# curtime、current_time

## 機能

現在の時間を取得し、TIME 型で返します。

この関数はタイムゾーンの影響を受けます。詳細は [タイムゾーンの設定](../../../administration/timezone.md) を参照してください。

## 文法

```Haskell
TIME CURTIME()
```

## 例

```Plain Text
select current_time();
+----------------+
| current_time() |
+----------------+
| 15:25:47       |
+----------------+
```
