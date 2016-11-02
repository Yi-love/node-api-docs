# 内容目录

*   [Stream]()
    *   [Organization of this document]()
    *   [Types of Streams]()
        *   [Object Mode]()
        *   [Buffering]()
    *   [API for Stream Consumers]()
        *   [Writable Streams]()
            *   [Class: stream.Writable]()
                *   [Event: 'close']()
                *   [Event: 'drain']()
                *   [Event: 'error']()
                *   [Event: 'finish']()
                *   [Event: 'pipe']()
                *   [Event: 'unpipe']()
                *   [writable.cork()]()
                *   [writable.end([chunk][, encoding][, callback])]()
                *   [writable.setDefaultEncoding(encoding)]()
                *   [writable.uncork()]()
                *   [writable.write(chunk[, encoding][, callback])]()

# Stream
>   Stability:  稳定

在Node.js里面 流是一个为流数据服务的抽象接口。`stream`模块提供基础的API来实现流接口是为了能更好的创建对象。

Node.js提供了很多的流对象。例如， [request to an HTTP server][request-to-an-HTTP-server] 和 [process.stdout][process.stdout] 都是流的实例。

`stream`是可读的，可写的，或者都有。所有的`stream`都是[EventEmitter][EventEmitter]的实例。

`stream`模块可以通过以下方式使用：

```js
  const stream = require('stream');
```

这对Node.js的使用者去理解`stream`是如何工作是非常重要的，`stream`模块本身对开发者创建新类型的流对象实例是很有帮助的。
那些要使用流对象的开发人员将很少(如果有的话)有需要直接使用流模块。

## Organization of this document

本文档分为两个主要部分和第三部分需要注意的地方。第一部分讲应用中使用的流的API。第二部分讲新的实现类型流的API。

## Types of Streams
Node.js有4种基本类型的流：

*   [Readable][Readable] - 可读流（例如： [fs.createReadStream()][fs.createReadStream()]）。
*   [Writable][Writable] - 可写流（例如：[fs.createWriteStream()][fs.createWriteStream()]）。
*   [Duplex][Duplex] - 可读和可写流（例如：[net.Socket][net.Socket]）。
*   [Transform][Transform] - 在可读和可写流的基础上可以对数据进行修改和转换的流（例如：[zlib.createDeflate()][zlib.createDeflate()]）。

### Object Mode
所有流的创建都是由Node.js操作字符串和`Buffer`对象。流可能实现与其它类型的JavaScript值的协同工作（除了`null`，一个流内的特殊目标）。这样的流被认为是在“对象模式”。

当创建流对象实例时可以使用`objectMode`参数切换到对象模式。试图切换现有流到对象模式是不安全的。

### Buffering
保存在内部缓冲区的[Readable][Readable]和[Writable][Writable]数据可以使用各自的`readable._readableState.buffer`和`writable._writableState.getBuffer()`重写读取。

缓冲区数据的总数取决于`stream`构造函数参数`highWaterMark`的大小。常规流，`highWaterMark`表示总字节数。对象模式的流，`highWaterMark`表示对象的总字节数。

当实现调用`stream.push(chunk)`时数据缓冲在可读流。如果`Stream`中的消费者没有调用`stream.read()`，数据会一缓存在内部队列。直到有消费者调用`stream.read()`。

一旦内部读取缓冲区的总大小达到`highWaterMark`指定的阈值,将暂时停止从底层资源读取数据流到当前数据缓冲直到当前缓存已被消耗(即流将停止调用内部`readable._read()`方法,这个方法用于填补读取缓冲区)。

当`writable.write(chunk)`方法被调用多次时数据缓冲在可写流。内部写缓冲区的总大小小于`highWaterMark`指定的阈值时，调用`writable.write()`将返回`true`。否则，返回`false`。

`highWaterMark`在`stream` API中是一个关键目标。特别是在`stream.pipe()`方法中，它限制缓冲区数据在一个可以接受的水平。如此数据来源和目标以不同的速度输入和输出以致不会压垮可用内存。

因为 `Duplex` 和 `Transform` 流都是可读和可写的，每个维持者2个分离的内部缓存用于读和写，允许每一方独立运作的同时保持一个适当的和有效的数据流。例如，`net.Socket`实例是`Duplex`流，可读流允许消费者从套字接口读取数据，另一边可写流允许往套字接口写数据。因为数据可以写入套接字以快或慢的速度比收到数据,这是对于每一方的操作很重要(和缓冲区)独立于其他操作。

## API for Stream Consumers
所有的Node.js应用，不管以多么简单的方式使用`stream`。下面这个例子在Node.js应用中实现一个`HTTP`服务器并使用`stream`:

```js
  const http = require('http');

  const server = http.createServer( (req, res) => {
    // req is an http.IncomingMessage, which is a Readable Stream
    // res is an http.ServerResponse, which is a Writable Stream

    let body = '';
    // Get the data as utf8 strings.
    // If an encoding is not set, Buffer objects will be received.
    req.setEncoding('utf8');

    // Readable streams emit 'data' events once a listener is added
    req.on('data', (chunk) => {
      body += chunk;
    });

    // the end event indicates that the entire body has been received
    req.on('end', () => {
      try {
        const data = JSON.parse(body);
        // write back something interesting to the user:
        res.write(typeof data);
        res.end();
      } catch (er) {
        // uh oh!  bad json!
        res.statusCode = 400;
        return res.end(`error: ${er.message}`);
      }
    });
  });

  server.listen(1337);

  // $ curl localhost:1337 -d '{}'
  // object
  // $ curl localhost:1337 -d '"foo"'
  // string
  // $ curl localhost:1337 -d 'not json'
  // error: Unexpected token o
```

[Writable][Writable]可写流（例如上面的`res`）提供`write()`和`end()`方法把数据写到流上。

[Readable][Readable]可读流使用[EventEmitter][EventEmitter]API对流中数据进行监听，数据可读的时候就通知应用程序代码。可以使用多种方式对流中的数据进行读取。

[Writable][Writable]可写流和 [Readable][Readable] 可读流都是可以在多种方式下使用[EventEmitter][EventEmitter]API与当前流进行通讯。

`Duplex` 和 `Transform` 流都是可读和可写的。

应用程序数据写入或消费的数据流都不需要直接实现流接口并且没有理由使用`require('stream')`。


开发者希望实现新类型的流，应该参考这一部分[API for Stream Implementers][]。

### Writable Streams
可写数据流是一个抽象的数据写入的目的地。

可写流包括：

*   [HTTP requests, on the client][HTTP-requests-on-the-client]
*   [HTTP responses, on the server][HTTP-responses-on-the-server]
*   [fs write streams][fs-write-streams]
*   [zlib streams][zlib-streams]
*   [crypto streams][crypto-streams]
*   [TCP sockets][TCP-sockets]
*   [child process stdin][child-process-stdin]
*   [process.stdout][process.stdout] , [process.stderr][process.stderr]

注意：有的例子实际是实现[Writable][Writable]接口的`Duplex`流。

所有[Writable][Writable]可写流的接口实现来自`stream.Writable`类。

可以使用多种不同的方式实现具体的[Writable][Writable]可写流实例。所有可写流遵循相同的基本使用模式如以下示例中所示：

```js
  const myStream = getWritableStreamSomehow();
  myStream.write('some data');
  myStream.write('some more data');
  myStream.end('done writing data');
```

### Class: stream.Writable
>   v0.9.4+

### Event: 'close'
>   v0.9.4+

底层资源（如文件描述符）关闭的时候触发`close`事件。事件表明,没有更多的事件将被触发，更深层次的计算也不会发生。

不是所有的可写流都会触发`close`事件。

### Event: 'drain'
>   v0.9.4+

如果调用[stream.write(chunk)]()返回`false`,在适当的时机往流里面重新写入数据时`drain`事件将会被触发。

```js
  // Write the data to the supplied writable stream one million times.
  // Be attentive to back-pressure.
  function writeOneMillionTimes(writer, data, encoding, callback) {
    let i = 1000000;
    write();
    function write() {
      var ok = true;
      do {
        i--;
        if (i === 0) {
          // last time!
          writer.write(data, encoding, callback);
        } else {
          // see if we should continue, or wait
          // don't pass the callback, because we're not done yet.
          ok = writer.write(data, encoding);
        }
      } while (i > 0 && ok);
      if (i > 0) {
        // had to stop early!
        // write some more once it drains
        writer.once('drain', write);
      }
    }
  }
```

### Event: 'error'
>   v0.9.4+

*   [\<Error\>][Error]

如果``写或管道数据发送错误将触发`error`事件。 监听函数将返回一个`Error`对象的参数。

注意： 流不会因为`error`事件的触发而关闭。

### Event: 'finish'
>   v0.9.4+

`finish`事件将会在[stream.end()]()方法和所有数据在底层系统都已经完成更新之后触发。

```js
  const writer = getWritableStreamSomehow();
  for (var i = 0; i < 100; i ++) {
    writer.write('hello, #${i}!\n');
  }
  writer.end('This is the end\n');
  writer.on('finish', () => {
    console.error('All writes are now complete.');
  });
```

### Event: 'pipe']
>   v0.9.4+

*   `src` [\<stream.Readable\>][stream.Readable] 数据源流管道可写的

当在可读流上调用[stream.pipe()]()方法时触发`pipe`事件，为目标添加可读性。

```js
  const writer = getWritableStreamSomehow();
  const reader = getReadableStreamSomehow();
  writer.on('pipe', (src) => {
    console.error('something is piping into the writer');
    assert.equal(src, reader);
  });
  reader.pipe(writer);
```

### Event: 'unpipe'
>   v0.9.4+

*   `src` \<[Readable][Readable] Stream \>  移除数据源流管道可写性

当在[Readable][Readable]可读流上调用[stream.unpipe()]()方法时触发`unpipe`事件，移除目标的可读性。

```js
  const writer = getWritableStreamSomehow();
  const reader = getReadableStreamSomehow();
  writer.on('unpipe', (src) => {
    console.error('Something has stopped piping into the writer.');
    assert.equal(src, reader);
  });
  reader.pipe(writer);
  reader.unpipe(writer);
```

### writable.cork()
>   v0.11.2+

`writable.cork()`方法迫使数据缓存到内存。只有[stream.uncork()]()或[stream.end()]()方法被调用了才会将缓存的数据更新到底层。

`write.cork`专注于避免写入过多小块数据的时候不会造成内部缓冲区阻塞的情况。这对性能有负面影响。`writable._writev()`可能更有优势。

### writable.end([chunk][, encoding][, callback])
>   v0.9.4+

*   `chunk` [\<String\>][String] | [\<Buffer\>][Buffer] | \<any\> 写入数据的参数（可选）。流不支持对象操作，`chunk`必须是`String`或者`Buffer`。
对象模式的流，`chunk`可以是任意的JavaScript值出了`null`。
*   `encoding` [\<String\>][String] 编码格式，`chunk`是字符串时有效。
*   `callback` [\<Function\>][Function] 可选参数，流写入完成的时候执行的回调。

执行`writable.end()`方法意味着不再有数据写入`Writable`。可选参数`chunk`和`encoding`参数允许在马上要关闭的流之前写入最后一块添加的数据。
`callback`如果提供，将监听`finish`事件。

在`stream.end()`方法之后调用`stream.write()`将提高错误率的发生。

```js
  // write 'hello, ' and then end with 'world!'
  const file = fs.createWriteStream('example.txt');
  file.write('hello, ');
  file.end('world!');
  // writing more now is not allowed!
```

### writable.setDefaultEncoding(encoding)
> v0.11.15+

*   `encoding` [\<String\>][String] 新的字符编码
*   返回：`this`

`writable.setDefaultEncoding()`方法用于设置`Writable`流的默认编码格式。

### writable.uncork()
>   v0.11.2+

`writable.uncork()`将冲洗所有的缓存数据，从`stream.cork()`调用之后。

当使用`writable.cork()` 和`writable.uncork()`来管理写入流缓存，调用`writable.uncork()`使用延时调用模式`process.nextTick()`将更高效。
这样`writable.write()`将发生在Node.js事件循环阶段。

```js
  stream.cork();
  stream.write('some ');
  stream.write('data ');
  process.nextTick(() => stream.uncork());
```

如果`writable.cork()`方式在流中被多次调用，那么必须以相同次数的`writable.uncork()`来冲洗缓冲区数据。

```js
  stream.cork();
  stream.write('some ');
  stream.cork();
  stream.write('data ');
  process.nextTick(() => {
    stream.uncork();
    // The data will not be flushed until uncork() is called a second time.
    stream.uncork();
  });
```

### writable.write(chunk[, encoding][, callback])
>   v0.9.4+

*   `chunk` [\<String\>][String] | [\<Buffer\>][Buffer] 写入的的数据
*   `encoding` [\<String\>][String] 编码格式 ， 当`chunk`为字符串时有效
*   `callback` [\<Function\>][Function] 当数据被冲洗的时候执行回调函数。
*   返回：[\<Boolean\>][Boolean] 如果希望执行代码等到`drain`事件在写入额外的数据之前触发。 则返回:`false`；否则：`true`。

`writable.write()`方法可以往流中写入数据，一旦数据被处理完成就会执行特定的`callback`回调函数。`callback`可能会或者可能不会把错误作为回调函数的第一个参数。
最可靠的处理错误就是为`error`事件添加监听函数。

返回值表明写入的`chunk`是否被内部缓存或已经超过了流创建是配置的`highWaterMark`值。如果返回`false`，应该停止写入数据，知道`drain`事件被触发之后。

对象模式下`Writable`流会忽略`encoding`参数。

### Readable Streams
可读流是等待被消费的数据。

例如可读流包括：

*   [HTTP responses, on the client][HTTP-responses-on-the-client] 客户端http返回值
*   [HTTP requests, on the server][HTTP-requests-on-the-server] 服务器http请求
*   [fs read streams][fs-read-streams] 读文件流
*   [zlib streams][zlib-streams]  压缩流
*   [crypto streams][crypto-streams] 加密流
*   [TCP sockets][TCP-sockets] TCP套接字
*   [child process stdout and stderr][child-process-stdout-and-stderr] 子进程的标准输出和错误流
*   [process.stdin][process.stdin]

所有的可读流实例都是通过`stream.Readable`类接口实现。

### Two Modes
可读流有效的操作模式有两种:流动和暂停。

流动模式，通过使用事件EventEmitter接口把数据从底层系统自动读取和尽快提供给应用程序。

暂停模式，`stream.read()`方法必须显式地读取流中的数据块。

所有的可读流可以通过以下任意一种方式从暂停模式切换到流动模式：

*   添加`'data'`事件句柄
*   调用`stream.resume()`方法
*   调用`stream.pipe()`方法传输可写数据

可以通过以下任何一种方式切换回暂停模式：

*   如果没有`pipe`终点，调用`stream.pause()`方法
*   如果有`pipe`终点，删除任何`data`事件句柄，使用`stream.unpipe()`方法移除所有`pipe`终点。



================================[未完待续...]======================================

[Buffer]: https://nodejs.org/dist/latest-v6.x/docs/api/buffer.html#buffer_class_buffer
[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type
[request-to-an-HTTP-server]: https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_class_http_incomingmessage
[process.stdout]: https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_process_stdout
[EventEmitter]: https://nodejs.org/dist/latest-v6.x/docs/api/events.html#events_class_eventemitter
[Readable]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_readable
[fs.createReadStream()]: https://nodejs.org/dist/latest-v6.x/docs/api/fs.html#fs_fs_createreadstream_path_options
[Writable]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_writable
[fs.createWriteStream()]: https://nodejs.org/dist/latest-v6.x/docs/api/fs.html#fs_fs_createwritestream_path_options
[Duplex]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_duplex
[net.Socket]: https://nodejs.org/dist/latest-v6.x/docs/api/net.html#net_class_net_socket
[Transform]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_transform
[zlib.createDeflate()]: https://nodejs.org/dist/latest-v6.x/docs/api/zlib.html#zlib_zlib_createdeflate_options
[HTTP-requests-on-the-client]: https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_class_http_clientrequest
[HTTP-responses-on-the-server]: https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_class_http_serverresponse
[fs-write-streams]: https://nodejs.org/dist/latest-v6.x/docs/api/fs.html#fs_class_fs_writestream
[zlib-streams]: https://nodejs.org/dist/latest-v6.x/docs/api/zlib.html
[crypto-streams]: https://nodejs.org/dist/latest-v6.x/docs/api/crypto.html
[TCP-sockets]: https://nodejs.org/dist/latest-v6.x/docs/api/net.html#net_class_net_socket
[child-process-stdin]: https://nodejs.org/dist/latest-v6.x/docs/api/child_process.html#child_process_child_stdin
[process.stdout]: https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_process_stdout
[process.stderr]: https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_process_stderr
[Error]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
[HTTP-responses-on-the-client]: https://nodejs.org/dist/latest-v7.x/docs/api/http.html#http_class_http_incomingmessage
[HTTP-requests-on-the-server]: https://nodejs.org/dist/latest-v7.x/docs/api/http.html#http_class_http_incomingmessage
[fs-read-streams]: https://nodejs.org/dist/latest-v6.x/docs/api/fs.html#fs_class_fs_readstream
[zlib-streams]: https://nodejs.org/dist/latest-v6.x/docs/api/zlib.html
[crypto-streams]: https://nodejs.org/dist/latest-v6.x/docs/api/crypto.html
[TCP-sockets]: https://nodejs.org/dist/latest-v6.x/docs/api/net.html#net_class_net_socket
[child-process-stdout-and-stderr]: https://nodejs.org/dist/latest-v6.x/docs/api/child_process.html#child_process_child_stdout
[process.stdin]: https://nodejs.org/dist/latest-v6.x/docs/api/process.html#process_process_stdin