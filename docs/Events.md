# 内容目录

*   [Events](#events)
    *   [Passing arguments and this to listeners](#passing-arguments-and-this-to-listeners)
    *   [Asynchronous vs. Synchronous](#asynchronous-vs-synchronous)
    *   [Handling events only once](#handling-events-only-once)
    *   [Error events](#error-events)
    *   [Class: EventEmitter](#class-eventemitter)
        *   [Event: 'newListener'](#event-newlistener)
        *   [Event: 'removeListener'](#event-removelistener)
        *   [EventEmitter.defaultMaxListeners](#eventemitterdefaultmaxlisteners)
        *   [emitter.addListener(eventName, listener)](#emitteraddlistenereventname-listener)
        *   [emitter.emit(eventName[, arg1][, arg2][, ...])](#emitteremiteventname-arg1-arg2-)
        *   [emitter.eventNames()](#emittereventnames)
        *   [emitter.getMaxListeners()](#emittergetmaxlisteners)
        *   [emitter.listenerCount(eventName)](#emitterlistenercounteventname)
        *   [emitter.listeners(eventName)](#emitterlistenerseventname)
        *   [emitter.on(eventName, listener)](#emitteroneventname-listener)
        *   [emitter.once(eventName, listener)](#emitteronceeventname-listener)
        *   [emitter.prependListener(eventName, listener)](#emitterprependlistenereventname-listener)
        *   [emitter.prependOnceListener(eventName, listener)](#emitterprependoncelistenereventname-listener)
        *   [emitter.removeAllListeners([eventName])](#emitterremovealllistenerseventname)
        *   [emitter.removeListener(eventName, listener)](#emitterremovelistenereventname-listener)
        *   [emitter.setMaxListeners(n)](#emittersetmaxlistenersn)

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

`EventEmitter`实例会触发自带的`newListener`事件在添加其它监听器到监听器列表之前。

注册`newListener`事件监听器将传递事件名称和添加的事件监听器的引用。

在添加监听器之前触发事件有着微妙但又严重的负面作用：在`newListener`回调函数中注册任何额外的相同名称的监听器都会比在进程中添加的监听器优先添加。

```js
  const myEmitter = new MyEmitter();
  // Only do this once so we don't loop forever
  myEmitter.once('newListener', (event, listener) => {
    if (event === 'event') {
      // Insert a new listener in front
      myEmitter.on('event', () => {
        console.log('B');
      });
    }
  });
  myEmitter.on('event', () => {
    console.log('A');
  });
  myEmitter.emit('event');
    // Prints:
    //   B
    //   A
```

### Event: 'removeListener'
>   v0.9.3+

*   `eventName`  [\<String\>][String] | [\<Symbol\>][Symbol] 需要移除的事件名称
*   `listener` [\<Function\>][Function] 函数句柄

`'removeListener'`事件会在 `listener`移除之后触发。

### EventEmitter.listenerCount(emitter, eventName)
>   Added in: v0.9.12  Deprecated: v4.0.0

>   用 `emitter.klistenerCount()`代替

这个方法返回`emitter`注册的监听器的数量。

```js
  const myEmitter = new MyEmitter();
  myEmitter.on('event', () => {});
  myEmitter.on('event', () => {});
  console.log(EventEmitter.listenerCount(myEmitter, 'event'));
    // Prints: 2
```

### EventEmitter.defaultMaxListeners
>  v0.11.2

默认情况下，任何一个事件最多只能注册 `10` 个监听器。这个限制是可以通过 `EventEmitter`实例的 `emitter.setMaxListeners(n)`方法进行修改。要改变所有的`EventEmitter`实例的默认值，可以使用`EventEmitter.defaultMaxListeners`属性进行设置。

设置`EventEmitter.defaultMaxListeners`要注意。`.defaultMaxListeners`属性的改变会影响所有的`EventEmitter`实例，包括改变`defaultMaxListeners`属性之前创建的实例。然而，调用`emitter.setMaxListeners(n)`还是优先于`EventEmitter.defaultMaxListeners`。

注意这里没有硬性规定。一旦`EventEmitter`添加了超过限制的监听器时将会输出标准的错误流警告信息“EventEmitter内存泄漏”。
对于许多`EventEmitter`，`emitter.getMaxListeners()`和`emitter.setMaxListater`方法可以用来处理这些警告。

```js
  emitter.setMaxListeners(emitter.getMaxListeners() + 1);
  emitter.once('event', () => {
    // do stuff
    emitter.setMaxListeners(Math.max(emitter.getMaxListeners() - 1, 0));
  });
```

`--trace-warnings` 命令行参数可以用来显示跟踪堆栈的警告信息。

通过`process.on(wraning)`检查发出的警告能查出 `emitter`，`type`和`count`属性--即事件`emeitter`实例的引用,名称和监听器的数量。

### emitter.addListener(eventName, listener)
>  v0.1.26+

`emitter.on(eventName, listener)`的别名。

### emitter.emit(eventName[, arg1][, arg2][, ...])
>   v0.1.26

根据它们注册的顺序，`eventName`注册的事件名称来同时触发监听器，并提供监听器所需要的参数。

当前事件监听器存在返回`true`,反之`false`。

### emitter.eventNames()
> v6.0.0+

返回 `emitter`注册的事件监听器监听事件的列表。这个值可以是`Array` ， `String` 或 `Symbol`。

```js
  const EventEmitter = require('events');
  const myEE = new EventEmitter();
  myEE.on('foo', () => {});
  myEE.on('bar', () => {});

  const sym = Symbol('symbol');
  myEE.on(sym, () => {});

  console.log(myEE.eventNames());
    // Prints [ 'foo', 'bar', Symbol(symbol) ]
```

### emitter.getMaxListeners()
>   v1.0.0

返回`EventEmitter`当前能添加监听器的最大值。可以通过 `emitter.setMaxListeners(n)`或`EventEmitter.defaultMaxListeners`修改这个值。

### emitter.listenerCount(eventName)
>   v3.2.0

*   `eventName` [\<String\>][String] | [\<Symbol\>][Symbol] 监听的事件名称

根据`eventName`返回这个事件的事件监听器的数量。

### emitter.listeners(eventName)
>   v0.1.26

根据事件名称`eventName`返回事件监听器的拷贝的数组。

```js
  server.on('connection', (stream) => {
    console.log('someone connected!');
  });
  console.log(util.inspect(server.listeners('connection')));
    // Prints: [ [Function] ]
```

### emitter.on(eventName, listener)
>   v0.1.101+

*   `eventName` [\<String\>][String] | [\<Symbol\>][Symbol] 监听的事件名称
*   `listener` [\<Function\>][Function] 回调函数

根据事件名称`eventName`往监听器数组里面添加监听器函数。不会检查当前事件是否已经添加监听器。给同一个事件名称`eventName`添加多个监听器，那么多个监听器函数都会被调用。

```js
  server.on('connection', (stream) => {
    console.log('someone connected!');
  });
```

返回`EventEmitter`的引用，所以可以使用链式调用。

默认情况下，事件监听器的执行顺序是根据添加顺序来着。 使用`emitter.prependListener()`方法可以添加附带监听器在事件监听数组函数执行之前执行。

```js
  const myEE = new EventEmitter();
  myEE.on('foo', () => console.log('a'));
  myEE.prependListener('foo', () => console.log('b'));
  myEE.emit('foo');
    // Prints:
    //   b
    //   a
```

### emitter.once(eventName, listener)
>   v0.3.0+

*   `eventName` [\<String\>][String] | [\<Symbol\>][Symbol] 监听的事件名称
*   `listener` [\<Function\>][Function] 回调函数

返回`EventEmitter`的引用，所以可以使用链式调用。

默认情况下，事件监听器的执行顺序是根据添加顺序来着。 使用`emitter.prependOnceListener()`方法可以添加附带监听器在事件监听数组函数执行之前执行。

```js
  const myEE = new EventEmitter();
  myEE.once('foo', () => console.log('a'));
  myEE.prependOnceListener('foo', () => console.log('b'));
  myEE.emit('foo');
    // Prints:
    //   b
    //   a
```

### emitter.prependListener(eventName, listener)
>   v6.0.0+

*   `eventName` [\<String\>][String] | [\<Symbol\>][Symbol] 监听的事件名称
*   `listener` [\<Function\>][Function] 回调函数

根据事件名称`eventName`添加一个监听函数在事件监听列表对应名称函数之前。不会检查当前事件是否已经添加监听器。给同一个事件名称`eventName`添加多个监听器，那么多个监听器函数都会被调用。

```js
  server.prependListener('connection', (stream) => {
    console.log('someone connected!');
  });
```

返回`EventEmitter`的引用，所以可以使用链式调用。

### emitter.prependOnceListener(eventName, listener)
>   v6.0.0+

*   `eventName` [\<String\>][String] | [\<Symbol\>][Symbol] 监听的事件名称
*   `listener` [\<Function\>][Function] 回调函数

根据事件名称`eventName`添加一个监听函数在事件监听列表对应名称函数之前只添加一次。下次`eventName`被触发，这个监听器会先移除，然后执行回调。

```js
  server.prependOnceListener('connection', (stream) => {
    console.log('Ah, we have our first user!');
  });
```
返回`EventEmitter`的引用，所以可以使用链式调用。

### emitter.removeAllListeners([eventName])
>   v0.1.26

根据事件名称`eventName`，移除所有监听这个事件的所有监听器。

在别的代码里面添加的监听器在这里通过这种不好的实践方式删除是不合理的，特别是`EventEmitter`实例由其它的组件或模块创建的时候（例如：`socket` 或 文件流）。

返回`EventEmitter`的引用，所以可以使用链式调用。

### emitter.removeListener(eventName, listener)
>   v0.1.26+

根据事件名称`eventName`删除事件监听列表里面指定的监听器`listener`。

```js
  var callback = (stream) => {
    console.log('someone connected!');
  };
  server.on('connection', callback);
  // ...
  server.removeListener('connection', callback);
```

`removeListener`最多异常只会从实例的事件监听器数组移除一个监听器。如果存在一些实例的事件名称`eventName`添加了多次事件监听器，那必须通过多次调用`removeListener`移除事件监听器。

注意，事件一旦被触发，所有监听这个事件的监听器都会根据添加顺序被触发执行。这就意味着在事件触发之后事件监听器执行完成之前触发执行`removeListener()`或`removeAllListener()`都必须等到当前回调或全部事件全部执行完成时才会按函数预期的那样继续执行。

```js
  const myEmitter = new MyEmitter();

  var callbackA = () => {
    console.log('A');
    myEmitter.removeListener('event', callbackB);
  };

  var callbackB = () => {
    console.log('B');
  };

  myEmitter.on('event', callbackA);

  myEmitter.on('event', callbackB);

  // callbackA removes listener callbackB but it will still be called.
  // Internal listener array at time of emit [callbackA, callbackB]
  myEmitter.emit('event');
    // Prints:
    //   A
    //   B

  // callbackB is now removed.
  // Internal listener array [callbackA]
  myEmitter.emit('event');
    // Prints:
    //   A
```

因为监听器是使用内联数组管理，调用删除函数会改变所有注册的事件监听器的下标。这不会对事件监听器调用顺序进行改变，但它会使`emitter.listeners()`方法返回的事件监听数组需要被从新创建。

返回`EventEmitter`的引用，所以可以使用链式调用。

### emitter.setMaxListeners(n)
>   v0.3.5+

默认情况下，如果一个事件被添加超过了10个监听器将会打印告警信息。这对内存泄漏的排查很有帮助。不是所有的事件都应该限制10个监听器。特定的`EventEmitter`实例可以使用`emitter.setMaxListeners(n)`来修改这个这个限制.这个值可以设置为`Infinity`(或 0)表示可以添加无穷多的监听器。

返回`EventEmitter`的引用，所以可以使用链式调用。


=====================================[完]======================================

[String]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#String_type
[Symbol]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Symbol_type
[Function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function