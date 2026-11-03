
# [Web逆向] protobuf 逆向学习笔记

- 原帖: https://www.52pojie.cn/forum.php?mod=viewthread&tid=2024402
- 抓取时间: 2026-11-01T22:10:39

## protobuf 笔记

### 什么是protobuf

简单来说，就是一直数据存储传输协议，但是它可以把数据压缩的很小，相对传输速度就会快很多。但是他的这个数据人类是很难看懂的，所以就有了反序列化。

详细语法：[//colobu.com/2017/03/16/Protobuf3-language-guide/" target="_blank" rel="noopener noreferrer nofollow"](https://colobu.com/2017/03/16/Protobuf3-language-guide/:rel="nofollow noopener" href="https)

### 正向开发基本流程(这里是重点，了解正向开发，你才知道怎么反向)

- 编写`proto`文件
- 通过编译器生成，编程语言对应的开发包
- 数据通过开发包序列化为二进制

简单尝试一下：

下载地址:[//github.com/protocolbuffers/protobuf/releases/" target="_blank" rel="noopener noreferrer nofollow"](https://github.com/protocolbuffers/protobuf/releases/:rel="nofollow noopener" href="https)

找到对应版本下载下来，会找到`protoc.exe`,这里你可以把它放到环境变量里面(我懒就没放)。

为了了解js，还要下载js的适配：`npm install -g protoc-gen-js`

手动写一个`test.proto`文件,代码如下：

```
syntax = "proto3";

package example;

message TestMessage {
  string name = 1;
  int32 age = 2;
}
```

执行：`protoc -I=C:\Users\admin\test --js_out=import_style=commonjs,binary:. C:\Users\admin\test\test.proto`

解释如下：

- `-I=C:\Users\admin\test`：指定 `.proto` 文件的搜索路径，让 `protoc` 在这个目录下查找 `.proto` 文件及其依赖。
- `--js_out=import_style=commonjs,binary:.`：指定输出 JavaScript 代码的选项，生成支持 CommonJS 模块和二进制格式的代码，并将生成的文件放在当前目录（`.`）。
- `C:\Users\admin\test\test.proto`：指定要编译的 `.proto` 文件路径。

得到如下代码，我稍微翻译了一下：

```
// source: test.proto

/**
 * @fileoverview
 * @enhanceable
 * @suppress {missingRequire} 报告对隐式类型用法的错误。
 * @suppress {messageConventions} JS 编译器如果变量或字段以 'MSG_' 开头且不是可翻译的消息，会报错。
 * @public
 */

// 生成的代码——请勿编辑！

/* eslint-disable */

// @ts-nocheck

var jspb = require('google-protobuf');

var goog = jspb;

var global =

    (typeof globalThis !== 'undefined' && globalThis) ||

    (typeof window !== 'undefined' && window) ||

    (typeof global !== 'undefined' && global) ||

    (typeof self !== 'undefined' && self) ||

    (function () { return this; }).call(null) ||

    Function('return this')();

goog.exportSymbol('proto.example.TestMessage', null, global);

/**
 * 由 JsPbCodeGenerator 生成。
 * [//www.52pojie.cn/home.php?mod=space&uid=952169" target="_blank"](https://www.52pojie.cn/home.php?mod=space&uid=952169:href="https) {Array=} opt_data 可选的初始数据数组，通常来自服务器响应，或者直接在 JavaScript 中构造。该数组被直接使用，并成为构造对象的一部分。它不会被克隆。
 * 如果未提供数据，则构造的对象将为空，但仍然是有效的。
 * @extends {jspb.Message}
 * @constructor
 */

proto.example.TestMessage = function(opt_data) {

  jspb.Message.initialize(this, opt_data, 0, -1, null, null);

};

goog.inherits(proto.example.TestMessage, jspb.Message);

if (goog.DEBUG && !COMPILED) {

  /**
   * @public
   * @override
   */

  proto.example.TestMessage.displayName = 'proto.example.TestMessage';

}

if (jspb.Message.GENERATE_TO_OBJECT) {

/**
 * 创建此 proto 的对象表示形式。
 * 在 JavaScript 中保留的字段名称将被重命名为 pb_name。
 * 未设置的可选字段将被设置为 undefined。
 * 要访问保留字段，请使用 foo.pb_，例如，foo.pb_default。
 * 有关保留名称的列表，请参见：
 *     net/proto2/compiler/js/internal/generator.cc#kKeyword。
 * @param {boolean=} opt_includeInstance 已废弃。是否包括用于过渡性 soy proto 支持的 JSPB 实例：
 *     http://goto/soy-param-migration
 * [//www.52pojie.cn/home.php?mod=space&uid=155549" target="_blank"](https://www.52pojie.cn/home.php?mod=space&uid=155549:href="https) {!Object}
 */

proto.example.TestMessage.prototype.toObject = function(opt_includeInstance) {

  return proto.example.TestMessage.toObject(opt_includeInstance, this);

};

/**
 * 静态版本的 {@see toObject} 方法。
 * @param {boolean|undefined} includeInstance 已废弃。是否包括用于过渡性 soy proto 支持的 JSPB 实例：
 *     http://goto/soy-param-migration
 * @param {!proto.example.TestMessage} msg 要转换的消息实例。
 * @return {!Object}
 * @suppress {unusedLocalVariables} f 仅用于嵌套消息
 */

proto.example.TestMessage.toObject = function(includeInstance, msg) {

  var f, obj = {

name: jspb.Message.getFieldWithDefault(msg, 1, ""),

age: jspb.Message.getFieldWithDefault(msg, 2, 0)

  };

  if (includeInstance) {

    obj.$jspbMessageInstance = msg;

  }

  return obj;

};

}

/**
 * 反序列化二进制数据（以 protobuf 线格式）。
 * @param {jspb.ByteSource} bytes 要反序列化的字节。
 * @return {!proto.example.TestMessage}
 */

proto.example.TestMessage.deserializeBinary = function(bytes) {

  var reader = new jspb.BinaryReader(bytes);

  var msg = new proto.example.TestMessage;

  return proto.example.TestMessage.deserializeBinaryFromReader(msg, reader);

};

/**
 * 从给定的读取器中将二进制数据（以 protobuf 线格式）反序列化到给定的消息对象中。
 * @param {!proto.example.TestMessage} msg 要反序列化到的消息对象。
 * @param {!jspb.BinaryReader} reader 要使用的 BinaryReader。
 * @return {!proto.example.TestMessage}
 */

proto.example.TestMessage.deserializeBinaryFromReader = function(msg, reader) {

  while (reader.nextField()) {

    if (reader.isEndGroup()) {

      break;

    }

    var field = reader.getFieldNumber();

    switch (field) {

    case 1:

      var value = /** @type {string} */ (reader.readString());

      msg.setName(value);

      break;

    case 2:

      var value = /** @type {number} */ (reader.readInt32());

      msg.setAge(value);

      break;

    default:

      reader.skipField();

      break;

    }

  }

  return msg;

};

/**
 * 将消息序列化为二进制数据（以 protobuf 线格式）。
 * @return {!Uint8Array}
 */

proto.example.TestMessage.prototype.serializeBinary = function() {

  var writer = new jspb.BinaryWriter();

  proto.example.TestMessage.serializeBinaryToWriter(this, writer);

  return writer.getResultBuffer();

};

/**
 * 将给定的消息序列化为二进制数据（以 protobuf 线格式），写入给定的 BinaryWriter。
 * @param {!proto.example.TestMessage} message
 * @param {!jspb.BinaryWriter} writer
 * @suppress {unusedLocalVariables} f 仅用于嵌套消息
 */

proto.example.TestMessage.serializeBinaryToWriter = function(message, writer) {

  var f = undefined;

  f = message.getName();

  if (f.length > 0) {

    writer.writeString(

      1,

      f

    );

  }

  f = message.getAge();

  if (f !== 0) {

    writer.writeInt32(

      2,

      f

    );

  }

};

/**
 * optional string name = 1;
 * @return {string}
 */

proto.example.TestMessage.prototype.getName = function() {

  return /** @type {string} */ (jspb.Message.getFieldWithDefault(this, 1, ""));

};

/**
 * @param {string} value
 * @return {!proto.example.TestMessage} 返回 this
 */

proto.example.TestMessage.prototype.setName = function(value) {

  return jspb.Message.setProto3StringField(this, 1, value);

};

/**
 * optional int32 age = 2;
 * @return {number}
 */

proto.example.TestMessage.prototype.getAge = function() {

  return /** @type {number} */ (jspb.Message.getFieldWithDefault(this, 2, 0));

};

/**
 * @param {number} value
 * @return {!proto.example.TestMessage} 返回 this
 */

proto.example.TestMessage.prototype.setAge = function(value) {

  return jspb.Message.setProto3IntField(this, 2, value);

};

goog.object.extend(exports, proto.example);
```

### 逆向手段

媒体类型是 `proto`，`content-type: application/grpc-web+proto`

我们首先要观察的就是`getFieldWithDefault`和`setProto3StringField`这两个函数可以让我们基本推断出字段类型和编号。

根据`exportSymbol`可以推断出`package`和`message`名称,当然这个你可以根据类似这种来推测`proto.example.TestMessage.prototype`。

`serializeBinary` 和 `serializeBinaryToWriter` (对象序列化为二进制与从二进制反序列化回对象)是明显推断点。

我简单总结一下：

`JS 表现形式` | 
推断的 .proto 类型 | 
说明 | 
`setProto3StringField(...)`
