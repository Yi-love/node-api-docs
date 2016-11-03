# 内容目录
    



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
*   `all`: [\<Boolean\>][Boolean] `true`的时候，回调函数返回解决的地址数组，其它时候返回单个地址，默认`false`。

参数使用例子：

```js
  {
    family: 4,
    hints: dns.ADDRCONFIG | dns.V4MAPPED,
    all: false
  }
```

`callback`回调函数有3个参数`(err , address , family)`。`address`表示一个`IPv4`或`IPv6`字符串地址。`family`参数表示`address`的家族符号为`4`或`6`中的任何一个(不一定要把最初的值传递给`lookup`)。

如果`all`参数设置为`true`,回调参数改变为`(err,addresses)`，并且`addresses`将变为属性`address`和`family`的对象数组.

出错情况下，`err`是一个`Error`对象，`err.code`代码错误码。需要注意的是`err.code`被设置为`ENOENT`时，不仅仅是在主机名不存在的情况下，在查找文件描述符失败的时候也会如此。

`dns.lookup()`不需要对DNS协议做任何事。实现使用的是操作系统底层设备可以将名称和地址联系起来，反之亦然. 这个实现有些微妙，但这对Node.js程序的行为有着重要的影响。使用`dns.lookup()`之前，请仔细阅读[Implementation-considerations-section][Implementation-considerations-section]。

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

=============================[未完待续...]==============================


[Implementation-considerations-section]: https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_implementation_considerations
[Number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Number_type
[supported-getaddrinfo-flags]: https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_supported_getaddrinfo_flags