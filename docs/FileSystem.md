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
    *   [fs.access(path[, mode], callback)]()
    *   [fs.accessSync(path[, mode])]()
    *   [fs.appendFile(file, data[, options], callback)]()
    *   [fs.appendFileSync(file, data[, options])]()
    *   [fs.chmod(path, mode, callback)]()
    *   [fs.chmodSync(path, mode)]()
    *   [fs.chown(path, uid, gid, callback)]()
    *   [fs.chownSync(path, uid, gid)]()
    *   [fs.close(fd, callback)]()
    *   [fs.closeSync(fd)]()
    *   [fs.constants]()
    *   [fs.createReadStream(path[, options])]()
    *   [fs.createWriteStream(path[, options])]()
    *   [fs.exists(path, callback)]()
    *   [fs.existsSync(path)]()
    *   [fs.fchmod(fd, mode, callback)]()
    *   [fs.fchmodSync(fd, mode)]()
    *   [fs.fchown(fd, uid, gid, callback)]()
    *   [fs.fchownSync(fd, uid, gid)]()
    *   [fs.fdatasync(fd, callback)]()
    *   [fs.fdatasyncSync(fd)]()
    *   [fs.fstat(fd, callback)]()
    *   [fs.fstatSync(fd)]()
    *   [fs.fsync(fd, callback)]()
    *   [fs.fsyncSync(fd)]()
    *   [fs.ftruncate(fd, len, callback)]()
    *   [fs.ftruncateSync(fd, len)]()
    *   [fs.futimes(fd, atime, mtime, callback)]()
    *   [fs.futimesSync(fd, atime, mtime)]()
    *   [fs.lchmod(path, mode, callback)]()
    *   [fs.lchmodSync(path, mode)]()
    *   [fs.lchown(path, uid, gid, callback)]()
    *   [fs.lchownSync(path, uid, gid)]()
    *   [fs.link(srcpath, dstpath, callback)]()
    *   [fs.linkSync(srcpath, dstpath)]()
    *   [fs.lstat(path, callback)]()
    *   [fs.lstatSync(path)]()
    *   [fs.mkdir(path[, mode], callback)]()
    *   [fs.mkdirSync(path[, mode])]()
    *   [fs.mkdtemp(prefix[, options], callback)]()
    *   [fs.mkdtempSync(prefix[, options])]()
    *   [fs.open(path, flags[, mode], callback)]()
    *   [fs.openSync(path, flags[, mode])]()
    *   [fs.read(fd, buffer, offset, length, position, callback)]()    
    *   [fs.readSync(fd, buffer, offset, length, position)]()
    *   [fs.readdir(path[, options], callback)]()
    *   [fs.readdirSync(path[, options])]()
    *   [fs.readFile(file[, options], callback)]()
    *   [fs.readFileSync(file[, options])]()
    *   [fs.readlink(path[, options], callback)]()
    *   [fs.readlinkSync(path[, options])]()
    *   [fs.realpath(path[, options], callback)]()
    *   [fs.realpathSync(path[, options])]()
    *   [fs.rename(oldPath, newPath, callback)]()
    *   [fs.renameSync(oldPath, newPath)]()
    *   [fs.rmdir(path, callback)]()
    *   [fs.rmdirSync(path)]()
    *   [fs.stat(path, callback)]()
    *   [fs.statSync(path)]()
    *   [fs.symlink(target, path[, type], callback)]()

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

*   `fd`\<Integer\> `ReadStream`中使用的文件描述符。

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

*   `fd` \<Integer\> 可写流用于文件写入的整型文件描述符。

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
*   `mode` \<Integer\>
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

## fs.accessSync(path[,mode])
>   v0.11.15+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `mode` <  Integer>

`fs.access()`的同步版本。鉴权失败抛出错误，反之什么都不做。

## fs.appendFile(file, data[, options], callback)
>   v0.6.7+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer] | [\<Number\>][Number]  文件名或者文件描述符
*   `data` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<Object\>][Object] | [\<String\>][String]
    *   `encoding` [\<String\>][String] | [\<Null\>][Null] 默认= `'utf-8'`
    *   `mode` \<Integer\> 默认= `0o666`
    *   `flag` [\<String\>][String] 默认= `'a'`
*   `callback` [\<Function\>][Function]

异步追加数据到文件，如果文件不存在则创建文件。`data`可以是字符串或`Buffer`。

例如：

```js
  fs.appendFile('message.txt', 'data to append', (err) => {
    if (err) throw err;
    console.log('The "data to append" was appended to file!');
  });
```

如果传入`options`参数为字符串,则认为是字符串编码。例如：

```js
  fs.appendFile('message.txt', 'data to append', 'utf8', callback);
```

任何特定的文件描述符都会在追加的时候都已经被打开。

注意：如果指定一个文件描述符,那么它就不会自动关闭。


## fs.appendFileSync(file , data[,options])
>   v0.6.7+

*   `file`  [\<String\>][String] | [\<Buffer\>][Buffer] | [\<Number\>][Number]  文件名或者文件描述符
*   `options` [\<Object\>][Object] | [\<String\>][String]
    *   `encoding` [\<String\>][String] | [\<Null\>][Null] 默认= `'utf-8'`
    *   `mode` \<Integer\> 默认= `0o666`
    *   `flag` [\<String\>][String] 默认= `'a'`

`fs.appendFile()`函数的同步版本。返回`undefined`。

## fs.chmod(path,mode,callback)
>   v0.1.30+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `mode`  \<Integer\>
*   `callback`  [\<Function\>][Function]

异步的[chmod(2)][chmod(2)],执行失败callback的参数error会抛出，否则error为空。

## fs.chmodSync(path,mode)
>   v0.6.7+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `mode`  \<Integer\>

同步的[chmod(2)][chmod(2)],返回`undefined`。

## fs.chown(path,uid,gid,callback)
>   v0.1.97+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `uid`  \<Integer\>
*   `gid`  \<Integer\>
*   `callback`  [\<Function\>][Function]

异步的[chown(2)][chown(2)],执行失败callback的参数error会抛出，否则error为空。

## fs.chownSync(path,uid,gid)
>   v0.1.97+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `uid`  \<Integer\>
*   `gid`  \<Integer\>

同步的[chown(2)][chown(2)],返回`undefined`。

## fs.close(fd,callback)
>   v0.0.2+

*   `fd` \<Integer\>
*   `callback`  [\<Function\>][Function]

异步的[close(2)][close(2)]。执行失败callback的参数error会抛出，否则error为空。

## fs.closeSync(fd)
>   v0.0.21+

*   `fd` \<Integer\>

同步的[close(2)][close(2)],返回`undefined`。

## fs.constants

返回一个包含文件系统操作使用的常量的对象。特定常量描述在[FS Constants](#fscontents)。

## fs.createReadStream(path[, options])
>   v0.1.31+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<Object\>][Object] | [\<String\>][String]
    *   `flags` [\<String\>][String]
    *   `encoding` [\<String\>][String]
    *   `fd` \<Integer\>
    *   `mode` \<Integer\>
    *   `autoClose` [\<Boolean\>][Boolean]
    *   `start` \<Integer\>
    *   `end`   \<Integer\>

返回新的[ReadStream](#classfsreadstream)对象。（[Readable Stream][Readable-Stream]）。

注意，可读流的`heighWaterMark`默认值是`16kb`，`fs.createReadStream()`方法返回的流的参数`heighWaterMark`默认值为`64kb`。

`options`可以是对象或字符串，默认值如下：

```js
  {
    flags: 'r',
    encoding: null,
    fd: null,
    mode: 0o666,
    autoClose: true
  }
```

`options`对象的参数`start`和`end`参数用于限制读取文件的字节范围。`start`和`end`位置都包含在内并且`start`从0开始。任何一个可接受`Buffer`缓冲区的字符编码。


如果给定`fd`，那么`ReadStream`将忽略`path`参数而是使用指定的文件描述符。这意味着`open`事件将不会被触发。注意`fd`应该是阻塞的；非阻塞的`fd`需要通过调用[net.Socket][net-Socket]。

如果`autoClose`设置为`false`,那么文件描述符`fd`将不会被自动关闭，即使产生错误。你必须确保文件描述符已经关闭没有遗漏。
如果`autoClose`设置为`true`(默认为：`true`)，当产生错误或读取文件完毕都会自动关闭文件描述符。

`mode`用于设置文件的模式，仅已经创建的文件有效。

下面的例子是读取长度为100字节的文件的最后10个字节：

```js
  fs.createReadStream('sample.txt', {start: 90, end: 99});
```

`options`为字符串类型，那么将默认视为编码格式。


## fs.createWriteStream(path[,options])
>   v0.1.31+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<Object\>][Object] | [\<String\>][String]
    *   `flags` [\<String\>][String]
    *   `defaultEncoding` [\<String\>][String]
    *   `fd` \<Integer\>
    *   `mode` \<Integer\>
    *   `autoClose` [\<Boolean\>][Boolean]
    *   `start` \<Integer\>


返回新的[WriteStream](#classfswritestream)对象。（[Writable Stream][Writable-Stream]）。

`options`可以是对象或字符串，默认值如下：

```js
  {
    flags: 'w',
    defaultEncoding: 'utf8',
    fd: null,
    mode: 0o666,
    autoClose: true
  }
```

`options`参数包含一个`start`参数表示允许在文件的位置开始写入数据。宁可使用`flags`模式为`r+`修改文件也不用模式`w`。
`defaultEncoding`: 任何一个可接受`Buffer`缓冲区的字符编码。

如果`autoClose`设置为`false`,那么文件描述符`fd`将不会被自动关闭，即使产生错误。你必须确保文件描述符已经关闭没有遗漏。
如果`autoClose`设置为`true`(默认为：`true`)，当产生错误或读取文件完毕都会自动关闭文件描述符。

类似于`ReadStream` ，如果给定`fd`，将忽略`path`参数而是使用指定的文件描述符。这意味着`open`事件将不会被触发。注意`fd`应该是阻塞的；非阻塞的`fd`需要通过调用[net.Socket][net-Socket]。

`options`为字符串类型，那么将默认视为编码格式。

## fs.exists(path, callback)
>  已弃用

>  v0.0.2 - v1.0.0

>  用 `fs.stat()`或`fs.access()`代替。

## fs.existsSync(path)
>  已弃用

>  v0.1.21 - v1.0.0

>  用 `fs.statSync()`或`fs.accessSync()`代替。

## fs.fchmod(fd,mode,callback)
>   v0.4.7+

*   `fd` \<Integer\>
*   `mode` \<Integer\>
*   `callback` [\<Function\>][Function]

异步的[fchmod(2)][fchmod(2)] 。执行失败callback的参数error会抛出，否则error为空。

## fs.fchmodSync(fd,mode)
>   v0.4.7+

*   `fd` \<Integer\>
*   `mode` \<Integer\>

同步的[fchmod(2)][fchmod(2)] 。返回`undefined`。


## fs.fchown(fd,uid,gid,callback)
>   v0.4.7+

*   `fd`  \<Integer\>
*   `uid`  \<Integer\>
*   `gid`  \<Integer\>
*   `callback`  [\<Function\>][Function]

异步的[fchown(2)][fchown(2)] 。执行失败callback的参数error会抛出，否则error为空。

## fs.fchownSync(fd,uid,gid)
>   v0.4.7+

*   `fd`  \<Integer\>
*   `uid`  \<Integer\>
*   `gid`  \<Integer\>

同步的[fchown(2)][fchown(2)] 。返回`undefined`。

## fs.fdatasync(fd , callback)
>   v0.1.96+

*   `fd` \<Integer\>
*   `callback` [\<Function\>][Function]

异步的[fdatasync(2)][fdatasync(2)] 。执行失败callback的参数error会抛出，否则error为空。

## fs.fdatasyncSync(fd)
>   v0.1.96+

*   `fd` \<Integer\>

同步的[fdatasync(2)][fdatasync(2)] 。返回`undefined`。

## fs.fstat(fd , callback)
>   v0.1.95+

*   `fd` \<Integer\>
*   `callback`  [\<Function\>][Function]

异步的[fstat(2)][fstat(2)] 。执行失败callback的参数error会抛出，否则error为空。

## fs.fstatSync(fd)
>   v0.1.95+

*   `fd` \<Integer\>

同步的[fstat(2)][fstat(2)] 。 返回`undefined`。

## fs.fsync(fd,callback)
>   v0.1.96+

*   `fd` \<Integer\>
*   `callback`  [\<Function\>][Function]

异步的[fsync(2)][fsync(2)] 。执行失败callback的参数error会抛出，否则error为空。

## fs.fsyncSync(fd)
>   v0.1.96+

*   `fd` \<Integer\>

同步的[fsync(2)][fsync(2)] 。 返回`undefined`。


## fs.ftruncate(fd, len, callback)
>   v0.8.6+

*   `fd` \<Integer\>
*   `len` \<Integer\> 默认：0
*   `callback` [\<Function\>][Function]

异步的[ftruncate(2)][ftruncate(2)] 。执行失败callback的参数error会抛出，否则error为空。

如果引用的文件描述符的文件的大小大于`len`。那么仅只有最开始长度为`len`的字节会被保留。

例如，下面的程序只保留第一个文件的四个字节

```js
  console.log(fs.readFileSync('temp.txt', 'utf8'));
    // prints Node.js

  // get the file descriptor of the file to be truncated
  const fd = fs.openSync('temp.txt', 'r+');

  // truncate the file to first four bytes
  fs.ftruncate(fd, 4, (err) => {
    assert.ifError(err);
    console.log(fs.readFileSync('temp.txt', 'utf8'));
  });
    // prints Node
```

如果文件的内容小于`len`字节。那么多余空出的部分将使用`\\0`填充。例如，

```js
  console.log(fs.readFileSync('temp.txt', 'utf-8'));
    // prints Node.js

  // get the file descriptor of the file to be truncated
  const fd = fs.openSync('temp.txt', 'r+');

  // truncate the file to 10 bytes, whereas the actual size is 7 bytes
  fs.ftruncate(fd, 10, (err) => {
    assert.ifError(!err);
    console.log(fs.readFileSync('temp.txt'));
  });
    // prints <Buffer 4e 6f 64 65 2e 6a 73 00 00 00>
    // ('Node.js\0\0\0' in UTF8)
```

最后3个字节将使用`\\0`填充，用来填补截断部分。

## fs.ftruncateSync(fd, len)
>   v0.8.6+

*   `fd` \<Integer\>
*   `len`  \<Integer\> 默认：0

同步的[ftruncate(2)][ftruncate(2)] 。返回`undefined`。

## fs.futimes(fd, atime, mtime, callback)
>   v0.4.2+

*   `fd`  \<Integer\>
*   `atime`  \<Integer\>
*   `mtime`   \<Integer\>
*   `callback`  [\<Function\>][Function]

根据给定的文件描述符引用修改文件的时间戳。

## fs.futimesSync(fd,atime,mtime)
>   v0.4.2+

*   `fd`  \<Integer\>
*   `atime`  \<Integer\>
*   `mtime`   \<Integer\>

`fs.futimes()`的同步版本。返回`undefined`。

## fs.lchmod(path,mode,callback)
>  弃用 v0.4.7+

## fs.lchmodSync(path,mode)
>  弃用 v0.4.7+

## fs.lchown(path,mode,callback)
>  弃用 v0.4.7+

## fs.lchownSync(path,mode)
>  弃用 v0.4.7+

## fs.link(srcpath , dstpath , callpack)
>   v0.1.31+

*   `srcpath`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `dstpath`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `callback` [\<Function\>][Function]

异步的[link(2)][link(2)]。执行失败callback的参数error会抛出，否则error为空。

## fs.linkSync(srcpath , dstpath)
>   v0.1.30+

*   `srcpath`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `dstpath`  [\<String\>][String] | [\<Buffer\>][Buffer]

同步的[link(2)][link(2)]。返回：`undfined`。

## fs.lstat(path,callback)
>   0.1.30+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `callback` [\<Function\>][Function]

异步的[lstat(2)][lstat(2)]。回调函数传入2个参数`(err,stats)`。`stats`表示`fs.Stats`对象。`lstat()`等同于`stat()`，如果`path`是个符号链接，那么这个状态就是当前这个符号链接的状态，而不是链接指向的文件的状态。

## fs.lstatSync(path)
>   0.1.30+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]

同步的[lstat(2)][lstat(2)]。返回：`fs.Stats`实例。

## fs.mkdir(path[,mode],callback)
>   v0.1.8+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `mode` \<Integer\>
*   `callback` [\<Function\>][Function]

异步的[mkdir(2)][mkdir(2)]。执行失败callback的参数error会抛出，否则error为空。

## fs.mkdirSync(path[,mode])
>   v0.1.21+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `mode` \<Integer\>

同步的[mkdir(2)][mkdir(2)]。返回：`undfined`。

## fs.mkdtemp(prefix[,options],callback)
>   v5.10.0+

*   `prefix` [\<String\>][String]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`
*   `callback` [\<Function\>][Function]

创建一个唯一的临时目录。

生成6位随机字符附加在创建的临时目录的名称`prefix`前缀后面。

创建的文件夹路径会通过callback的第二个参数传入callback。

`options`参数不管是字符串还是一个存在`encoding`属性的对象都会被视为传入字符编码。

例如：

```js
  fs.mkdtemp('/tmp/foo-', (err, folder) => {
    if (err) throw err;
    console.log(folder);
      // Prints: /tmp/foo-itXde2
  });
```

注意：`fs.mkdtemp()`方法将随机产生的6个字符直接附加到字符串的前缀后面。例如，给定一个`/tmp`目录，如果目的是在这个`/tmp`目录里面创建一个临时的目录，那么`prefix`前缀后面必须以一个特定平台的分隔符作为结束标识(`require('path').sep`)。

```js
  // The parent directory for the new temporary directory
  const tmpDir = '/tmp';

  // This method is *INCORRECT*:
  fs.mkdtemp(tmpDir, (err, folder) => {
    if (err) throw err;
    console.log(folder);
      // Will print something similar to `/tmpabc123`.
      // Note that a new temporary directory is created
      // at the file system root rather than *within*
      // the /tmp directory.
  });

  // This method is *CORRECT*:
  const path = require('path');
  fs.mkdtemp(tmpDir + path.sep, (err, folder) => {
    if (err) throw err;
    console.log(folder);
      // Will print something similar to `/tmp/abc123`.
      // A new temporary directory is created within
      // the /tmp directory.
  });
```

## fs.mkdtempSync(prefix[,options])
>   v5.10.0+

*   `prefix` [\<String\>][String]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`

同步版的[fs.mkdtemp()] 。 返回创建的文件目录。

`options`参数不管是字符串还是一个存在`encoding`属性的对象都会被视为传入字符编码。

## fs.open(path,flags[,mode] , callback)
>   v0.0.2+

*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `flags` [\<String\>][String] | [\<Number\>][Number]
*   `mode` \<Integer\>
*   `callback` [\<Function\>][Function]

异步打开一个文件。可以查看[Open(2)][Open(2)]。`flags`可能的值：

*   `r` - 以读的方式打开文件。文件不存在时抛出异常。
*   `r+` - 以读和写的方式打开文件。文件不存在时抛出异常。
*   `rs+` - 在同步模式下，以读和写的方式打开文件。命令操作系统忽视本地文件系统缓存。这主要是用于NFS挂载上打开文件,因为它允许您跳过可能不新鲜的本地缓存。它是一个对I / O性能有非常严重的影响所以不要使用这个标志,除非你需要它。这里并没有把`fs.open()`改为同步阻塞模式调用。相反`fs.openSync()`才是以同步阻塞的模式执行。
*   `w` - 以写的方式打开文件。如果文件不存在则创建或者如果文件存在则截取。
*   `wx` - 与`w`模式类似，但是文件存在时会报错。
*   `w+` - 。如果文件不存在则创建或者如果文件存在则截取。
*   `wx+` - 与`w+`模式类似，但是文件存在时会报错。
*   `a` - 以追加的方式打开文件。如果文件不存在则创建。
*   `ax` - 与`a`模式类似，但是文件存在时会报错。
*   `a+` - 以读和追加的方式打开文件。如果文件不存在则创建。
*   `ax+` - 与`a+`模式类似，但是文件存在时会报错。

如果文件存在通过`mode`设置文件的模式。默认`0666`，可读和可写。

callback函数参数`(err,fd)`。

参数`flags`为`'x'`(`open(2)`的`O_EXCL`)确保路径是新创建的。在POSIX系统中,路径被认为存在,即使它是一个符号链接到一个不存在的文件。使用网络文件系统时这个标志可能会或可能不会有效。

`flags`参数在[open(2)][Open(2)]函数文档中有对应的数字表示。通常使用`fs.contants`里面的有效值。在Windows系统，`flags`有对应的等价表示值来表示。例如。`O_WRONLY` 到`FILE_GENERITE`,或者`O_EXCL | O_CREAT`到`CREATE_NEW`。

在Linux中，以追加模式打开文件时指定位置写入是无效的。内核会忽略定位参数总是直接把数据追加到文件的末尾。

注意：`fs.open()`的某些`flags`的行为在不同平台表现出不同的特性。根据下面的例子，同样的在`OS X`和`Linux`以`a+`模式打开一个目录，都会报错。在`Windows`和`FreeBSD`,文件描述符会成功返回。

```js
  // OS X and Linux
  fs.open('<directory>', 'a+', (err, fd) => {
    // => [Error: EISDIR: illegal operation on a directory, open <directory>]
  });

  // Windows and FreeBSD
  fs.open('<directory>', 'a+', (err, fd) => {
    // => null, <fd>
  });
```

## fs.openSync(path, flags[, mode])
>   v0.1.21+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `flags` [\<String\>][String] | [\<Number\>][Number]
*   `mode` \<Integer\>

同步版的`fs.open()`。返回文件描述符`fd`。

## fs.read(fd, buffer, offset, length, position, callback)
>   v0.0.2+

*   `fd` \<Integer\>
*   `buffer` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `offset` \<Integer\>
*   `length` \<Integer\>
*   `position` \<Integer\>
*   `callback` [\<Function\>][Function]

根据`fd`指定的文件描述符读取文件数据。

`buffer`是需要被写入的数据。

`offset`是从`buffer`缓冲区的哪个位置开始开始写入数据。

`length`是需要写入文件的字节长度。

`postion` 是一个整型值，指读取的文件起始位置，如果position为null，将会从文件当前的位置读取数据。

`callback` 包含3个参数 `(err , bytesRead , buffer)`。

## fs.readSync(fd, buffer, offset, length, position)
>   v0.1.21+

*   `fd` \<Integer\>
*   `buffer` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `offset` \<Integer\>
*   `length` \<Integer\>
*   `position` \<Integer\>

同步的`fs.read()`。返回读取的字节长度`bytesRead`。

## fs.readdir(path[, options], callback)
>   v0.1.8+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`
*   `callback` [\<Function\>][Function]

异步的[readdir(3)][readdir(3)]读取目录信息。`callback`包含2个参数`(err,files)`。`files`是一个包含这个目录文件名称的数组，`.`和`..`除外。

`options`参数如果是字符串或者是一个包含`encoding`属性的对象都视为是设置`callback`回调函数里面`files`数组中文件名的字符编码。如果`encoding`设为`buffer`，返回的`files`数组中文件名都是`Buffer`对象。

fs.readdirSync(path[, options])
>   v0.1.29+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`

同步的[readdir(3)][readdir(3)]读取目录信息。返回个包含这个目录文件名称的数组，`.`和`..`除外。

`options`参数如果是字符串或者是一个包含`encoding`属性的对象都视为是设置`callback`回调函数里面`files`数组中文件名的字符编码。如果`encoding`设为`buffer`，返回的`files`数组中文件名都是`Buffer`对象。

## fs.readFile(file[, options], callback)
>   v0.1.29+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer] | \<Integer\> 文件或文件描述符
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] | [\<Null\>][Null] 默认：`null`
    *   `flag` [\<String\>][String] 默认： `'r'`
*   `callback` [\<Function\>][Function]

异步的读取文件内容，例如：

```js
  fs.readFile('/etc/passwd', (err, data) => {
    if (err) throw err;
    console.log(data);
  });
``` 

`callback`有2个参数`(err,data)`，`data`为文件内容。

没有指定编码，将返回原始的`buffer`数据。

`options`为字符串，视为指定的编码格式。例如：

```js
  fs.readFile('/etc/passwd', 'utf8', callback);
```

任何指定文件描述符都必须是可读的。

注意：根据指定的文件描述符读文件，文件不会在读取完成或发生错误后自动关闭。

## fs.readFileSync(file[, options])
>   v0.1.8+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer] | \<Integer\> 文件或文件描述符
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] | [\<Null\>][Null] 默认：`null`
    *   `flag` [\<String\>][String] 默认： `'r'`

同步的`fs.readFile`，返回文件内容。

没有指定编码，将返回原始的`buffer`数据。

## fs.readlink(path[, options], callback)
>   v0.1.31+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`
*   `callback` [\<Function\>][Function]

异步的[readlink(2)][readlink(2)]，`callback`包含2个参数`(err,linkString)`。

`options`参数如果是字符串或者是一个包含`encoding`属性的对象都视为是设置链接路径的字符编码。如果`encoding`设为`buffer`，返回的链接路径是`Buffer`对象。

## fs.readlinkSync(path[, options])
>   v0.1.31+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`

异步的[readlink(2)][readlink(2)]，返回符号链接的字符串。

`options`参数如果是字符串或者是一个包含`encoding`属性的对象都视为是设置链接路径的字符编码。如果`encoding`设为`buffer`，返回的链接路径是`Buffer`对象。

## fs.realpath(path[, options], callback)
>   v0.1.31+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`
*   `callback` [\<Function\>][Function]

异步的[realpath(3)][realpath(3)]。`callback`包含2个参数`(err,resolvedPath)`。路径相对于`process.cwd`。

只支持`'utf-8'`字符串的路径。

`options`参数如果是字符串或者是一个包含`encoding`属性的对象都视为是设置路径的字符编码。如果`encoding`设为`buffer`，返回的路径是`Buffer`对象。

## fs.realpathSync(path[, options])
>   v0.1.31+

*   `file` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `options` [\<String\>][String] | [\<Object\>][Object]
    *   `endcoding` [\<String\>][String] 默认：`'utf-8'`

同步的[realpath(3)][realpath(3)]。返回路径。

只支持`'utf-8'`字符串的路径。

`options`参数如果是字符串或者是一个包含`encoding`属性的对象都视为是设置路径的字符编码。如果`encoding`设为`buffer`，返回的路径是`Buffer`对象。

## fs.rename(oldPath, newPath, callback)
>   v0.0.2+

*   `oldPath` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `newPath` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `callback` [\<Function\>][Function]

异步的[rename(2)][rename(2)]。执行失败callback的参数error会抛出，否则error为空。

## fs.renameSync(oldPath, newPath)
>   v0.1.21+

*   `oldPath` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `newPath` [\<String\>][String] | [\<Buffer\>][Buffer]

同步的[rename(2)][rename(2)]。返回`undefined`。

## fs.rmdir(path, callback)
>   v0.0.2+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `callback` [\<Function\>][Function]

异步的[rmdir(2)][rmdir(2)]。执行失败callback的参数error会抛出，否则error为空。

## fs.rmdirSync(path)
>   v0.1.21+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]

同步的[rmdir(2)][rmdir(2)]。返回`undefined`。

## fs.stat(path, callback)
>   v0.0.2+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `callback` [\<Function\>][Function]

异步的[stat(2)][stat(2)]。`callback`包含2个参数(err,stats)。`stats`是一个`fs.Stats`对象。更多信息请查看`fs.Stats`。

## fs.statSync(path)
>   v0.1.21+

*   `path` [\<String\>][String] | [\<Buffer\>][Buffer]

同步的[stat(2)][stat(2)]。返回`fs.Stats`实例。

## fs.symlink(target, path[, type], callback)
>   v0.1.31+

*   `target` [\<String\>][String] | [\<Buffer\>][Buffer]
*   `path`  [\<String\>][String] | [\<Buffer\>][Buffer]
*   `type`  [\<String\>][String]
*   `callback` [\<Function\>][Function]

异步的[symlink(2)][symlink(2)]。执行失败callback的参数error会抛出，否则error为空。

==================================未完待续...====================================

[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type
[Boolean]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type
[Buffer]: ./Buffer.md#buffer
[Object]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
[Null]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Null_type
[Object]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object
[Number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type
[Function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
[Error]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error
[util-inspect]: https://nodejs.org/dist/latest-v6.x/docs/api/util.html#util_util_inspect_object_options
[Date]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[MDN-JavaScript-Reference]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[stream-readable]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_readable
[stream-writable]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_writable
[chmod(2)]: http://man7.org/linux/man-pages/man2/chmod.2.html
[chown(2)]: http://man7.org/linux/man-pages/man2/chown.2.html
[close(2)]: http://man7.org/linux/man-pages/man2/close.2.html
[Readable-Stream]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_readable
[Writeable-Stream]: https://nodejs.org/dist/latest-v6.x/docs/api/stream.html#stream_class_stream_writable
[net-Socket]: https://nodejs.org/dist/latest-v6.x/docs/api/net.html#net_class_net_socket
[fchmod(2)]: http://man7.org/linux/man-pages/man2/fchmod.2.html
[fchown(2)]: http://man7.org/linux/man-pages/man2/fchown.2.html
[fdatasync(2)]: http://man7.org/linux/man-pages/man2/fdatasync.2.html
[fstat(2)]: http://man7.org/linux/man-pages/man2/fstat.2.html
[fsync(2)]: http://man7.org/linux/man-pages/man2/fsync.2.html
[ftruncate(2)]: http://man7.org/linux/man-pages/man2/ftruncate.2.html
[link(2)]: http://man7.org/linux/man-pages/man2/link.2.html
[lstat(2)]: http://man7.org/linux/man-pages/man2/lstat.2.html
[mkdir(2)]: http://man7.org/linux/man-pages/man2/mkdir.2.html
[Open(2)]: http://man7.org/linux/man-pages/man2/open.2.html
[readlink(2)]: http://man7.org/linux/man-pages/man2/readlink.2.html
[realpath(3)]: http://man7.org/linux/man-pages/man3/realpath.3.html
[rename(2)]: http://man7.org/linux/man-pages/man2/rename.2.html
[rmdir(2)]: http://man7.org/linux/man-pages/man2/rmdir.2.html
[stat(2)]: http://man7.org/linux/man-pages/man2/stat.2.html