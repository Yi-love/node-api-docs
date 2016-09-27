# 内容目录

*   [Buffer](#buffer)
    *   [Buffer.from(), Buffer.alloc(), 和 Buffer.allocUnsafe()](#bufferfrom-bufferalloc-和-bufferallocunsafe)
        *   [`--zero-fill-buffers` 命令行参数](#--zero-fill-buffers-命令行参数)
        *   [是什么让 `Buffer.allocUnsafe()` 和 `Buffer.allocUnsafeSlow()` "不安全"？](#是什么让-bufferallocunsafe-和-bufferallocunsafeslow-不安全)
    *   [Buffers and Character Encodings （缓存和字符编码）](#buffers-and-character-encodings-缓存和字符编码)
    *   [Buffers and TypedArray](#buffers-and-typedarray)
    *   [Buffers and ES6 iteration](#buffers-and-es6-iteration)
    *   [Class:Buffer](#classbuffer)
        *   [new Buffer(array)](#new-bufferarray)
        *   [new Buffer(buffer)](#new-bufferbuffer)
        *   [new Buffer(arrayBuffer[,byteOffset[,length]])](#new-bufferarraybufferbyteoffsetlength)
        *   [new Buffer(size)](#new-buffersize)
        *   [new Buffer(string[,encoding])](#new-bufferstringencoding)
        *   [Class Method: Buffer.alloc(size[, fill[, encoding]])](#class-methodbufferallocsizefillencoding)
        *   [Class Method: Buffer.allocUnsafe(size)](#class-methodbufferallocunsafesize)
        *   [Class Method: Buffer.allocUnsafeSlow(size)](#class-methodbufferallocunsafeslowsize)
# Buffer
> Stability: 稳定的

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

## Buffer.from(), Buffer.alloc(), 和 Buffer.allocUnsafe()
Node.js V6 版本，可以使用 `Buffer`的构造函数创建，`Buffer`会根据参数的不同选择选择其中的一种分配形式返回。

*   给 `Buffer()` 第一个参数传人数字 （eq: `new Buffer(10)`）,那将分配一个新的指定大小的`buffer`缓冲区对象。这样分配的`Buffer`不会初始化，可能会存在敏感数据。这种 `Buffer` 必须使用 `buffer.fill(0)` 或完全重写 `Buffer` 的方式来完成对缓冲区的初始化。使用这种方式初始化缓冲区是为了提高性能，他们的区别在于需要创建一个快但没初始化的缓冲区还是创建一个慢但是安全的缓冲区。
*   传人字符串、数组或将对象的数据复制到`Buffer`缓冲区。
*   传人返回值为 `Buffer` 的 [ArrayBuffer][ArrayBuffer] , 它将`ArrayBuffer`内容在共享内存里分配内存。

因为 `new Buffer()` 的行为会根据传人的第一个参数的类型发生变化，应用程序并不会校验 `new Buffer()` 是传人参数的正确性，或者造成未能正确的初始化 `Buffer` 的内容，无意中在代码里会引入安全和可靠性问题。

为了实现 `Buffer` 实例时更加可靠和减少错误的发生,使用多种方式 `Buffer.from()`,`Buffer.alloc()`，和 `Buffer.allocUnsafe()`方法替换 `new Buffer()`构造函数来创建缓冲区。

开发者应该选择新API中的其中一种方法替换现在存在使用`new Buffer()`的地方。

*   `Buffer.from(array)` 返回一个新的包含8个字节的缓冲区副本。
*   `Buffer.from(arrayBuffer[,byteOffset , [,length]])` 返回一个新的`ArrayBuffer`内容在共享内存分配缓冲区。
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

### 是什么让 `Buffer.allocUnsafe()` 和 `Buffer.allocUnsafeSlow()` "不安全"？
使用 `Buffer.allocUnsafe()` 和 `Buffer.allocUnsafeSlow()` 分配内存的时候是不会初始化的(不会用 0 填充)。这样的设计让申请分配内存非常快，分配内存时可能包含有潜在敏感数据的内存段。使用 `Buffer.allocUnsafe()`创建的缓冲区没有完全重写内存，将有可能会在读取缓冲区数据的时候将老数据暴露出来。

这就是使用 `Buffer.allocUnsafe()` 的优势，在使用的时候需要特别注意以免往应用中引入安全漏洞。

[ArrayBuffer]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

## Buffers and Character Encodings （缓存和字符编码）
`Buffer`实例通常用于序列编码字符例如 UTF-8 , UCS2,Base64 甚至 Hex-encoded 编码的数据缓存方面。使用明确的字符编码可以使 `Buffer`实例和二进制Javascript字符串之间来回转换成为可能。

例如：

```js
  const buf = Buffer.from('hello world' , 'ascii');//<Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>

  buf.toString('hex');//'68656c6c6f20776f726c64'

  buf.toString('base64');//'aGVsbG8gd29ybGQ='
```

Node.js最新支持的字符编码格式：

*   `'ascii'` - 仅限于7位的ASCII数据。这种编码非常快但内存暂用大。
*   `'utf-8'` - 多字节编码的Unicode字符。许多web页面和文档都支持使用 utf-8。
*   `'uft16le'` - 2 或 4字节，低位优先的Unicode字符编码格式。支持（U+10000 到 U+10FFFF）。
*   `'ucs2'` - `'utf16le'`的别名。
*   `'base64'` - Base64编码。在创建一个字符串类型的 `Buffer`是，这种编码能正确的解析 "URL and Filename Safe Alphabet" 在 [RFC4648, Section 5][RFC4648Section5]。
*   `'latin1'` - 一种可以对单字节字符串进行编码的方法(as defined by the IANA in RFC1345, page 63, to be the Latin-1 supplement block and C0/C1 control codes)。
*   `'binary'` - `'latin1'` 的别名。
*   `'hex'` - 每个字节对应2个16进制编码。

提示：现在的浏览器跟着 [WHATWG spec][WHATWGspec] 把 'latin1' 和 ISO-8895-1 同时添加 win-1252 这个别名。这意味着当做一些像 `http.get()` 的事时，假如返回的字符编码在WHATWG spec规范中，有可能服务器会返回以win-1252编码的数据，并且使用 `'latin1'`编码格式对当前字符串解码。  

## Buffers and TypedArray
`Buffer` 实例同样是 [Uint8Array][Uint8Array] 的实例。然而，在 ECMAScript 2015 中它们在实现TypedArray规范上还是存在一些小的不兼容。例如，
[ArrayBuffer#slice][ArrayBuffer-Slice]会返回切片部分的副本，[Buffer#slice][Buffer-slice]的实现是复用现在的`Buffer`而不是复制，从而使
[Buffer#slice][Buffer-slice]更高效。

根据如下说明也可以从`Buffer`中创建新的 [TypedArray][TypedArray]实例：

*   `Buffer`对象内存是复制到[TypedArray][TypedArray],不是共享。
*   `Buffer`对象内存被解析成包含特定元素的数组，而不是一个字节数组的目标类型。就是说，`new Uint32Array(Buffer.from([1,2,3,4]))` 通过元素 `[1,2,3,4]`创建了一个4个元素的 `Uint32Array`,而不是一个单个元素 `[0x1020304]` 或 `[0x4030201]`的 `Uint32Array`。

使用TypedArray对象的`.buffer`属性可以创建一个新的与 [TypedArray][TypedArray]实例共享内存的 `Buffer`缓冲区。

例如：

```js
  const arr = new Uint16Array(2);

  arr[0] = 5000;
  arr[1] = 4000;

  // Copies the contents of `arr`
  const buf1 = Buffer.from(arr);

  // Shares memory with `arr`
  const buf2 = Buffer.from(arr.buffer);

  // Prints: <Buffer 88 a0>
  console.log(buf1);

  // Prints: <Buffer 88 13 a0 0f>
  console.log(buf2);

  arr[1] = 6000;

  // Prints: <Buffer 88 a0>
  console.log(buf1);

  // Prints: <Buffer 88 13 70 17>
  console.log(buf2);
```
注意在使用 [TypedArray][TypedArray] 对象的`.buffer`属性的时候，可以传人`byteOffset`和`length`参数来指定使用范围以内的 [ArrayBuffer][ArrayBuffer]。

例如：

```js
  const arr = new Uint16Array(20);
  const buf = Buffer.from(arr.buffer, 0, 16);

  // Prints: 16
  console.log(buf.length);
```

`Buffer.from()` 和 [TypedArray.from()][TypedArray-from] (例如：`Uint8Array.from`)有着不同的特点和实现。特别是, [TypedArray][TypedArray]另一种传参是第二个参数传人一个映射函数匹配参数数组的每一个元素。

*   `TypedArray.from(source[, mapFn[, thisArg]])`

然而 `Buffer.from()`方法，没有提供映射函数：

*   [Buffer.from(array)]
*   [Buffer.from(buffer)]
*   [Buffer.from(arrayBuffer[,byteOffset [,length]])]
*   [Buffer.from(string[,encoding])]

## Buffers and ES6 iteration
`Buffer`实例可以使用 ECMAScript 2015 (ES6) 的 `for..of` 语法迭代器遍历。

例如：

```js
  const buf = Buffer.from([1,2,3]);

  for(var b of buf) {
    console.log(b); // 1 2 3
  }
``` 

另外，[buf.values()] , [buf.keys()] , 和 [buf.entries()]方法都可以用来创建迭代器。


## Class:Buffer
`Buffer`类是一个全局类用来直接处理二进制数据。可以使用多种方式创建。

### new Buffer(array)
>   弃用：v6.0.0+

>   用 `Buffer.from(array)` 代替 

*   `array` [<Array>][Array] 复制数组中的字节。

分配一个新的`Buffer`缓冲区使用8位字节数组。

例如：

```js
  // Creates a new Buffer containing the ASCII bytes of the string 'buffer'
  const buf = new Buffer([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```
### new Buffer(buffer)
>   弃用：v6.0.0+

>   用 ` Buffer.from(buffer)`代替

*   `buffer`[<Buffer>](#class-buffer) 从当前 `Buffer`复制数据

复制缓冲区数据到新的 `Buffer` 实例。

例如:

```js
  const buf1 = new Buffer('buffer');
  const buf2 = new Buffer(buf1);

  buf1[0] = 0x61;

  // Prints: auffer
  console.log(buf1.toString());

  // Prints: buffer
  console.log(buf2.toString());
```

### new Buffer(arrayBuffer[,byteOffset[,length]])
>   弃用：v6.0.0+

>   用 `Buffer.from(arrayBuffer[,byteOffset[,length]]) 代替

*   `arrayBuffer` [<ArrayBuffer>][ArrayBuffer]  [TypedArray][TypedArray] 的`.buffer`属性值或 [ArrayBuffer][ArrayBuffer]。
*   `byteOfset` <Integer> 从那个位置开始复制 `arrayBuffer` 。 默认：开始位置：0。
*   `length`<Integer> 需要从`arrayBuffer`复制多少字节的数据。默认：`arrayBuffer.length-byteOffset`。 

当引用 [TypedArray][TypedArray]实例的`.buffer`属性值时，新创建的缓冲区将与[TypedArray][TypedArray]共享相同部分的内存。

可选`byteOffset`和`length` 参数指定内存范围内的 `arrayBuffer` 将共享`Buffer`缓冲区。

例如：

```js
  const arr = new Uint16Array(2);

  arr[0] = 5000;
  arr[1] = 4000;

  // Shares memory with `arr`
  const buf = new Buffer(arr.buffer);

  // Prints: <Buffer 88 13 a0 0f>
  console.log(buf);

  // Changing the original Uint16Array changes the Buffer also
  arr[1] = 6000;

  // Prints: <Buffer 88 13 70 17>
  console.log(buf);
```

### new Buffer(size)
>   弃用：v6.0.0+

>   用 `Buffer.alloc(size)` ( 通常用 `Buffer.allocUnsafe(size)`)

*   `size` <Integer> 创建 `Buffer` 的长度。

创建分配内存大小为 `size` 大小字节的`Buffer`。`size`必须小于等于可以分配内存的最大限制`buffer.kMaxLength`。反之，则会抛出范围错误。`size <= 0`则会创建长度为0的`Buffer`。

不像 [ArrayBuffer][ArrayBuffer] , `Buffer`实例底下暂用的内存不会在创建的时候初始化。创建的`Buffer`是未知的并且可能包含敏感数据。使用 `buf.fill(0)` 去初始化`Buffer`为 0。

例如：

```js
  const buf = new Buffer(5);

  // Prints (contents may vary): <Buffer 78 e0 82 02 01>
  console.log(buf);

  buf.fill(0);

  // Prints: <Buffer 00 00 00 00 00>
  console.log(buf);
```

### new Buffer(string[,encoding])
>   弃用：v6.0.0+

>   用 `Buffer.from(string[,encoding])` 代替


*   `string` [<String>][String] 需要编码的字符串
*   `encoding` [<String>][String] 编码格式。默认：`'utf-8'`

根据JavaScript字符串内容创建新的 `Buffer` 缓冲区。如果传人了编码格式参数那么将根据提供的编码格式编码字符串。

例如：

```js
  const buf1 = new Buffer('this is a tést');

  // Prints: this is a tést
  console.log(buf1.toString());

  // Prints: this is a tC)st
  console.log(buf1.toString('ascii'));


  const buf2 = new Buffer('7468697320697320612074c3a97374', 'hex');

  // Prints: this is a tést
  console.log(buf2.toString());
```

### Class Method:Buffer.alloc(size[,fill[,encoding]])
>   v5.10.0+

*   `size` <Integer> `Buffer`缓冲区的大小
*   `fill` [<String>][String] | [<Buffer>](#classbuffer) | <Integer> 初始化缓冲区的值。默认为：0
*   `encoding` [<String>][String] 编码格式。默认： `'utf-8'`

创建一个大小为 `size`字节 的缓冲区。如果参数`fill`是`undefined` , `Buffer`缓冲区将使用 `0` 填充。

例如：

```js
  const buf = Buffer.alloc(5);

  // Prints: <Buffer 00 00 00 00 00>
  console.log(buf);
```

`size`参数必须小于等于 [buffer.kMaxLength](#buffer-kMaxLength)，反之，抛出范围错误。`size <= 0`将创建长度等于0的 `Buffer`。

如果提供了 `fill` 参数，创建缓冲区的时候将使用 `buf.fill(fill)`进行初始化。

例如：

```js
  const buf = Buffer.alloc(5, 'a');

  // Prints: <Buffer 61 61 61 61 61>
  console.log(buf);
```

如果参数 `fill` 和 `encoding` 都提供，创建的`Buffer`缓冲区将使用 `buf.fill(fill,encoding)`初始化。

例如：

```js
  const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

  // Prints: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
  console.log(buf);
```

执行 `Buffer.alloc()`是比 `Buffer.allocUnsafe()` 低效率，但是可以确定`Buffer.alloc()`在创建`Buffer`缓冲区实例的内容是不存在敏感数据。

`size`如果不是数字会报类型错误。

### Class Method:Buffer.allocUnsafe(size)
> v5.10.0+

*   `size` `Buffer`缓冲区的大小

`size`参数必须小于等于 [buffer.kMaxLength](#buffer-kMaxLength)，反之，抛出范围错误。`size <= 0`将创建长度等于0的 `Buffer`。

`Buffer`实例底下暂用的内存不会在创建的时候初始化。创建的`Buffer`是未知的并且可能包含敏感数据。使用 `buf.fill(0)` 去初始化`Buffer`为 0。

例如：
```js
  const buf = Buffer.allocUnsafe(5);

  // Prints (contents may vary): <Buffer 78 e0 82 02 01>
  console.log(buf);

  buf.fill(0);

  // Prints: <Buffer 00 00 00 00 00>
  console.log(buf);
```

`size`如果不是数字会报类型错误。

`Buffer`模块会根据`Buffer.poolSize`大小采用预分配创建`Buffer`实例。只有当在使用 [Buffer.allocUnsafe()](#class-methodbufferallocunsafesize) 并且`size`小于等于`Buffer.poolSize`的一半的时候，会采用快速分配创建缓冲区。

和`Buffer.allocUnsafe(size).fill(0)`最关键的不同是`Buffer.alloc(size,fill)`使用预分配内存池。特别是，`Buffer.alloc(size,fill)`从不使用内部缓冲池，当`size`大小小于等于`Buffer.poolSize`的一半的时候`Buffer.allocUnsafe(size).fill(fill)`将使用内部缓冲区。当应用需要`Buffer.allocUnsafe()`提供而外的性能时，这些微小的差异就显得极为重要了。


### Class Method: Buffer.allocUnsafeSlow(size)
> v5.10.0+

*   `size` <Integer> `Buffer`缓冲区的大小

`size`参数必须小于等于 [buffer.kMaxLength](#buffer-kMaxLength)，反之，抛出范围错误。`size <= 0`将创建长度等于0的 `Buffer`。

`Buffer`实例底下暂用的内存不会在创建的时候初始化。创建的`Buffer`是未知的并且可能包含敏感数据。使用 `buf.fill(0)` 去初始化`Buffer`为 0。

当使用 [Buffer.allocUnsafeSlow()](#class-methodbufferallocunsafeslowsize) 创建`Buffer`缓冲区实例，当大小小于4kb时，默认按4kb分配缓冲区。它允许应用创建更多`Buffer`实例时避免更多垃圾回收开销。通过消除需要跟踪和清理持久对象可以提高性能和内存的使用。



[TypedArray]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray
[TypedArray-from]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/from
[Uint8Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array
[RFC4648Section5]: https://tools.ietf.org/html/rfc4648#section-5
[WHATWGspec]: https://encoding.spec.whatwg.org/
[ArrayBuffer-Slice]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer/slice
[Buffer-slice]: https://nodejs.org/dist/latest-v6.x/docs/api/buffer.html#buffer_buf_slice_start_end
[Array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
[ArrayBuffer]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer
[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type