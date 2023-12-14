---
displayed_sidebar: "Chinese"
---

# url_encode

## 描述

将编码URL字符串。

## 语法

```haskell
url_encode(str)
```

## 参数

- `str`：要编码的字符串。如果`str`不是字符串类型，将尝试进行隐式转换。

## 返回值

返回一个编码字符串。

## 示例

```plaintext
mysql> select url_encode('https://docs.starrocks.io/en-us/latest/quick_start/Deploy');
+-------------------------------------------------------------------------+
| url_encode('https://docs.starrocks.io/en-us/latest/quick_start/Deploy') |
+-------------------------------------------------------------------------+
| https%3A%2F%2Fdocs.starrocks.io%2Fen-us%2Flatest%2Fquick_start%2FDeploy |
+-------------------------------------------------------------------------+
```