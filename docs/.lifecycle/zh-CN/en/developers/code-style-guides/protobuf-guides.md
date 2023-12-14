---
displayed_sidebar: "English"
---

# 规则

## 永远不要使用required

随着项目的进行，任何字段都可能变为可选。但如果被定义为必需，就不能被删除。

因此不应该使用`required`。

## 永远不要更改序数

为了向后兼容，字段的序数不应改变。

# 命名

## 文件名

消息的名称全部小写，单词之间用下划线分隔。文件应以`.proto`结尾。


```
my_message.proto            // 好
mymessage.proto             // 差
my_message.pb                // 差
```

## 消息名称

消息名称首字母大写，每个新单词的首字母大写，不使用下划线，并以`PB`作为后缀：MyMessagePB

```protobuf
message MyMessagePB       // 好
message MyMessage         // 差
message My_Message_PB     // 差
message myMessagePB       // 差
```

## 字段名称

消息的名称全部小写，单词之间用下划线分隔。 

```
optional int64 my_field = 3;        // 好
optional int64 myField = 3;         // 差
```