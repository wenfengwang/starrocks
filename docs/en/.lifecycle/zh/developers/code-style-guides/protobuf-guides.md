---
displayed_sidebar: English
---

# 规则

## 永远不使用 required

随着项目的发展，任何字段都可能变成可选的。但如果将其定义为必需的，就不能将其删除。

因此不应使用 `required`。

## 永远不改变序数

为了保持向后兼容性，字段的序数不应更改。

# 命名

## 文件名

消息的名称都是小写的，单词之间用下划线分隔。
文件应以 `.proto` 结尾。

```
my_message.proto            // 正确
mymessage.proto             // 错误
my_message.pb                // 错误
```

## 消息名称

消息名称以大写字母开头，每个新单词的首字母大写，不带下划线，并以 `PB` 作为后缀：MyMessagePB

```protobuf
message MyMessagePB       // 正确
message MyMessage         // 错误
message My_Message_PB     // 错误
message myMessagePB       // 错误
```

## 字段名称

消息的名称都是小写的，单词之间用下划线分隔。 

```
optional int64 my_field = 3;        // 正确
optional int64 myField = 3;         // 错误
```