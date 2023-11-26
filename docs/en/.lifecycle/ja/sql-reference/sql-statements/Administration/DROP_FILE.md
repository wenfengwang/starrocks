---
displayed_sidebar: "Japanese"
---

# ファイルの削除

ファイルを削除するには、DROP FILE ステートメントを実行します。このステートメントを使用してファイルを削除すると、ファイルはフロントエンド（FE）メモリと Berkeley DB Java Edition（BDBJE）の両方で削除されます。

## 構文

```SQL
DROP FILE "file_name" [FROM database]
[properties]
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| file_name     | Yes      | ファイルの名前。                                              |
| database      | No       | ファイルが所属するデータベース。                              |
| properties    | Yes      | ファイルのプロパティ。以下の表には、プロパティの構成項目が記載されています。 |

**`properties` の構成項目**

| **構成項目** | **必須** | **説明**                       |
| ------------ | -------- | ----------------------------- |
| catalog      | Yes      | ファイルが所属するカテゴリ。 |

## 例

名前が **ca.pem** のファイルを削除します。

```SQL
DROP FILE "ca.pem" properties("catalog" = "kafka");
```
