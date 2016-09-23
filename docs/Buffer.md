# Buffer
在 ECMAScript 2015 (ES6) [TypedArray][TypedArray] 之前，JavaScript语言是没有机制去读取和处理二进制文件流的。
Node.js API 引进`Buffer`类是为了让它能处理上下文像TCP流和文件系统操作流一样的8位字节的流。

ES6已经添加了 [TypedArray][TypedArray] ，`Buffer` 实现了 [Uint8Array] API 换句话说在Node.js可以更合理的使用`Buffer`操作。

`Buffer`实例和整数数组都有固定的大小，内存是在 V8的堆上分配的。`Buffer`一旦创建，它的大小就不能被修改。

`Buffer`在 Node.js里面是一个全局对象 , 他不需要已类似 `require('buffer').buffer` 的方式引入使用。

例如：

```js
  //创建一个长度为 10 的 buffer 并且以 0填充
  const buf1 = Buffer.alloc(10); // <Buffer 00 00 00 00 00 00 00 00 00 00>
  
  //创建一个长度为10 的buffer 并且以 1填充
  const buf2 = Buffer.alloc(10 , 1); //<Buffer 01 01 01 01 01 01 01 01 01 01>
  
  //Why
  //Buffer.alloc(10 , 259)
  //<Buffer 03 03 03 03 03 03 03 03 03 03>
  
  //创建一个长度为10且没有初始化的 buffer
  //这样会比 Buffer.alloc() 更快的返回
  //Buffer 有时候可能包含旧数据
  //使用 fill() 或 write() 重写
  const buf3 = Buffer.allocUnsafe(10); //<Buffer 40 8c 34 00 00 00 00 00 c0 87>

  //创建一个内容为 [0x1, 0x2, 0x3] 的Buffer
  const buf4 = Buffer.from([1, 2, 3]);//<Buffer 01 02 03>

  //创建编码为 ASCII [0x74, 0x65, 0x73, 0x74] 的 Buffer
  const buf5 = Buffer.from('test'); //<Buffer 74 65 73 74>

  //创建编码为 UTF-8 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer
  const buf6 = Buffer.from('tést', 'utf8'); //<Buffer 74 c3 a9 73 74>
```

[TypedArray]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray
[Uint8Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array


## Buffer.from(), Buffer.alloc(), 和 Buffer.allocUnsafe()
Node.js V6 版本，可以使用 `Buffer`的构造函数创建，`Buffer`会根据参数的不同选择选择其中的一种分配形式返回。

*   给 `Buffer()` 第一个参数传人数字 （eq: `new Buffer(10)`）,那将分配一个新的指定大小的`buffer`缓冲区对象。这样分配的`Buffer`不会初始化，可能会存在敏感数据。这种 `Buffer` 必须使用 `buffer.fill(0)` 或完全重写 `Buffer` 的方式来完成对缓冲区的初始化。使用这种方式初始化缓冲区是为了提高性能，他们的区别在于需要创建一个快但没初始化的缓冲区还是创建一个慢但是安全的缓冲区。
*   传人字符串、数组或将对象的数据复制到`Buffer`缓冲区。
*   传人返回值为 `Buffer` 的 [ArrayBuffer][ArrayBuffer] , 它们将共享`ArrayBuffer`分配的内存。

因为 `new Buffer()` 的行为会根据传人的第一个参数的类型发生变化，应用程序并不会校验 `new Buffer()` 是传人参数的正确性，或者造成未能正确的初始化 `Buffer` 的内容，无意中在代码里会引入安全和可靠性问题。

为了实现 `Buffer` 实例时更加可靠和减少错误的发生,使用多种方式 `Buffer.from()`,`Buffer.alloc()`，和 `Buffer.allocUnsafe()`方法替换 `new Buffer()`构造函数来创建缓冲区。

开发者应该选择新API中的其中一种方法替换现在存在使用`new Buffer()`的地方。

*   `Buffer.from(array)` 返回一个新的包含8个字节的缓冲区副本。
*   `Buffer.from(arrayBuffer[,byteOffset , [,length]])` 返回一个新的与ArrayBuffer共享分配内存的缓冲区。
*   `Buffer.from(buffer)` 返回一个新的缓冲区内容和提供buffer一样的副本。
*   `Buffer.alloc(size [,fill[,encoding]])` 返回一个已经初始化大小的`Buffer`实例。这个方法值得注意的是它比 `Buffer.allocUnsafe(size)`慢 ，但是它确保`Buffer`实例初始化内容不会包含敏感数据。
*   `Buffer.allocUnsafe(size)` 和 `Buffer.allocUnsafeSlow(size)` 都会返回一个新的指定大小的缓冲区，内容必须使用 `Buffer.fill(0)`或者完全重写中的其中一种进行初始化。`Buffer.allocUnsafe()`会在共享内存池中分配内存，当大小小于等于`Buffer.poolSize`一半的时候返回`Buffer`实例。`Buffer.allocUnsafeSlow()`返回`Buffer`实例不是使用共享内存池分配内存。

### `--zero-fill-buffers` 命令行参数
>   Added in: v5.10.0

Node.js 可以使用 `--zero-fill-buffers` 命令行参数强迫 `new Buffer(size)`,`Buffer.allocUnsafe()`,`Buffer.allocUnsafeSlow()` 或 `new SlowBuffer(size)`自动在 `zero-filled`上创建新的缓冲区。使用这种方法修改默认行为，可能会对性能产生重大的影响。建议只有在新分配的缓冲区不能包含敏感数据的时候才使用 `--zero-fill-buffers`。

例如：

```sh
  $ node --zero-fill-buffers
  > Buffer.allocUnsafe(5);
  <Buffer 00 00 00 00 00>
```

### 是什么让 `Buffer.allocUnsafe()` 和 `Buffer.allocUnsafeSlow()` "不安全" ？
使用 `Buffer.allocUnsafe()` 和 `Buffer.allocUnsafeSlow()` 分配内存的时候是不会初始化的(不会用 0 填充)。这样的设计让申请分配内存非常快，分配内存时可能包含有潜在敏感数据的内存段。使用 `Buffer.allocUnsafe()`创建的缓冲区没有完全重写内存，将有可能会在读取缓冲区数据的时候将老数据暴露出来。

这就是使用 `Buffer.allocUnsafe` 的优势，在使用的时候需要特别注意以免往应用中引入安全漏洞。


[ArrayBuffer]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer
