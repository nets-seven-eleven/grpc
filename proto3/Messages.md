# Protocol Buffers v3

## 定义消息
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

&emsp;&emsp; 2）可以导入 `proto2` 消息类型并在 `proto3` 消息中使用它们，反之亦然。但是，不能在 `proto3` 语法中直接使用 `proto2 枚举` （如果导入的 `proto2` 消息使用它们也没关系）

</br>

### 11、嵌套类型

&emsp;&emsp; 可以在其他消息类型中定义和使用消息类型：

```proto
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

&emsp;&emsp; 如果要在其父类型之外重用此类消息，请将其称为 `_Parent_._Type_` ：

```proto 
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}

```

&emsp;&emsp; 你可以根据需要嵌套消息：

```proto
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

</br>

### 12、更新消息类型

&emsp;&emsp; 请记住以下规则：

&emsp;&emsp;&emsp;&emsp; a）不要更改任何现有字段的字段编号。

&emsp;&emsp;&emsp;&emsp; b）如果添加新字段，则任何使用“旧”消息格式由代码序列化的消息仍然可以由新生成的代码解析。您应该记住这些元素的默认值，以便新代码能够正确地与旧代码生成的消息交互。类似地，由新代码创建的消息可以由旧代码解析：旧的二进制文件在解析时只需忽略新字段。

&emsp;&emsp;&emsp;&emsp; c）只要在更新的消息类型中不再使用字段号，就可以删除字段。您可能需要重命名字段，也许可以添加前缀“OBSOLETE_”，或者保留字段编号，这样.proto的未来用户就不会意外地重复使用该编号。

&emsp;&emsp;&emsp;&emsp; d）int32、uint32、int64、uint64和bool都是兼容的，这意味着您可以将字段从其中一种类型更改为另一种类型，而不会破坏向前或向后的兼容性。如果从连线中解析出一个不适合对应类型的数字，则会得到与在C++中将该数字强制转换为该类型相同的效果（例如，如果将64位数字读取为int32，则会将其截断为32位）。

&emsp;&emsp;&emsp;&emsp; e）sint32和sint64彼此兼容，但与其他整数类型不兼容。

&emsp;&emsp;&emsp;&emsp; f）字符串和字节是兼容的，只要字节是有效的UTF-8。

&emsp;&emsp;&emsp;&emsp; g）如果字节包含消息的编码版本，则嵌入消息与字节兼容。

&emsp;&emsp;&emsp;&emsp; h）fixed32与sfixed32兼容，fixed64与sfixer64兼容。

&emsp;&emsp;&emsp;&emsp; i）对于字符串、字节和消息字段，可选与重复兼容。给定重复字段的序列化数据作为输入，如果该字段是基元类型字段，则期望该字段为可选字段的客户端将获取最后一个输入值，或者如果它是消息类型字段，将合并所有输入元素。请注意，这对于包括布尔和枚举在内的数字类型通常是不安全的。数字类型的重复字段可以以压缩格式序列化，当需要可选字段时，压缩格式将无法正确解析。

&emsp;&emsp;&emsp;&emsp; j）enum在有线格式方面与int32、uint32、int64和uint64兼容（请注意，如果值不合适，则会被截断）。然而，请注意，当消息被反序列化时，客户端代码可能会对它们进行不同的处理：例如，无法识别的proto3枚举类型将保留在消息中，但当消息被反串行化时，如何表示这一点取决于语言。Int字段总是保持其值。

&emsp;&emsp;&emsp;&emsp; k）将单个可选字段或扩展更改为新字段或扩展的成员是二进制兼容的，但对于某些语言（尤其是Go），生成的代码的API将以不兼容的方式更改。出于这个原因，谷歌没有在其公共API中做出这样的更改，正如AIP-180中所记录的那样。关于源代码兼容性，同样需要注意的是，如果您确信没有代码一次设置多个字段，那么将多个字段移动到一个新的字段中可能是安全的。将字段移动到的现有字段中是不安全的。同样，将其中一个字段更改为可选字段或扩展也是安全的。

</br>

### 13、未知字段

&emsp;&emsp; 1）未知字段是格式良好的协议缓冲区序列化数据，表示解析器无法识别的字段。例如，当一个旧二进制文件用新字段解析新二进制文件发送的数据时，这些新字段将成为旧二进制文件中的未知字段。

&emsp;&emsp; 2）最初，`proto3` 消息在解析过程中总是丢弃未知字段，但在 `3.5` 版本中，我们重新引入了未知字段的保留，以匹配 `proto2` 的行为。在 `3.5` 及更高版本中，解析过程中会保留未知字段，并将其包含在序列化输出中。

</br>

### 14、Any

&emsp;&emsp; 1）`Any` 消息类型允许您将消息用作嵌入类型，而不需要它们的 `.proto` 定义。`Any` 包含以字节形式的任意序列化消息，以及作为该消息的全局唯一标识符并解析为该消息类型的 `URL`。要使用 `Any` 类型，您需要导入`google/protobuf/Any.proto`。

```proto3
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

&emsp;&emsp; 2）给定消息类型的默认类型 `URL` 是：`type.googleapis.com/_packagename_._messagename_`

&emsp;&emsp; 3）不同的语言实现将支持运行时库助手以类型安全的方式打包和解包Any值——例如，在 `Java` 中，类型`Any`将有特殊的 `pack()` 和 `unpack()` 访问器，而在 `C++` 中有 `PackFrom()` 和 `UnpackTo()` 方法。

</br>

### 15、Oneof

&emsp;&emsp; 1）如果您有一条包含多个字段的消息，并且最多同时设置一个字段，则可以使用 `oneof` 功能强制执行此行为并节省内存。

&emsp;&emsp; 2）`oneof` 字段与常规字段类似，不同之处在于oneof 中的所有字段共享内存，并且最多可以同时设置一个字段。设置 `oneof` 的任何成员会自动清除所有其他成员。根据您选择的语言，您可以使用特殊 `case()`或方法检查 `oneof` 中的哪个值被设置（如果有） 。`WhichOneof()`

&emsp;&emsp; 3）注意，如果设置了多个值，`proto` 中顺序确定的最后一个设置值将覆盖之前的所有值。

&emsp;&emsp; 4）使用 `Oneof`，你可以添加除 `map` 和 `repeated` 字段外的任何类型的字段到 `oneof` 的定义中：

```proto3
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

&emsp;&emsp; 5）Oneof 的特点：

&emsp;&emsp;&emsp;&emsp; a）设置一个 `oneof` 字段将自动清除 `oneof` 的所有其他成员。因此，如果您设置了多个 `oneof` 字段，则只有您设置的最后一个字段仍然有值。

&emsp;&emsp;&emsp;&emsp; b）如果解析器在网络中遇到同一个 `oneof` 的多个成员，则在解析的消息中只使用最后看到的成员。

&emsp;&emsp;&emsp;&emsp; c）`Oneof` 不能是 `repeated`。

&emsp;&emsp;&emsp;&emsp; d）反射 `API` 适用于其中一个字段。

&emsp;&emsp;&emsp;&emsp; e）如果将 `oneof` 字段设置为默认值（例如将 `int32 oneof` 字段设置为 `0`），将设置该 `oneof` 字段的“大小写”，并且该值将在线上序列化。

&emsp;&emsp;&emsp;&emsp; f）

&emsp;&emsp; 6）向后兼容问题：添加或删除其中一个字段时要小心。如果检查 `oneof` 的值返回 `None/ NOT_SET`，则可能意味着 `oneof` 尚未设置或已设置为不同版本的 `oneof` 中的字段。无法区分差异，因为无法知道线路上的未知字段是否是 `oneof` 的成员。

&emsp;&emsp; 7）标记重用问题：

&emsp;&emsp;&emsp;&emsp; a）将字段移入或移出 `oneof`：在消息被序列化和解析后，您可能会丢失一些信息（一些字段将被清除）。但是，您可以安全地将单个字段移动到新的 `oneof` 中，并且如果知道只有一个字段被设置过，则可以移动多个字段。

&emsp;&emsp;&emsp;&emsp; b）删除 `oneof` 字段并将其添加回：这可能会在消息被序列化和解析后清除您当前设置的 `oneof` 字段。

&emsp;&emsp;&emsp;&emsp; c）拆分或合并 `Oneof`：这与移动常规字段有类似的问题。

</br>

### 16、Maps

```proto3
// 创建一个关联映射作为数据定义的一部分，protocol buffers 提供了一个方便的快捷语法：
map<key_type, value_type> map_field = N;

// 其中，key_type 可以是任何整数或字符串类型（因此，除了浮点类型和字节之外的任何标量类型）。
// 请注意，enum 不是有效的 key_type。value_type 可以是除其他映射之外的任何类型。
```

&emsp;&emsp; 1）`map` 字段不能是 `repeated`。

&emsp;&emsp; 2）`map` 值的有线格式排序和 `map` 迭代排序是未定义的，因此您不能依赖 `map` 项的特定顺序。

&emsp;&emsp; 3）为 a 生成文本格式时.proto，`map` 将按 `key` 排序。数字键按数字排序。

&emsp;&emsp; 4）从线路解析或合并时，如果存在重复的 `map keys` ，则使用最后看到的 `key`。从文本格式解析 `map` 时，如果有重复的 `key`，解析可能会失败。

&emsp;&emsp; 5）如果您为 `map` 字段提供键但不提供值，则序列化该字段时的行为取决于语言。在 `C++`、`Java`、`Kotlin` 和 `Python` 中，类型的默认值是序列化的，而在其他语言中，什么都没有序列化。

&emsp;&emsp; 6）向后兼容性：

&emsp;&emsp;&emsp;&emsp; a）`map` 语法等同于以下在线，因此不支持 `map` 的协议缓冲区实现仍然可以处理您的数据：

```proto3
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

&emsp;&emsp;&emsp;&emsp; b）任何支持 `map` 的协议缓冲区实现都必须生成和接受上述定义可以接受的数据。

</br>

### 17、Packages

&emsp;&emsp; 1）您可以向 `.proto` 文件添加可选说明符 `package`，以防止协议消息类型之间的名称冲突。

```proto3
package foo.bar;
message Open { ... }
```

&emsp;&emsp; 2）然后，您可以在定义消息类型的字段时使用 `package` 说明符：

```proto3
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

&emsp;&emsp; 3）在 `Go` 中，除非在 `.proto` 文件中明确提供选项 `go_package` ，否则该包将用作 `Go` 包名称。

&emsp;&emsp; 4）包和名称解析：

&emsp;&emsp;&emsp;&emsp; a）`protocol buffer` 语言中的类型名称解析像 `C++` 一样工作：首先搜索最内层的作用域，然后搜索下一个最内层的范围，依此类推，每个包都被认为是其父包的“内部”。领先的“。” （例如，`.foo.bar.Baz`）表示从最外层的作用域开始。

&emsp;&emsp;&emsp;&emsp; b）协议缓冲区编译器通过解析导入的 `.proto` 文件来解析所有类型名称。每种语言的代码生成器都知道如何引用该语言中的每种类型，即使它具有不同的范围规则。




