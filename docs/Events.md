# 内容目录

*   [Events](#events)
    *   [Passing arguments and this to listeners]()
    *   [Asynchronous vs. Synchronous]()
    *   [Handling events only once]()
    *   [Error events]()
    *   [Class: EventEmitter]()
        *   [Event: 'newListener']()

# Events
>   Stability：稳定

许多的Node.js核心API是建立在围绕一个惯用异步事件驱动的体系结构。触发器会定期触发监听函数。

例如：`net.Server`对象会在每次连接是触发事件；`fs.ReadStream` 会在文件打开后触发事件;流数据可用时会触发事件。

全部的触发器事件都是`EventEmitter`类的实例。这个类暴露`eventEmitter.on()`函数来允许一个或多个函数来对对象进行监听事件触发。
通常情况下,事件名称为“骆峰式”字符串,但可以使用任何有效的JavaScript字符串。

当`EventEmitter`对象触发了一个事件，所有监听这个事件的函数都会同步被触发。所有监听函数的返回值都会被忽略和丢弃。

下面的例子展示了一个简单的`EventEmitter`实例和一个事件监听函数。使用`eventEmitter.on()`方法来注册监听函数，使用`eventEmitter.emit()`函数来触发事件。

```js
  const EventEmitter = require('events');

  class MyEmitter extends EventEmitter {}

  const myEmitter = new MyEmitter();
  myEmitter.on('event', () => {
    console.log('an event occurred!');
  });
  myEmitter.emit('event');
```

## Passing arguments and this to listeners
`eventEmitter.emit()`方法允许设置任意参数来传递给事件监听函数。
最重要的一点一定要记住当一个普通的监听函数被`EventEmitter`触发是，`this`关键字的引用对象是当前`EventEmitter`事件监听对象。

```js
  const myEmitter = new MyEmitter();
  myEmitter.on('event', function(a, b) {
    console.log(a, b, this);
      // Prints:
      //   a b MyEmitter {
      //     domain: null,
      //     _events: { event: [Function] },
      //     _eventsCount: 1,
      //     _maxListeners: undefined }
  });
  myEmitter.emit('event', 'a', 'b');
```

使用ES6箭头函数作为事件监听函数时,`this`关键字将不再引用`EventEmitter`实例，而是当前函数本身:

```js
  const myEmitter = new MyEmitter();
  myEmitter.on('event', (a, b) => {
    console.log(a, b, this);
      // Prints: a b {}
  });
  myEmitter.emit('event', 'a', 'b');
```

## Asynchronous vs. Synchronous
`EventListener`会根据监听事件函数的注册顺序同步触发。这在保证事件的顺序触发和避免竞争状态或逻辑错误上有非常重要的作用。
适当的时候，使用`setImmediate()`或`process.nextTick()`可以把监听函数切换到异步操作模式。

```js
  const myEmitter = new MyEmitter();
  myEmitter.on('event', (a, b) => {
    setImmediate(() => {
      console.log('this happens asynchronously');
    });
  });
  myEmitter.emit('event', 'a', 'b');
```

## Handling events only once
当使用`eventEmitter.on()`方法注册监听器，这个监听器不管什么时候多会在事件触发时被调用。

```js
  const myEmitter = new MyEmitter();
  var m = 0;
  myEmitter.on('event', () => {
    console.log(++m);
  });
  myEmitter.emit('event');
    // Prints: 1
  myEmitter.emit('event');
    // Prints: 2
```

使用`eventEmitter.once()`方法，可以注册一个事件监听器,为一个特定的事件最多触发执行一次。
一旦触发事件，执行过后，这个监听器会被注销掉。

```js
  const myEmitter = new MyEmitter();
  var m = 0;
  myEmitter.once('event', () => {
    console.log(++m);
  });
  myEmitter.emit('event');
    // Prints: 1
  myEmitter.emit('event');
    // Ignored
```

## Error events
当`EventEmitter`实例内部抛出一个错误时，一个典型的`'error'`监听事件动作会被触发。在Node.js里面这种情况会当作特殊情况处理。

如果一个`EventEmitter`没有注册监听器来监听`'error'`事件，一旦`'error'`事件触发，将抛出错误，打印堆栈信息，Node.js进程退出。

```js
  const myEmitter = new MyEmitter();
  myEmitter.emit('error', new Error('whoops!'));
    // Throws and crashes Node.js
```

为了防止Node.js进程崩溃，可以在`process`对象的`uncaughException`事件或使用`domain`模块进行注册监听器。（注意，`domain`模块已经被弃用）

```js
  const myEmitter = new MyEmitter();

  process.on('uncaughtException', (err) => {
    console.log('whoops! there was an error');
  });

  myEmitter.emit('error', new Error('whoops!'));
    // Prints: whoops! there was an error
```

最佳实践，总是在监听列表里添加`'error'`监听器。

```js
  const myEmitter = new MyEmitter();
  myEmitter.on('error', (err) => {
    console.log('whoops! there was an error');
  });
  myEmitter.emit('error', new Error('whoops!'));
    // Prints: whoops! there was an error
```

## Class: EventEmitter
>   v0.1.26+

`EventEmitter`类由`events`模块定义和提供:

```js
  const EventEmitter = require('events');
```

新监听器添加时会触发`EventEmitter`的`'newListener'`事件，移除监听器会触发`'removeListener'`事件。

### Event: 'newListener'
>   v0.1.26+

*   `eventName` [\<String\>][String] | [\<Symbol\>][Symbol] 需要监听的事件名称
*   `listener` [\<Function\>][Function] 函数句柄




[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type
[Symbol]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Symbol_type