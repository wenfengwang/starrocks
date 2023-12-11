---
displayed_sidebar: "Japanese"
---

# ルール

## 使用禁止：`required`

プロジェクトは進行するにつれて、任意のフィールドが必須ではなくなる可能性があります。ただし、それを必須と定義してしまった場合、削除することはできません。

したがって、`required` は使用しないでください。

## 順序変更禁止

後方互換性を保つために、フィールドの順序を変更してはいけません。

# 命名規則

## ファイル名

メッセージの名前はすべて小文字で、単語と単語の間にはアンダースコアを使用します。
ファイルは `.proto` で終わらせる必要があります。

```
my_message.proto            // 良い
mymessage.proto             // 悪い
my_message.pb                // 悪い
```

## メッセージ名

メッセージ名は大文字で始まり、新しい単語ごとに大文字で始め、アンダースコアを使用せず、末尾に `PB` を付けます： MyMessagePB

```protobuf
message MyMessagePB       // 良い
message MyMessage         // 悪い
message My_Message_PB     // 悪い
message myMessagePB       // 悪い
```

## フィールド名

メッセージの名前はすべて小文字で、単語と単語の間にはアンダースコアを使用します。

```
optional int64 my_field = 3;        // 良い
optional int64 myField = 3;         // 悪い
```