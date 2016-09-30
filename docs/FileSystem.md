# 内容目录

*   [File System](#file-system)
    *   [Buffer API](#buffer-api)
    *   [Class:fs.FSWatcher](#classfsfswatcher)
        *   [Event: 'change'](#event-change)
        *   [Event: 'error'](#event-error)
        *   [watcher.close()](#watcherclose)
    *   [Class: fs.ReadStream](#class-fsreadstream)

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



==================================未完待续...====================================



[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type
[Error]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error