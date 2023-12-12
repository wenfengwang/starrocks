---
displayed_sidebar: "Japanese"
---

# ルール

## 必須の使用禁止

プロジェクトが進行するにつれて、任意のフィールドが必須ではなくなることがあります。しかし、必須として定義されている場合は削除できません。

したがって、「required」は使用しないでください。

## 順序の変更禁止

後方互換性を保つため、フィールドの順序を変更してはいけません。

# 名前付け

## ファイル名

メッセージの名前はすべて小文字で、単語の間にアンダースコアを入れます。
ファイルの拡張子は `.thrift` で終わる必要があります。

```
my_struct.thrift            // 良い
MyStruct.thrift             // 悪い
my_struct.proto             // 悪い
```

## 構造体名

構造体の名前は大文字の `T` で始まり、新しい単語ごとに大文字を使用し、アンダースコアは使用しません: TMyStruct

```
struct TMyStruct;           // 良い
struct MyStruct;            // 悪い
struct TMy_Struct;          // 悪い
struct TmyStruct;           // 悪い
```

## フィールド名

構造体メンバーの名前はすべて小文字で、単語の間にアンダースコアを入れます。

```
1: optional i64 my_field;       // 良い
1: optional i64 myField;        // 悪い
```