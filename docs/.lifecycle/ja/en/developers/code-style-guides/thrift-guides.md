---
displayed_sidebar: English
---

# ルール

## `required`は使用しない

プロジェクトの進行に伴い、どのフィールドもオプションになる可能性があります。しかし、一度`required`として定義された場合、それを取り除くことはできません。

そのため、`required`は使用しないでください。

## 順序を変更しない

後方互換性を保つために、フィールドの順序は変更してはいけません(SHOULD NOT)。

# 命名規則

## ファイル名

メッセージの名前は全て小文字で、単語間にはアンダースコアを使用します。
ファイル名は`.thrift`で終わるべきです。

```
my_struct.thrift            // Good
MyStruct.thrift             // Bad
my_struct.proto             // Bad
```

## 構造体の名前

構造体の名前は大文字の`T`で始まり、新しい単語ごとに大文字を使用し、アンダースコアは使用しません: TMyStruct

```
struct TMyStruct;           // Good
struct MyStruct;            // Bad
struct TMy_Struct;          // Bad
struct TmyStruct;           // Bad
```

## フィールド名

構造体メンバーの名前は全て小文字で、単語間にアンダースコアを使用します。

```
1: optional i64 my_field;       // Good
1: optional i64 myField;        // Bad
```