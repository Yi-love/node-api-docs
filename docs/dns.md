# 内容目录

*   [DNS](#dns)
    *   [dns.getServers()](#dnsgetservers)
    *   [dns.lookup(hostname[, options], callback)](#dnslookuphostname-options-callback)
        *   [Supported getaddrinfo flags](#supported-getaddrinfo-flags)
    *   [dns.lookupService(address, port, callback)](#dnslookupserviceaddress-port-callback)
    *   [dns.resolve(hostname[, rrtype], callback)](#dnsresolvehostname-rrtype-callback)
    *   [dns.resolve4(hostname, callback)](#dnsresolve4hostname-callback)
    *   [dns.resolve6(hostname, callback)](#dnsresolve6hostname-callback)
    *   [dns.resolveCname(hostname, callback)](#dnsresolvecnamehostname-callback)
    *   [dns.resolveMx(hostname, callback)](#dnsresolvemxhostname-callback)
    *   [dns.resolveNaptr(hostname, callback)](#dnsresolvenaptrhostname-callback)
    *   [dns.resolveNs(hostname, callback)](#dnsresolvenshostname-callback)
    *   [dns.resolveSoa(hostname, callback)](#dnsresolvesoahostname-callback)
    *   [dns.resolveSrv(hostname, callback)](#dnsresolvesrvhostname-callback)
    *   [dns.resolvePtr(hostname, callback)](#dnsresolveptrhostname-callback)
    *   [dns.resolveTxt(hostname, callback)](#dnsresolvetxthostname-callback)
    *   [dns.reverse(ip, callback)](#dnsreverseip-callback)
    *   [dns.setServers(servers)](#dnssetserversservers)
    *   [Error codes](#error-codes)
    *   [Implementation considerations](#implementation-considerations)
        *   [dns.lookup()](#dnslookup)
        *   [dns.resolve(), dns.resolve*() and dns.reverse()](#dnsresolve-dnsresolve-and-dnsreverse)

# DNS
>   稳定性：稳定

`dns`模块中包含的函数属于两个不同的类别:

1) 函数使用底层操作系统设施进行名称解析，不一定要执行任何网络通信。这个类别只包含一个函数：
`dns.lookup()`。在同一操作系统上开发者希望以同样的方式进行名称解析和其它应用程序使用`dns.lookup()`进行解析行为一致。

例如，查找`nodejs.org`。

```js
  const dns = require('dns');

  dns.lookup('nodejs.org', (err, addresses, family) => {
    console.log('addresses:', addresses);
  });
```

2) 函数连接到真实的DNS服务去进行名称解析，并且总是使用网络来执行DNS查询。这个类别包含所有`dns`模块的所有函数除了`dns.lookup()`函数。
这些函数没有使用配置文件，像`dns.lookup()`那样使用类似（`/etc/hosts`）。这些函数应该使用在不想被操作系统底层进行名称解析的域名上，并且总是使用网络来执行DNS查询。

下面是一个示例,解决`'nodejs.org'`然后反向解析返回的IP地址。

```js
  const dns = require('dns');

  dns.resolve4('nodejs.org', (err, addresses) => {
    if (err) throw err;

    console.log(`addresses: ${JSON.stringify(addresses)}`);

    addresses.forEach((a) => {
      dns.reverse(a, (err, hostnames) => {
        if (err) {
          throw err;
        }
        console.log(`reverse for ${a}: ${JSON.stringify(hostnames)}`);
      });
    });
  });
```

两者直接有一点点的不同，详情请查看[Implementation considerations section][Implementation-considerations-section]。

## dns.getServers()
>   v0.11.3+

根据名称解析返回ip字符串数组。


## dns.lookup(hostname[, options], callback)
>   v0.1.90+

解析主机名(例如, `nodejs.org`)到第一个发现(`IPv4`)或AAAA级(`IPv6`)记录。可选参数`options`可以是`object`对象或整数`integer`。如果不传`options`参数，那么`IP4`和`IP6`都有效。
`options`是一个整型的时候，必须是`4`或`6`。

`options`是一个对象时包含以下属性：

*   `family` [\<Number\>][Number] `IP`家族.如果传入,必须是`4`或`6`。如果不提供，`IP4`和`IP6`地址都有效。
*   `hints`: [\<Number\>][Number] 如果传入，它应该是支持一个或多个`getaddrinfo`标记。如果`hints`没有提供，没有标记传入`getaddrinfo`。多个标记可以通过逻辑`or`计算它们的值。更多支持标记信息请查看[supported getaddrinfo flags][supported-getaddrinfo-flags]
*   `all`: [\<Boolean\>][Boolean] `true`的时候，回调函数返回解析的地址数组，其它时候返回单个地址，默认`false`。

参数使用例子：

```js
  {
    family: 4,
    hints: dns.ADDRCONFIG | dns.V4MAPPED,
    all: false
  }
```

`callback`回调函数有3个参数`(err , address , family)`。`address`表示一个`IPv4`或`IPv6`字符串地址。`family`参数表示`address`的家族符号为`4`或`6`中的任何一个(不一定要把最初的值传递给`lookup`)。

如果`all`参数设置为`true`,回调参数改变为`(err,addresses)`，并且`addresses`将为包含属性`address`和`family`的对象数组.

出错情况下，`err`是一个`Error`对象，`err.code`代码错误码。需要注意的是`err.code`被设置为`ENOENT`时，不仅仅是在主机名不存在的情况下，在查找文件描述符失败的时候也会如此。

`dns.lookup()`不需要对DNS协议做任何事。实现使用的是操作系统底层设备来将主机名和ip地址联系起来，反之亦然. 这个实现有些微妙，但这对Node.js程序的行为有着重要的影响。使用`dns.lookup()`之前，请仔细阅读[Implementation-considerations-section][Implementation-considerations-section]。

### Supported getaddrinfo flags
`dns.lookup()`可选参数`options`的`hints`参数:

*   `dns.ADDRCONFIG` : 根据当前系统支持的地址类型返回确切的地址类型。例如，只有在当前系统中仅存在一个IPv4地址的时候才返回IPv4地址。环回地址(127.0.0.1)不考虑。
*   `dns.V4MAPPED` : 如果指定为IPv6家族，但没有查找到IPv6地址，就会返回由 IPv4映射的IPv6地址。注意有些操作系统不支持(例如：FreeBSD 10.1).

## dns.lookupService(address, port, callback)
>   V0.11.14+

使用操作系统底层`getnameinfo`实现传入参数`address`和`port`返回主机名和服务。

如果`address`不是有效的IP地址，会抛出`TypeError`。`port`必须是一个整数.如果不是规定的端口号，会抛出`TypeError`.

`callback`有3个参数`(err , hostname , service)。`hostname`和`service`都是字符串（例如：分别为`localhost` 和`http`）。

出错情况下，`err`是一个`Error`对象，`err.code`代码错误码。

```js
  const dns = require('dns');
  dns.lookupService('127.0.0.1', 22, (err, hostname, service) => {
    console.log(hostname, service);
      // Prints: localhost ssh
  });
```

## dns.resolve(hostname[, rrtype], callback)
>   0.1.27+

使用DNS协议根据指定类型的`rrtype`数组解析`hostname`（例如： `'nodejs.org'`）.

`rrtype`有效值如下：

*   `'A'` - IPv4 地址。默认
*   `'AAAA'` - IPv6地址
*   `'MX'` - 邮件交换记录
*   `'TXT'` - 文本记录
*   `'SRV'` - SRV记录
*   `'PTR'` - PTR记录
*   `'NS'` -  名称服务器记录
*   `'CNAME'` - 规范内的名称记录
*   `'SOA'` - 权限控制记录
*   `'NAPTR'` - 名字权限控制引用记录

`callback`函数有2个参数`(err,addresses)`。成功的时候，`addresses`将是一个数组，除了解析`  SOA`记录返回对象结构,`dns.resolveSoa()`同样返回对象结构。`adresses`的每一项类型取决于记录类型，`lookup`方法中有相应的文档描述。

出错情况下，`err`是一个`Error`对象，`err.code`代码错误码。错误码列表：[here](https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_error_codes).

## dns.resolve4(hostname, callback)
>   v0.1.16+

使用`DNS`协议解析IPv4地址主机名(`A`记录)。`adresses`参数是传递给`callback`函数的IPv4地址数组。（例如：`['74.125.79.104', '74.125.79.105', '74.125.79.106']`）

## dns.resolve6(hostname, callback)
>   v0.1.16+

使用`DNS`协议解析IPv6地址主机名(`AAAA`记录)。`adresses`参数是传递给`callback`函数的IPv6地址数组.

## dns.resolveCname(hostname, callback)
>   V0.3.2+

使用`DNS`协议解析`CNAME`记录主机名。`adresses`参数是传递给`callback`函数规范内有效的主机名数组（例如：`['bar.example.com']`）.

## dns.resolveMx(hostname, callback)
>   v0.1.27+

使用DNS协议解决邮件交换记录主机名(`MX`记录)。`adresses`参数是传递给`callback`函数主机名对象数组，对象包含`priority`和`exchange`属性（例如： `[{priority: 10, exchange: 'mx.example.com'}, ...]`）。

## dns.resolveNaptr(hostname, callback)
>   v0.9.12+

使用DNS协议来解决基于正则表达式匹配的记录(`NAPTR`记录)的主机名。`callback`函数有2个参数`(err , addresses)`。adresses`参数是传递给`callback`函数主机名对象数组，对象包含属性：

*   `flags`
*   `service`
*   `regexp`
*   `replacement`
*   `order`
*   `preference`

例如：

```js
  {
    flags: 's',
    service: 'SIP+D2U',
    regexp: '',
    replacement: '_sip._udp.example.com',
    order: 30,
    preference: 100
  }
```

## dns.resolveNs(hostname, callback)
>   v0.1.90+

使用DNS协议解决名称服务器主机名记录(`NS`记录)。`adresses`为有效的名称服务器记录主机名数组（eg:`['ns1.example.com', 'ns2.example.com']`）。

## dns.resolveSoa(hostname, callback)
>   v0.11.10+

使用DNS协议解决主机名开始权威（子域名）记录(`SOA`记录)。`addresses`为为一个对象包含以下属性：

*   `nsname`
*   `hostmaster`
*   `serial`
*   `refresh`
*   `retry`
*   `expire`
*   `minttl`

```js
  {
    nsname: 'ns.example.com',
    hostmaster: 'root.example.com',
    serial: 2013101809,
    refresh: 10000,
    retry: 2400,
    expire: 604800,
    minttl: 3600
  }
```

## dns.resolveSrv(hostname, callback)
>   v0.1.27+

使用DNS协议来解决主机名服务记录(SRV记录)。`addresses`传入`callback`函数，`addresses`为对象数组,每个对象包含以下属性：

*   `priority`
*   `weight`
*   `port`
*   `name`

```js
  {
    priority: 10,
    weight: 5,
    port: 21223,
    name: 'service.example.com'
  }
```

## dns.resolvePtr(hostname, callback)
>   v6.0.0+

使用DNS协议解决主机名引用记录(PTR记录)。`addresses`参数将一个字符串数组传递给回调函数`callback`,其中包含回复记录。

## dns.resolveTxt(hostname, callback)
>   v0.1.27+

使用DNS协议解决文本查询主机名(TXT记录)。`addresses`参数传递给回调函数`callback`的地址是一个二维数组的文本记录用于主机名(例如：`[ ['v=spf1 ip4:0.0.0.0 ', '~all' ] ]`).
每个数组文本块包含一条记录。根据用例,这些可以是连接在一起或单独对待。

## dns.reverse(ip, callback)
>   v0.1.16+

执行一个反向DNS查询解决IPv4和IPv6地址的主机名的数组。

`callback`包含2个参数`(err,hostnames)` 。 根据`ip`返回`hostnames`数组。

出错情况下，`err`是一个`Error`对象，`err.code`代码错误码。错误码列表：[here](https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_error_codes).

## dns.setServers(servers)
>   v0.11.3+

设置解析时使用的服务器IP地址。`servers`服务器参数是一个数组的IPv4和IPv6地址.

如果指定的ip包含端口，端口会被移除。

`ip`地址无效将会报错。

`dns.setServers()`方法不要在DNS查询进程中使用。

## Error codes
每个DNS查询可以返回一个错误代码如下:

*   `dns.NODATA`: DNS服务返回没有数据.
*   `dns.FORMERR`: DNS服务器查询没有格式化.
*   `dns.SERVFAIL`: DNS服务器返回失败。
*   `dns.NOTFOUND`: 域名未找到。
*   `dns.NOIMP`: DNS服务器不执行请求的操作。
*   `dns.REFUSED`: 查询DNS服务器拒绝。
*   `dns.BADQUERY`:  未格式化DNS查询。
*   `dns.BADNAME`: 未格式化主机名
*   `dns.BADFAMILY`: 没有提供地址族
*   `dns.BADRESP`: 未格式化DNS回复
*   `dns.CONNREFUSED`: 无法连接DNS服务器
*   `dns.TIMEOUT`: 连接DNS服务器超时
*   `dns.EOF`: 文件末尾
*   `dns.FILE`: 读取文件错误
*   `dns.NOMEM`: 内存溢出
*   `dns.DESTRUCTION`: 通道以及销毁
*   `dns.BADSTR`: 未格式化字符串
*   `dns.BADFLAGS`: 指定非法标记
*   `dns.NONAME`: 给定的主机名不是数字。
*   `dns.BADHINTS`: 指定非法的提示标志。
*   `dns.NOTINITIALIZED`: `c-ares`异步DNS请求库初始化未完成。
*   `dns.LOADIPHLPAPI`: 加载`iphlpapi.dll`(Windows IP辅助API应用程序接口模块)错误
*   `dns.ADDRGETNETWORKPARAMS`: 找不到`GetNetworkParams`(读取本机DNS信息)函数
*   `dns.CANCELLED`: DNS查询取消

## Implementation considerations

> 注意事项

尽管`dns.lookup()`和各种`dns.resolve *()/ dns.reverse()``函数有相同的目标将网络的名字与网络地址联系在一起(反之亦然)，他们的行为是完全不同的。
这些差异可以有微妙但重大影响着Node.js程序行为。

### dns.lookup()

在底层,`dns.lookup()`使用操作系统设施与大多数其他程序相同。例如，`dns.lookup()`几乎总是解析给定的主机名与`ping`命令一样。在许多类POSIX操作系统中，
`dns.lookup()`函数的行为可以通过改变`nsswitch.conf(5)`并且/或`resolv.conf(5)`设置进行改变，但是需要注意改变这些文件就意味着改变所有正在这个操作系统中运行
的所有进程的行为。

尽管以异步`JavaScript`的角度来调用`dns.lookup()`,但在内部`libuv`底层线程池中却是同步的调用`getaddrinfo(3)`。因为`libuv`线程池有固定大小，它意味着,如果出于某种原因调用`getaddrinfo(3)`需要很长时间,其他操作可以运行在libuv线程池中(如文件系统操作)就会存在性能问题。为了缓解这个问题,一个可能的解决办法是增加libuv的线程池大小通过设置`'UV_THREADPOOL_SIZE'`环境变量值大于`4`(当前的默认值).更多libuv线程池信息，请查看：[here](http://docs.libuv.org/en/latest/threadpool.html)

### dns.resolve(), dns.resolve*() and dns.reverse()
这些功能实现与dns.lookup()截然不同。它们不仅没有使用`getaddrinfo(3)`并且通过网络执行DNS查询。使用异步网络通信，并且没有使用libuv线程池。

因此,这些函数不会像使用libuv线程池的`dns.lookup()`函数一样会对其它进程有负面影响。

它们不像`dns.lookup()`一样使用相同的配置文件。例如，它们不会使用来自`/etc/hosts`配置。


=============================[完]==============================


[Boolean]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Boolean_type
[Implementation-considerations-section]: https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_implementation_considerations
[Number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type
[supported-getaddrinfo-flags]: https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_supported_getaddrinfo_flags