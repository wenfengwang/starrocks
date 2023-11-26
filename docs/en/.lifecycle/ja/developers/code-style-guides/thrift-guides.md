---
displayed_sidebar: "Japanese"
---

# ルール

## 必須は使用しない

プロジェクトが進行するにつれて、任意のフィールドがオプションになる可能性があります。しかし、必須として定義されている場合、削除することはできません。

したがって、`required`は使用しないでください。

## 順序を変更しない

後方互換性を保つために、フィールドの順序は変更しないでください。

# 命名規則

## ファイル名

メッセージの名前はすべて小文字で、単語間にアンダースコアを使用します。
ファイルの拡張子は `.thrift` で終わる必要があります。

```
my_struct.thrift            // 良い例
MyStruct.thrift             // 悪い例
my_struct.proto             // 悪い例
```

## 構造体名

構造体の名前は、大文字の `T` で始まり、新しい単語ごとに大文字を使用し、アンダースコアは使用しません: TMyStruct

```
struct TMyStruct;           // 良い例
struct MyStruct;            // 悪い例
struct TMy_Struct;          // 悪い例
struct TmyStruct;           // 悪い例
```

## フィールド名

構造体のメンバーの名前はすべて小文字で、単語間にアンダースコアを使用します。

```
1: optional i64 my_field;       // 良い例
1: optional i64 myField;        // 悪い例
```
