# Protobuf语法

gRPC推荐使用proto3，本节只介绍常用语法，更多高级使用姿势请参考[官方文档](https://developers.google.com/protocol-buffers/)


## Message定义
一个message类型定义描述了一个请求或相应的消息格式，可以包含多种类型字段。例如定义一个搜索请求的消息格式，每个请求包含查询字符串、页码、每页数目。

```
syntax = "proto3";

message SearchRequest {
    string query = 1;           // 查询字符串
    int32  page_number = 2;     // 页码
    int32  result_per_page = 3; // 每页条数
}
```
首行声明使用的protobuf版本为**proto3**

SearchRequest 定义了三个字段，每个字段声明以分号结尾，可使用双斜线`//`添加注释。

#### 字段类型声明
所有的字段需要前置声明数据类型，上面的示例指定了两个数值类型和一个字符串类型。除了基本的标量类型还有复合类型，如枚举、其它message类型等。

#### 标识符Tags
可以看到，消息的定义中，每个字段都有一个**唯一的数值型标识符**。这些标识符用于标识字段在消息中的二进制格式，使用中的类型不应该随意改动。需要注意的是，[1-15]内的标识在编码时只占用一个字节，包含标识符和字段类型。[16-2047]之间的标识符占用2个字节。建议为频繁出现的消息元素使用[1-15]间的标识符。如果考虑到以后可能或扩展频繁元素，可以预留一些标识符。

最小的标识符可以从1开始，最大到2<sup>29</sup> - 1，或536,870,911。不可以使用[19000－19999]之间的标识符， Protobuf协议实现中预留了这些标识符。在.proto文件中使用这些预留标识号，编译时就会报错。

#### 字段规则
* repeated：标识字段可以重复任意次，类似数组
* proto3不支持proto2中的required和optional

#### 添加更多message类型
一个.proto文件中可以定义多个消息类型，一般用于同时定义多个相关的消息，例如在同一个.proto文件中同时定义搜索请求和响应消息：

```
syntax = "proto3";

// SearchRequest 搜索请求
message SearchRequest {
    string query = 1;           // 查询字符串
    int32  page_number = 2;     // 页码
    int32  result_per_page = 3; // 每页条数
}

// SearchResponse 搜索响应
message SearchResponse {
	...
}
```

#### 添加注释
向.proto文件中添加注释，支持C风格双斜线`//`单行注释

#### 保留字段与标识符
可以使用reserved关键字指定保留字段和保留标识符：

```
message Foo {
    reserved 2, 15, 9 to 11;
    reserved "foo", "bar";
}
```
注意，不能在一个reserved声明中混合字段名和标识符。

#### `.proto`文件编译结果
当使用protocol buffer编译器运行`.proto`文件时，编译器将生成所选语言的代码，用于使用在`.proto`文件中定义的消息类型、服务接口约定等。不同语言生成的代码格式不同：

* C++: 每个`.proto`文件生成一个`.h`文件和一个`.cc`文件，每个消息类型对应一个类
* Java: 生成一个`.java`文件，同样每个消息对应一个类，同时还有一个特殊的`Builder`类用于创建消息接口
* Python: 姿势不太一样，每个`.proto`文件中的消息类型生成一个含有静态描述符的模块，该模块与一个元类*metaclass*在运行时创建需要的Python数据访问类
* Go: 生成一个`.pb.go`文件，每个消息类型对应一个结构体
* Ruby: 生成一个`.rb`文件的Ruby模块，包含所有消息类型
* JavaNano: 类似Java，但不包含`Builder`类
* Objective-C: 每个`.proto`文件生成一个`pbobjc.h`和一个`pbobjc.m`文件
* C#: 生成`.cs`文件包含，每个消息类型对应一个类

各种语言的更多的使用方法请参考[官方API文档](https://developers.google.com/protocol-buffers/docs/reference/overview)


## 数据类型
这里直接引用[官方文档](https://developers.google.com/protocol-buffers/docs/proto3#scalar)的描述：

|.proto | C++ | Java | Python | Go | Ruby | C# |
|-------|-----|------|--------|----|------|----|
|double|double|double|float|float64|Float|double|
|float|float|float|float|float32|Float|float|
|int32|int32|int|int|int32|Fixnum or Bignum|int|
|int64|int64|long|ing/long<sup>[3]</sup>|int64|Bignum|long|
|uint32|uint32|int<sup>[1]</sup>|int/long<sup>[3]</sup>|uint32|Fixnum or Bignum|uint|
|uint64|uint64|long<sup>[1]</sup>|int/long<sup>[3]</sup>|uint64|Bignum|ulong|
|sint32|int32|int|intj|int32|Fixnum or Bignum|int|
|sint64|int64|long|int/long<sup>[3]</sup>|int64|Bignum|long|
|fixed32|uint32|int<sup>[1]</sup>|int|uint32|Fixnum or Bignum|uint|
|fixed64|uint64|long<sup>[1]</sup>|int/long<sup>[3]</sup>|uint64|Bignum|ulong|
|sfixed32|int32|int|int|int32|Fixnum or Bignum|int|
|sfixed64|int64|long|int/long<sup>[3]</sup>|int64|Bignum|long|
|bool|bool|boolean|boolean|bool|TrueClass/FalseClass|bool|
|string|string|String|str/unicode<sup>[4]</sup>|string|String(UTF-8)|string|
|bytes|string|ByteString|str|[]byte|String(ASCII-8BIT)|ByteString|

关于这些类型在序列化时的编码规则请参考 [Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding).

**<sup>[1]</sup>** java

**<sup>[2]</sup>** all

**<sup>[3]</sup>** 64

**<sup>[4]</sup>** Python


## 默认值
* 字符串类型默认为空字符串
* 字节类型默认为空字节
* 布尔类型默认false
* 数值类型默认为0值
* enums类型默认为第一个定义的枚举值，必须是0

针对不同语言的默认值的具体行为参考 [generated code guide](https://developers.google.com/protocol-buffers/docs/reference/overview)

## 枚举(Enum) TODO


## 使用其它Message
```
message SearchResponse {
    repeated Result results = 1;
}

message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
}
```
message支持嵌套使用，作为另一message中的字段类型

#### 导入定义(import)
可以使用import语句导入使用其它描述文件中声明的类型

```
import "others.proto";
```
protocol buffer编译器会在 `-I / --proto_path`参数指定的目录中查找导入的文件，如果没有指定该参数，默认在当前目录中查找。


## Message嵌套
```
message SearchResponse {
    message Result {
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
    repeated Result results = 1;
}
```
内部声明的message类型名称只可在内部直接使用，在外部引用需要前置父级message名称,如`Parent.Type`：

```
message SomeOtherMessage {
    SearchResponse.Result result = 1;
}
```
支持多层嵌套：

```
message Outer {                // Level 0
    message MiddleAA {         // Level 1
        message Inner {        // Level 2
            int64 ival = 1;
            bool  booly = 2;
        }
    }
    message MiddleBB {         // Level 1
        message Inner {        // Level 2
            int32 ival = 1;
            bool  booly = 2;
        }
    }
}
```
## Message更新 TODO


## Map类型
proto3支持map类型声明:

```
map<key_type, value_type> map_field = N;

message Project {...}
map<string, Project> projects = 1;

```

* 键、值类型可以是内置的标量类型，也可以是自定义message类型
* 字段不支持`repeated`属性
* 不要依赖map类型的字段顺序


## 包(Packages)
在`.proto`文件中使用`package`声明包名，避免命名冲突。

```
syntax = "proto3";
package foo.bar;
message Open {...}
```
在其他的消息格式定义中可以使用**包名+消息名**的方式来使用类型，如：

```
message Foo {
    ...
    foo.bar.Open open = 1;
    ...
}
```
在不同的语言中，包名定义对编译后生成的代码的影响不同：

* C++ 中：对应C++命名空间，例如`Open`会在命名空间`foo::bar`中
* Java 中：package会作为Java包名，除非指定了`option jave_package`选项
* Python 中：package被忽略
* Go 中：默认使用package名作为包名，除非指定了`option go_package`选项
* JavaNano 中：同Java
* C# 中：package会转换为驼峰式命名空间，如`Foo.Bar`,除非指定了`option csharp_namespace`选项


## 定义服务(Service)
如果想要将消息类型用在RPC(远程方法调用)系统中，可以在`.proto`文件中定义一个RPC服务接口，protocol buffer编译器会根据所选择的不同语言生成服务接口代码。例如，想要定义一个RPC服务并具有一个方法，该方法接收`SearchRequest`并返回一个`SearchResponse`，此时可以在`.proto`文件中进行如下定义：

```
service SearchService {
    rpc Search (SearchRequest) returns (SearchResponse) {}
}
```

生成的接口代码作为客户端与服务端的约定，服务端必须实现定义的所有接口方法，客户端直接调用同名方法向服务端发起请求。比较蛋疼的是即便业务上不需要参数也必须指定一个请求消息，一般会定义一个空message。


## 选项(Options)
在定义.proto文件时可以标注一系列的options。Options并不改变整个文件声明的含义，但却可以影响特定环境下处理方式。完整的可用选项可以查看`google/protobuf/descriptor.proto`.

一些选项是文件级别的，意味着它可以作用于顶层作用域，不包含在任何消息内部、enum或服务定义中。一些选项是消息级别的，可以用在消息定义的内部。当然有些选项可以作用在字段、enum类型、enum值、服务类型及服务方法中。但是到目前为止，并没有一种有效的选项能作用于这些类型。

一下是一些常用的选择：

* `java_package` (file option)：指定生成java类所在的包，如果在.proto文件中没有明确的声明java_package，会使用默认包名。不需要生成java代码时不起作用
* `java_outer_classname` (file option)：指定生成Java类的名称，如果在.proto文件中没有明确声明java_outer_classname，生成的class名称将会根据.proto文件的名称采用驼峰式的命名方式进行生成。如（foo_bar.proto生成的java类名为FooBar.java）,不需要生成java代码时不起任何作用
* `objc_class_prefix` (file option): 指定Objective-C类前缀，会前置在所有类和枚举类型名之前。没有默认值，应该使用3-5个大写字母。注意所有2个字母的前缀是Apple保留的。


## 基本规范
##### 描述文件以`.proto`做为文件后缀，除结构定义外的语句以分号结尾
* 结构定义包括：message、service、enum
* rpc方法定义结尾的分号可有可无

##### Message命名采用驼峰命名方式，字段命名采用小写字母加下划线分隔方式
	
```
message SongServerRequest {
    required string song_name = 1;
}
```
	
##### Enums类型名采用驼峰命名方式，字段命名采用大写字母加下划线分隔方式

```
enum Foo {
    FIRST_VALUE = 1;
    SECOND_VALUE = 2;
}
```
##### Service与rpc方法名统一采用驼峰式命名


## 详解Go语言编译结果 TODO
**message对应golang中的struct，编译生成go代码后，字段名会转换为驼峰式**

## 编译
通过定义好的`.proto`文件生成Java, Python, C++, Go, Ruby, JavaNano, Objective-C, or C# 代码，需要安装编译器`protoc`。参考Github项目[google/protobuf](https://github.com/google/protobuf)安装编译器.Go语言需要同时安装一个特殊的插件：[golang/protobuf](https://github.com/golang/protobuf/)。

运行命令：
```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

这里只做参考就好，具体语言的编译实例请参考详细文档，其中，Go语言的使用姿势会在其它章节详细说明：

* [C++ generated code reference](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated)
* [Java generated code reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated)
* [Python generated code reference](https://developers.google.com/protocol-buffers/docs/reference/python-generated)
* [Go generated code reference](https://developers.google.com/protocol-buffers/docs/reference/go-generated)
* [Objective-C generated code reference](https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated)
* [C# generated code reference](https://developers.google.com/protocol-buffers/docs/reference/csharp-generated)

吐槽: 照着官方文档一步步操作不一定成功哦！


## 更多
* Any 消息类型
* Oneof 字段
* 自定义Options

这些用法在实践中很少使用，这里不做详细介绍，尤其自定义选项设计高级用法，有需要请参考[官方文档](https://developers.google.com/protocol-buffers/)。