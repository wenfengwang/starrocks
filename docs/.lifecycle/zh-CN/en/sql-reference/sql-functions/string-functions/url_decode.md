---
displayed_sidebar: "Chinese"
---

# url_decode

## 描述

将网址转换为解码字符串。

## 语法

```haskell
url_decode(str)
```

## 参数

- `str`：要解码的字符串。如果`str`不是字符串类型，则会尝试隐式转换。

## 返回值

返回一个编码字符串。

## 示例

```plaintext
mysql> select url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fen-us%2Flatest%2Fquick_start%2FDeploy');
+---------------------------------------------------------------------------------------+
| url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fen-us%2Flatest%2Fquick_start%2FDeploy') |
+---------------------------------------------------------------------------------------+
| https://docs.starrocks.io/en-us/latest/quick_start/Deploy                             |
+---------------------------------------------------------------------------------------+
```