---
displayed_sidebar: "Japanese"
---

# url_extract_host

## 説明

URL文字列からホストを抽出します。

## 構文

```haskell
url_extract_host(str)
```

## パラメータ

- `str`: ホスト文字列を抽出する文字列。`str` が文字列型でない場合、暗黙的なキャストを試みます。

## 戻り値

エンコードされた文字列を返します。

## 例

```plaintext
mysql> select url_extract_host('httpa://starrocks.com/test/api/v1');
+-------------------------------------------------------+
| url_extract_host('httpa://starrocks.com/test/api/v1') |
+-------------------------------------------------------+
| starrocks.com                                         |
+-------------------------------------------------------+
```