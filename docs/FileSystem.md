# 内容目录

*   [File System](#file-system)
    *   [Buffer API](#buffer-api)
    *   [Class:fs.FSWatcher](#classfsfswatcher)
        *   [Event: 'change'](#event-change)
        *   [Event: 'error'](#event-error)
        *   [watcher.close()](#watcherclose)
    *   [Class: fs.ReadStream](#class-fsreadstream)
        *   [Event: 'open']()
        *   [Event: 'close']()
        *   [readStream.bytesRead]()
        *   [readStream.path]()
    *   [Class: fs.Stats]()
        *   [Stat Time Values]()
    *   [Class: fs.WriteStream]()
        *   [Event: 'open']()
        *   [Event: 'close']()
        *   [writeStream.bytesWritten]()
        *   [writeStream.path]()

# File System
>   Stability: 稳定

文件I / O是由简单的包装标准POSIX函数提供的。通过`require('fs')`引入。所有的方法都异步和同步两种形式。

异步形式总是把最后一个参数当作回调函数在完成时执行。完成回调传递过来的参数依赖于方法，但第一个参数是总是留给异常情况。当操作成功完成，那第一个参数将会被置为`null`或`undefined`。

当使用同步形式进行操作产生的异常会直接抛出。可以使用`try/catch`处理异常或继续通过`throw`将异常往上一层抛。

异步例子：

```js
  const fs = require('fs');

  fs.unlink('/tmp/hello', (err) => {
    if (err) throw err;
    console.log('successfully deleted /tmp/hello');
  });
```

同步例子：

```js
  const fs = require('fs');

  fs.unlinkSync('/tmp/hello');
  console.log('successfully deleted /tmp/hello');
```

异步方法是没法保证执行顺序的。所以下面的会报错：

```js
  fs.rename('/tmp/hello', '/tmp/world', (err) => {
    if (err) throw err;
    console.log('renamed complete');
  });
  fs.stat('/tmp/world', (err, stats) => {
    if (err) throw err;
    console.log(`stats: ${JSON.stringify(stats)}`);
  });
```

保证`fs.rename`在`fs.stat`之前执行的正确方法是采用链式回调方法：

```js
  fs.rename('/tmp/hello', '/tmp/world', (err) => {
    if (err) throw err;
    fs.stat('/tmp/world', (err, stats) => {
      if (err) throw err;
      console.log(`stats: ${JSON.stringify(stats)}`);
    });
  });
```

在高负载的进程中，编程人员强烈建议使用异步回调的方式。同步方式会阻塞程序执行直到它完成操作--停止所有链接。

可以使用相对路径的文件名。不过需要记住，路径是相对于`process.cwd()`来的。

大多数`fs`函数让你忽略了回调参数。如果你这样做，函数将会使用默认的回调函数来重新抛出异常。为了方便查看异常位置，可以通过设置`NODE_DEBUG`环境变量实现：

```js
  $ cat script.js
  function bad() {
    require('fs').readFile('/');
  }
  bad();

  $ env NODE_DEBUG=fs node script.js
  fs.js:88
          throw backtrace;
          ^
  Error: EISDIR: illegal operation on a directory, read
      <stack trace.>
```

## Buffer API
> v6.0.0+

`fs`函数对字符串类型和`Buffer`类型的路径都能解析。`Buffer`是为了兼容不支持`'utf-8'`的文件系统。最典型的用法，对`Buffer`路径的手动转换不是必须的，`'utf-8'`编码的`Buffer`可以根据字符串API自动转换。

注意，某些文件系统（例如：`NTFS` 和 `HFS+`）的文件名称编码是`'utf-8'`。那些不支持`'utf-8'`编码`Buffer`的文件系统`fs`函数就不能正常工作了。

## Class:fs.FSWatcher
> v0.5.8+

[fs.watch()]()返回一个对象。

`fs.watch()`函数的回调监听器`listener`用于接收来自FSWatcher's的`change`事件。

对象本身发出这些事件:

### Event: 'change'
>   v0.5.8+

*   `eventType` [\<String\>][String] `fs`事件类型
*   `filname`  [\<String\>][String] | [\<Buffer\>](./Buffer.md/#buffer) 发生改变的文件名(如果关联/可用)

当监听的文件或目录发生改变时触发。更多的详细信息可以通过[fs.watch()]查看。

文件名`filename`参数不会依赖操作系统支持。如果`fs.watch()`函数添加了参数`endcoding`等于`buffer`，那么`filname`的值将会以`Buffer`编码格式返回,反之`filname`为字符串。

```js
  // Example when handled through fs.watch listener
  fs.watch('./tmp', {encoding: 'buffer'}, (eventType, filename) => {
    if (filename)
      console.log(filename);
      // Prints: <Buffer ...>
  });
```

### Event: 'error'
>   v0.5.8+

*   `error` [\<Error\>][Error]

发生错误时发出。

### watcher.close()
>   v0.5.8+

停止对`fs.FSWatchers`改变的监听。

## Class: fs.ReadStream
>  v0.1.93+

`ReadStream` 是一个[可读流][stream-readable]。

### Event:'open'
>   v0.1.93+

*   `fd`<Integer> `ReadStream`中使用的文件描述符。

### Event:'close'
>   v0.1.93+

使用`fs.close()`方法触发`ReadStream`关闭底层文件描述符。

### readStream.bytesRead
>   6.4.0+

读取的字节数。

### readStream.path
>   0.1.93+

`fs.createReadStream()`传人的第一个参数。
如果传人的是字符串类型的`path`，`readStream.path`返回字符串。如果传人的是`Buffer`，那么`reaStream.path`将返回`Buffer`。

## Class:fs.Stats
>   v0.1.21+

返回`fs.stat()`,`fs.lstat()`,`fs.fstat()`和同行对应的类型对象。

*   `stats.isFile()`
*   `stats.isDirecttory()`
*   `stats.isBlockDevice()`
*   `stats.isCharacterDevice()`
*   `stats.isSymbolicLink()`（仅对`fs.lstat()`有效）
*   `stats.isFIFO()`
*   `stats.isSocket()`

相对于一个常规文件 [util.inspect(stats)][util-inspect]返回一个非常相似的字符串。

```js
  {
    dev: 2114,
    ino: 48064969,
    mode: 33188,
    nlink: 1,
    uid: 85,
    gid: 100,
    rdev: 0,
    size: 527,
    blksize: 4096,
    blocks: 8,
    atime: Mon, 10 Oct 2011 23:24:11 GMT,
    mtime: Mon, 10 Oct 2011 23:24:11 GMT,
    ctime: Mon, 10 Oct 2011 23:24:11 GMT,
    birthtime: Mon, 10 Oct 2011 23:24:11 GMT
  }
```

请注意`atime`,`mtime`,`birthtime`和`ctime`都是[Date][Date]对象的实例并且比较它们的值必须使用合适的方法。
通常大多数是使用`getTime()`方法得到一个`1970-01-01 00:00:00`到当前时间的毫秒数。这个整数可以满足任何比较。
然而还有其他方法可以用来显示模糊信息.更多详情可以查看 [MDN JavaScript Reference][MDN-JavaScript-Reference]。

### Stat Time Values
stat对象的时间参数是遵循语义化的。

*   `atime` "Access Time" - 文件数据最后被访问的时间。通过系统调用`mknod(2)`,`utimes(2)`,和`read(2)`进行改变。
*   `mtime` "Modified Time" - 文件数据最后被修改的时间。通过系统调用`mknod(2)`,`utimes(2)`和`write(2)`进行改变。
*   `ctime` "Change Time" - 文件状态最后一次修改（索引数据修改）。通过系统调用`chmod(2)`,`chown(2)`,`link(2)`,`mknod(2)`,
`rename(2)`,`unlink(2)`,`utimes(2)`,`read(2)`和`write(2)`进行改变。
*   `birthtime` "Birth Time" -  文件创建时间。文件创建的时候设置。当文件存在`ctime`或者`birthtime === 1970-01-01T00:00:00`(例如，unix时间搓等于`0`),`birthtime`在文件系统上是不可用的。注意这种情况下`birthtime`可能大于`atime`或`mtime`。在`Darwin`操作系统和其它`FreeBSD`操作系统变体中，如果`atime`的时间早于当前`birthtime`的时间，那么系统将调用`utimes(2)`改变`atime`。

Node v0.12版本之前，在windows系统上`ctime`和`birthtime`都表示创建时间。注意从Node v0.12之后，`ctime`不再是创建时间，Unix系统从来没有这种情况。

## Class: fs.WriteStream
>   v0.1.93+

`WriteStream`是一个[可写流][stream-writable]。

### Event:'open'
>   v0.1.93+

*   `fd` <Integer> 可写流用于文件写入的整型文件描述符。

当可写流文件打开的时候触发。

### Event:'close'
>   v0.1.93+

使用`fs.close()`方法触发`WriteStream`关闭底层文件描述符时触发。

### writeStream.bytesWritten
>   v0.4.7+

每次调用写入的字节数。不包含正在排队等待写入的数据。

### writeStream.path
>   v0.1.93+

`fs.createWriteStream()`传人的第一个参数。
如果传人的是字符串类型的`path`，`writeStream.path`返回字符串。如果传人的是`Buffer`，那么`reaStream.path`将返回`Buffer`。

## fs.access(path[,mode],callback)
> v0.11.15+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `mode` <Integer>
*   `callback` [\<Function\>][Function]

根据`path`检查用户有没有读取该文件或目录的权限。`mode`参数是一个可选的整型,指定要执行的可访问性检查。下面定义的常量都可能是`mode`的值。可以用一些这些值可以创建2个或2个以上的值。

*   `fs.constants.F_OK` - `path` 是可访问的调用过程。对决定文件是否存在有很大的帮助，但不会说明`rwx`权限，如果没有提供`mode`将使用默认值。
*   `fs.constants.R_OK` - `path` 是可读的调用过程。
*   `fs.constants.W_OK` - `path` 是可写的调用过程。
*   `fs.constants.X_ok` - `path` 是可执行的调用过程。Windows系统无效（行为类似于`fs.constants.F_OK`）。

最后一个和参数是`callback`回调函数，函数传人`error`参数。当鉴权失败，这个`error`参数会将错误抛出.下面的例子展示`etc/passwd`文件是否可读或或写在当前进程。

```js
  fs.access('/etc/passwd', fs.constants.R_OK | fs.constants.W_OK, (err) => {
    console.log(err ? 'no access!' : 'can read/write');
  });
```

在调用`fs.open()`,`fs.readFile()`或`fs.writeFile()`之前用`fs.access()`鉴权是不被推荐的。这样做了一个竞态条件,因为其他进程可能会改变文件的状态在同时存在两个调用时。最合适的是，用户应该直接打开/读取/写入文件或在文件不存在的时候抛出错误。

例如：

### write (NOT RECOMMENDED)

```js
  fs.access('myfile', (err) => {
    if (!err) {
      console.error('myfile already exists');
      return;
    }

    fs.open('myfile', 'wx', (err, fd) => {
      if (err) throw err;
      writeMyData(fd);
    });
  });
```
### write (RECOMMENDED)

```js
  fs.open('myfile', 'wx', (err, fd) => {
    if (err) {
      if (err.code === "EEXIST") {
        console.error('myfile already exists');
        return;
      } else {
        throw err;
      }
    }

    writeMyData(fd);
  });
```

### read (NOT RECOMMENDED)

```js
  fs.access('myfile', (err) => {
    if (err) {
      if (err.code === "ENOENT") {
        console.error('myfile does not exist');
        return;
      } else {
        throw err;
      }
    }

    fs.open('myfile', 'r', (err, fd) => {
      if (err) throw err;
      readMyData(fd);
    });
  });
```

### read (RECOMMENDED)

```js
  fs.open('myfile', 'r', (err, fd) => {
    if (err) {
      if (err.code === "ENOENT") {
        console.error('myfile does not exist');
        return;
      } else {
        throw err;
      }
    }

    readMyData(fd);
  });
```

上面的“不推荐”示例检查可访问性,然后使用文件;“推荐”的例子是更好的,因为鉴权成功他们直接使用文件或者处理错误。

通常，鉴权仅发生在一个文件不会被马上使用的情况。例如，可访问性是一个信号从另一个进程传过来的。


==================================未完待续...====================================

[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type
[Buffer]: ./Buffer.md#buffer
[Function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
[Error]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
[util-inspect]: https://nodejs.org/dist/latest-v6.x/docs/api/util.html#util_util_inspect_object_options
[Date]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[MDN-JavaScript-Reference]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[stream-readable]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_readable
[stream-writable]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_writable