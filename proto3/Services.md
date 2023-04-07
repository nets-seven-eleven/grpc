# Protocol Buffers v3

## 定义服务

```proto3
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}

// 可以在 .proto 文件中定义一个 RPC 服务接口
// protocolbuffer 编译器将以你选择的编译器生成服务接口代码和存根
```

</br>

### 1、gRPC

&emsp;&emsp; 一种由 `Google` 开发的语言和平台中立的开源 `RPC` 系统，`gRPC` 特别适用于协议缓冲区，并允许你的 `.proto` 使用指定的协议缓冲区编译器插件直接从文件生成相关的 `RPC` 代码。

</br>

### 2、JSON 映射

&emsp;&emsp; 1）`Proto3` 支持 `JSON` 中的编码规范，从而更容易在系统之间共享数据。

&emsp;&emsp;&emsp;&emsp; a）当将 `JSON` 编码的数据解析到协议缓冲区时，如果缺少一个值或者它的值为 `null`，他将被解释为相应的默认值。

&emsp;&emsp;&emsp;&emsp; b）从协议缓冲区生成 `JSON` 编码的输出时，如果 `protobuf` 字段具有默认值并且该字段不支持字段存在，则默认情况下它将从输出中省略。实现可以提供选项以在输出中包含具有默认值的字段。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; i）使用 `optional` 关键字定义的 `proto3` 字段支持字段存在。设置了值切支持字段存在的字段始终在 `JSON` 编码输出中包含字段值，即使它是默认值。

</br>

---
| proto3 | JSON	| JSON example | Notes |
| :----: | :---:| :----------: | :---: |
| message |	object | {"fooBar": v, "g": null, …} | 生成 JSON 对象。消息字段名称映射为 lowerCamelCase 并成为 JSON 对象键。如果 json_name 指定了字段选项，则指定的值将用作键。解析器接受 lowerCamelCase 名称（或由选项指定的名称json_name）和原始原型字段名称。null 是所有字段类型的可接受值，并被视为相应字段类型的默认值。 |
| enum | string | "FOO_BAR" | 使用 proto 中指定的枚举值的名称。解析器接受枚举名称和整数值。 |
| map<K,V> | object | {"k": v, …} | 所有键都转换为字符串。 ｜
| repeated V | array | [v, …] | null被接受为空列表[]。 |
| bool | true, false | true, false | |	
| string | string |	"Hello World!" | |
| bytes |	base64 string |	"YWJjMTIzIT8kKiYoKSctPUB+" | JSON 值将是使用带填充的标准 base64 编码编码为字符串的数据。接受带/不带填充的标准或 URL 安全 base64 编码。 |
| int32, fixed32, uint32 | number |	1, -10, 0	| JSON 值将是一个十进制数。接受数字或字符串。 |
| int64, fixed64, uint64 | string	| "1", "-10" | JSON 值将是一个十进制字符串。接受数字或字符串。 |
| float, double |	number | 1.1, -10.0, 0, "NaN", "Infinity" |	JSON 值将是一个数字或特殊字符串值“NaN”、“Infinity”和“-Infinity”之一。接受数字或字符串。指数符号也被接受。-0 被认为等同于 0。 |
| Any |	object | {"@type": "url", "f": v, … } | 如果Any包含具有特殊 JSON 映射的值，它将按如下方式转换：{"@type": xxx, "value": yyy}. 否则，该值将被转换为 JSON 对象，并"@type"插入该字段以指示实际数据类型。 |
| Timestamp |	string | "1972-01-01T10:00:20.021Z" |	使用 RFC 3339，其中生成的输出将始终进行 Z 标准化并使用 0、3、6 或 9 小数位。也接受除“Z”以外的偏移量。 |
| Duration | string |	"1.000340012s", "1s" | 生成的输出始终包含 0、3、6 或 9 个小数位，具体取决于所需的精度，后跟后缀“s”。接受任何小数位（也可以没有），只要它们符合纳秒精度并且需要后缀“s”。 |
| Struct | object | { … } |	任何 JSON 对象。看struct.proto。 |
| Wrapper types |	various types |	2, "2", "foo", true, "true", null, 0, … | 包装器在 JSON 中使用与包装的原始类型相同的表示，除了null在数据转换和传输期间允许和保留。 |
| FieldMask |	string | "f.fooBar,h" |	看 field_mask.proto |
| ListValue |	array | [foo, bar, …]	| |
| Value |	value |	 | 任何 JSON 值。检查 google.protobuf.Value 了解详情。 |
| NullValue	| null |  | JSON null |
| Empty |	object | {}	| 一个空的 JSON 对象。 |

</br>

&emsp;&emsp; 2）JSON 选项

&emsp;&emsp;&emsp;&emsp; `proto3 JSON` 实现可提供以下选项：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; i）___发出具有默认值的字段：___ 在 `proto3 JSON` 输出中默认省略具有默认值的字段。实现可以提供一个选项来覆盖此行为并使用默认值输出字段。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; ii）___忽略未知字段：___ `proto3 JSON` 解析器默认应该拒绝未知字段，但可能会提供一个选项来忽略解析中的未知字段。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; iii）___使用 `proto` 字段名称而不是 `lowerCamelCase` 名称：___ 默认情况下，`proto3 JSON` 打印机应将字段名称转换为 `lowerCamelCase` 并将其用作 `JSON` 名称。一个实现可以提供一个选项来代替使用 `proto` 字段名称作为 `JSON` 名称。`Proto3 JSON` 解析器需要接受转换后的 `lowerCamelCase` 名称和 `proto` 字段名称。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; iiii）___将枚举值作为整数而不是字符串发出：___ `JSON` 输出中默认使用枚举值的名称。可以提供一个选项来代替使用枚举值的数值。

</br>

### 3、选项

&emsp;&emsp; 文件中的单个声明 `.proto` 可以使用多个选项进行注释。选项不会改变声明的整体含义，但可能会影响它在特定上下文中的处理方式。可用选项的完整列表在 [/google/protobuf/descriptor.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) 中定义

&emsp;&emsp; 一些选项是文件级选项，这意味着它们应该写在顶级范围内，而不是在任何消息、枚举或服务定义中。一些选项是消息级选项，这意味着它们应该写在消息定义中。一些选项是字段级选项，这意味着它们应该写在字段定义中。选项也可以写在枚举类型、枚举值、`oneof` 字段、服务类型和服务方法上。
