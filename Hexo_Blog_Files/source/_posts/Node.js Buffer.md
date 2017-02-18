---
title: Node.js Buffer缓冲区(4)
date: 2016-06-27 15:54:06
tags: node
categories: nodeJS学习笔记
---

### 4、Node.js Buffer(缓冲区)
JavaScript 语言自身只有字符串数据类型，没有二进制数据类型。但在处理像TCP流或文件流时，必须使用到二进制数据。因此在 Node.js中，定义了一个 Buffer 类，该类用来创建一个专门存放二进制数据的缓存区。一个 `Buffer` 类似于一个整数数组，但它对应于 V8 堆内存之外的一块原始内存。
<!-- more -->
#### 创建Buffer类
``` javascript
  var buf = new Buffer(10);//创建长度为10字节的Buffer实例
  var buf = new Buffer([10, 20, 30, 40, 50]);//通过给定的数组创建Buffer实例
  var buf = new Buffer("www.runoob.com", "utf-8");//通过一个字符串来创建Buffer实例
```
#### 写入缓冲区
``` javascript
  buf.write(string, [offset], [length], [encoding]);
```
参数：
  `string` - 写入缓冲区的字符串。
  `offset` - 缓冲区开始写入的索引值，默认为0.
  `length` - 写入的字节数，默认为buffer.length
  `encoding` - 使用的编码。默认‘utf-8’
返回值：返回实际写入的大小。如果 buffer 空间不足， 则只会写入部分字符串。
#### 从缓冲区读取数据
``` javascript
  buf.toString([encoding], [start], [end]);
```
参数：
  `encoding` - 使用的编码。默认‘utf-8’.
  `start` - 指定开始读取的索引位置，默认为0.
  `end` - 结束位置，默认缓冲区末尾。
返回值：解码缓冲区数据并使用指定的编码返回字符串。
#### 将 Buffer 转换为 JSON 对象
``` javascript
  buf.toJSON();
```
返回值：返回JSON对象。
#### 缓冲区合并
``` javascript
  Buffer.concat(list, [totalLength]);
```
参数：
  `list` - 用于合并的Buffer对象数组列表。
  `totalLength` - 指定合并后Buffer对象的总长度。
返回值：返回一个多个成员合并的新 Buffer 对象。
#### 缓冲区比较
``` javascript
  buf.compare(otherBuffer);
```
返回值：返回一个数字，表示 buf 在 otherBuffer 之前，之后或相同。
#### 拷贝缓冲区
``` javascript
  buf.copy(targetBuffer,[targetStart],[sourceStart],[sourceEnd]);
```
参数：
  `targetBuffer` - 要拷贝的 Buffer 对象。
   `targetStart` - 数字, 可选, 默认: 0
   `sourceStart` - 数字, 可选, 默认: 0
     `sourceEnd` - 数字, 可选, 默认: buffer.length
返回值：无。
#### 缓冲区剪裁
``` javascript
  buf.slice([start],[end]);
```
参数：
  `start` - 数字, 可选, 默认: 0
  `end` - 数字, 可选, 默认: buffer.length
返回值：返回一个新的缓冲区，它和旧缓冲区指向同一块内存，但是从索引 start 到 end 的位置剪切。
#### 缓冲区长度
``` javascript
  buf.length;
```
返回值：返回Buffer对象所占据的内存长度。
#### Node.js Buffer 模块常用的方法
  `new Buffer(size)`;//分配一个新的 size 大小单位为8位字节的 buffer。 注意, size 必须小于 kMaxLength，否则，将会抛出异常 RangeError。
  `new Buffer(buffer)`;//拷贝参数 buffer 的数据到 Buffer 实例。
  `new Buffer(str, [encoding])`;//分配一个新的 buffer ，其中包含着传入的 str 字符串。 encoding 编码方式默认为 'utf8'。
  `buf.length`;//返回这个 buffer 的 bytes 数。注意这未必是 buffer 里面内容的大小。length 是 buffer 对象所分配的内存数，它不会随着这个 buffer 对象内容的改变而改变。
  `buf.toString([encoding], [start], [end])`;//根据 encoding 参数（默认是 'utf8'）返回一个解码过的 string 类型。还会根据传入的参数 start (默认是 0) 和 end (默认是 buffer.length)作为取值范围。
  `buf.toJSON()`;//将 Buffer 实例转换为 JSON 对象。
  `buf[index]`;//获取或设置指定的字节。返回值代表一个字节，所以返回值的合法范围是十六进制0x00到0xFF 或者十进制0至 255。
  `buf.equals(otherBuffer)`;//比较两个缓冲区是否相等，如果是返回 true，否则返回 false。
  `buf.compare(otherBuffer)`;//比较两个 Buffer 对象，返回一个数字，表示 buf 在 otherBuffer 之前，之后或相同。
  `buf.copy(targetBuffer, [targetStart], [sourceStart], [sourceEnd])`;//buffer 拷贝，源和目标可以相同。 targetStart 目标开始偏移和 sourceStart 源开始偏移默认都是 0。 sourceEnd 源结束位置偏移默认是源的长度 buffer.length 。
  `buf.slice([start, [end]])`;//剪切 Buffer 对象，根据 start(默认是 0 ) 和 end (默认是 buffer.length ) 偏移和裁剪了索引。 负的索引是从 buffer 尾部开始计算的。
  `buf.fill(value, [offset], [end])`;//使用指定的 value 来填充这个 buffer。如果没有指定 offset (默认是 0) 并且 end (默认是 buffer.length) ，将会填充整个buffer。
  `buf.write(string, [offset], [length], [encoding])`;//根据参数 offset 偏移量和指定的 encoding 编码方式，将参数 string 数据写入buffer。 offset 偏移量默认值是 0, encoding 编码方式默认是 utf8。 length 长度是将要写入的字符串的 bytes 大小。 返回 number 类型，表示写入了多少 8 位字节流。如果 buffer 没有足够的空间来放整个 string，它将只会只写入部分字符串。 length 默认是 buffer.length - offset。 这个方法不会出现写入部分字符。
  `buf.writeUIntLE(value, offset, byteLength, [noAssert])`;//将value 写入到 buffer 里， 它由offset 和 byteLength 决定。noAssert 值为 true 时，不再验证 value 和 offset 的有效性。 默认是 false。/* 下同。*/
  `buf.writeUIntBE(value, offset, byteLength, [noAssert])`;
  `buf.writeUInt8(value, offset, [noAssert])`;
  `buf.writeUInt16LE(value, offset, [noAssert])`;
  `buf.writeUInt16BE(value, offset, [noAssert])`;
  `buf.writeUInt32LE(value, offset, [noAssert])`;
  `buf.writeUInt32BE(value, offset, [noAssert])`;
  `buf.writeIntLE(value, offset, byteLength, [noAssert])`;
  `buf.writeIntBE(value, offset, byteLength, [noAssert])`;
  `buf.writeInt8(value, offset, [noAssert])`;
  `buf.writeInt16LE(value, offset, [noAssert])`;
  `buf.writeInt16BE(value, offset, [noAssert])`;
  `buf.writeInt32LE(value, offset, [noAssert])`;
  `buf.writeInt32BE(value, offset, [noAssert])`;
  `buf.writeFloatLE(value, offset, [noAssert])`;
  `buf.writeFloatBE(value, offset, [noAssert])`;
  `buf.writeDoubleLE(value, offset, [noAssert])`;
  `buf.writeDoubleBE(value, offset, [noAssert])`;
  `buf.readUInt8(offset, [noAssert])`;//读取。
  `buf.readUInt16LE(offset, [noAssert])`;
  `buf.readUInt16BE(offset, [noAssert])`;
  `buf.readUInt32LE(offset, [noAssert])`;
  `buf.readUInt32BE(offset, [noAssert])`;
  `buf.readUIntLE(offset, byteLength, [noAssert])`;
  `buf.readUIntBE(offset, byteLength, [noAssert])`;
  `buf.readIntLE(offset, byteLength, [noAssert])`;
  `buf.readIntBE(offset, byteLength, [noAssert])`;
  `buf.readInt8(offset, [noAssert])`;
  `buf.readInt16LE(offset, [noAssert])`;
  `buf.readInt16BE(offset, [noAssert])`;
  `buf.readInt32LE(offset, [noAssert])`;
  `buf.readInt32BE(offset, [noAssert])`;
  `buf.readFloatLE(offset, [noAssert])`;
  `buf.readFloatBE(offset, [noAssert])`;
  `buf.readDoubleLE(offset, [noAssert])`;
  `buf.readDoubleBE(offset, [noAssert])`;