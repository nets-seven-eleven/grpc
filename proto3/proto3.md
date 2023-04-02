# Protocol Buffers v3

## 一、定义消息
```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

// a、该文件的第一行指定您正在使用 `proto3` 语法，如果您不这样做，协议缓冲区编译器将假定您正在使用 `proto2`。这必须是文件的第一个非空、非注释行。

// b、消息中每个字段都有名称和类型（名称/值对）
```

</br>

### 1、指定字段类型

&emsp;&emsp; 1）字段类型可以是 `标量类型`，也可以是 `复合类型`，包括 `枚举` 和 其他复合类型

</br>

### 2、分配字段编号

&emsp;&emsp; 1）消息定义中的每个字段都有一个唯一编号。

&emsp;&emsp;&emsp;&emsp; a）这些字段编号用于在 `二进制格式消息` 中标识字段，一旦消息类型被使用，就不应该更改。

&emsp;&emsp;&emsp;&emsp; b）请注意，`1~15` 范围内的字段编号需要一个字节进行编码，包括字段编号和字段类型。`16~2047` 范围内的字段编号占用两个字节。因此，我们应该为非常频繁出现的消息元素保留数字1~15。也可以为将来可能添加的频繁出现的元素流出一些空间。

&emsp;&emsp;&emsp;&emsp; c）可以指定的 `最小字段编号为1`，`最大字段编号为2^29 - 1，即 536870911`。但是，也不能使用数字 `19000～19999（FieldDescriptor::kFirstReservedNumber到 FieldDescriptor::kLastReservedNumber）`，因为它们是为 `Protocol Buffers` 实现保留的。

</br>

### 3、指定字段规则

&emsp;&emsp; 消息字段可以是以下之一：

&emsp;&emsp;&emsp;&emsp; 1）`singular` ：使用 `proto3` 语法时，当没有为给定字段指定其他字段规则时，这是默认字段规则。一个格式良好的消息可以有零个或一个这个字段（但不超过一个）。你无法确定它是否是从线路中解析出来的。除非它是默认值，否则它将被序列化到线路。

&emsp;&emsp;&emsp;&emsp; 2）`optional` ：与 `singular` 相同，只是你可以检查该值是否已明确设置。字段 `optional` 处于两种可能的状态之一：

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; a）该字段已设置，并包含一个明确设置或从线路解析的值。它将被序列化到线路。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; b）该字段未设置，并将返回默认值。它不会被序列化到线路。

&emsp;&emsp;&emsp;&emsp; 3）`repeated` ：此字段类型可以在格式正确的消息中重复零次或多次。`重复的顺序将被保留`。在 `proto3` 中，`repeated` 标量数值类型的字段默认使用 `packed` 编码。

&emsp;&emsp;&emsp;&emsp; 4）`map` ：这是成对的 `键/值` 字段类型。 

</br>

### 4、添加注释

&emsp;&emsp; 要向 `.proto` 文件添加注释，请使用 `C/C++` 样式 `//` 和 `/* ... */` 语法。

</br>

### 5、保留字段

&emsp;&emsp; 1）如果你通过 `完全删除某个字段` 或 `注释掉` 来更新消息类型，则未来用户可以在对该类型进行自己的更新时重用该字段编号。但是如果他们以后加载相同的旧版本 `.proto`，这可能会导致严重的问题，包括数据损坏、隐私错误等。

&emsp;&emsp; 2）确保这种情况不会发生的一种方法是指定已删除字段的字段编号（和/或名称，这也可能导致Json序列化问题）是 `reserved`。如果任何未来的用户尝试使用这些字段标识符，`protocol buffer` 编译器会抱怨。

```proto
message Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "foo", "bar";
}

// 可以使用关键字指定保留的数值范围到最大可能值 max

// 请注意，你不能在用一 reserved 语句中混合使用字段名称和字段编号
```

</br>

### 6、标量值类型

| .proto Type | Notes | Go Type |
| :---------: | :---: | :-----: |
| double |  | float64 |
| float  |  |   float32 |
| int32 | 使用可变长度编码。编码负数效率低下——如果您的字段可能有负值，请改用 sint32。| int32 |
| int64 | 使用可变长度编码。编码负数效率低下——如果您的字段可能有负值，请改用 sint64。| int64 |
| uint32 | 使用可变长度编码。| unint32 |
| uint64 | 使用可变长度编码。| unint64 |
| sint32 | 使用可变长度编码。有符号的 int 值。这些比常规的 int32 更有效地编码负数。| int32 |
| sint64 | 使用可变长度编码。有符号的 int 值。这些比常规的 int64 更有效地编码负数。| int64 |
| fixed32 | 总是四个字节。如果值通常大于 2^28 ，则比 uint32 更有效。| uint32 |
| fixed64 | 总是八个字节。如果值通常大于 2^56 ，则比 uint64 更有效。| uint64 |
| sfixed32 | 总是四个字节。｜int32 |
| sfixed64 | 总是八个字节。｜int64 |
| bool |  | bool |
| string | 字符串必须始终包含 UTF-8 编码或 7 位 ASCII 文本，并且不能长于 2^32。| string |
| bytes | 可能包含不超过 2^32 的任意字节序列。| []byte |

</br>

### 7、默认值

&emsp;&emsp; 1）解析消息时，如果编码消息不包含特定的单数元素，则解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定于类型的：

&emsp;&emsp;&emsp;&emsp; a）对于字符串，默认值为空字符串。

&emsp;&emsp;&emsp;&emsp; b）对于字节，默认值为空字节。

&emsp;&emsp;&emsp;&emsp; c）对于布尔值，默认值为 false。

&emsp;&emsp;&emsp;&emsp; d）对于数字类型，默认值为零。

&emsp;&emsp;&emsp;&emsp; e）对于枚举类型，默认值时第一个定义的枚举值，它必须是0。

&emsp;&emsp;&emsp;&emsp; f）对于消息字段，该字段未设置。它的确切值取决于语言。

&emsp;&emsp; 2）重复字段的默认值为空（通常是相应语言的空列表）。

&emsp;&emsp; 3）注意，对于标量消息字段，一旦消息被解析，就无法判断某个字段是显示设置为默认值还是根本没有设置。

&emsp;&emsp; 4）如果将标量消息字段设置为其默认值，则该值将不会在线上序列化。

</br>

### 8、枚举

```proto
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  Corpus corpus = 4;
}
```

&emsp;&emsp; 1）`Corpus` 枚举的第一个常量映射到零：每个枚举定义都必须包含一个映射到零的常量作为其第一个元素。这是因为：

&emsp;&emsp;&emsp;&emsp; a）必须有一个零值，这样我们就可以使用0作为数字默认值。

&emsp;&emsp;&emsp;&emsp; b）零值需要是第一个元素，以便与第一个枚举值始终是默认值的 `proto2语义` 兼容。

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2;
}

enum EnumNotAllowingAlias {
  ENAA_UNSPECIFIED = 0;
  ENAA_STARTED = 1;
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message.
  ENAA_FINISHED = 2;
}

```

&emsp;&emsp; 2）可以通过将相同的值分配给不同的枚举常量来定义别名。为此，需要将 `allow_alias` 选项设置为 `true`，否则协议编译器会在发现别名时生成一条警告消息。尽管在反序列化期间所有别名值都有效，但在序列化时始终使用第一个值。

&emsp;&emsp; 3）枚举值常量必须在32位整数范围内。

&emsp;&emsp; 4）由于 `enum` 在线路上使用 `varint` 编码，因此负值效率低下，因此不推荐使用。

&emsp;&emsp; 5）在反序列化过程中，无法识别的枚举值将保留在消息中，尽管在反序列化消息时如何表示取决于语言。

&emsp;&emsp; 6）在支持值超出指定符号范围的开放枚举类型的语言中，未知枚举值仅存储为其基础整数表示形式。

&emsp;&emsp; 7）在任何一种情况下，如果消息被序列化，无法识别的值仍将与消息一起序列化。

</br>

### 9、使用其他消息类型

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

</br>

### 10、导入定义

&emsp;&emsp; 1）通过导入，你可以引用其他 `.proto` 文件中的定义。只需在文件顶部添加一个导入语句：

```proto
import "myproject/other_protos.proto";
```






