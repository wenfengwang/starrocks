```yaml
---
displayed_sidebar: "Japanese"
---

# url_extract_host

## Description

URL文字列からホストを抽出します。

## 構文

```haskell
url_extract_host(str)
```

## パラメータ

- `str`: ホスト文字列を抽出するための文字列です。`str`が文字列型でない場合は、暗黙の型変換を試みます。

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