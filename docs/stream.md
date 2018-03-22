stream 模块快速指南
==================

stream(流)
---------
流(stream)在 Node.js 中是处理流数据的抽象接口(abstract interface)。 stream 模块提供了基础的 API 。使用这些 API 可以很容易地来构建实现流接口的对象。

Node.js 提供了多种流对象。 例如， HTTP 请求 和 process.stdout 就都是流的实例。

流可以是可读的、可写的，或是可读写的。所有的流都是 EventEmitter 的实例。

stream 模块可以通过以下方式引入：
```js
const stream = require('stream');
```
尽管所有的 Node.js 用户都应该理解流的工作方式，这点很重要， 但是 `stream` 模块本身只对于那些需要创建新的流的实例的开发者最有用处。 对于主要是 *消费* 流的开发者来说，他们很少（如果有的话）需要直接使用 stream 模块。

本文档的组织
-----------
本文档主要分为两节，第三节是一些额外的注意事项。
第一节阐述了在应用中 和`使用` 流相关的 API 。
第二节阐述了 和`实现` 新的流类型相关的 API。

Types of streams (流的类型)
-------------------------
Node.js 中有四种基本的流类型：

- Readable - 可读的流 (例如 fs.createReadStream()).
- Writable - 可写的流 (例如 fs.createWriteStream()).
- Duplex   - 可读写的流 (例如 net.Socket).
- Transform - 在读写过程中可以修改和变换数据的 Duplex 流 (例如 zlib.createDeflate()).

Object Mode (对象模式)
---------------------

所有使用 Node.js API 创建的流对象都只能操作 strings 和 Buffer（或 Uint8Array） 对象。但是，通过一些第三方流的实现，你依然能够处理其它类型的 JavaScript 值 (除了 null，它在流处理中有特殊意义)。 这些流被认为是工作在 “对象模式”（object mode）。

在创建流的实例时，可以通过 objectMode 选项使流的实例切换到对象模式。试图将已经存在的流切换到对象模式是不安全的。

Buffering (缓冲)
----------------
Writable 和 Readable 流都会将数据存储到内部的缓冲器（buffer）中。这些缓冲器可以 通过相应的 writable._writableState.getBuffer() 或 readable._readableState.buffer 来获取。

缓冲器的大小取决于传递给流构造函数的 highWaterMark 选项。 对于普通的流， highWaterMark 选项指定了总共的字节数。对于工作在对象模式的流， highWaterMark 指定了对象的总数。

当可读流的实现调用 stream.push(chunk) 方法时，数据被放到缓冲器中。如果流的消费者 没有调用 stream.read() 方法， 这些数据会始终存在于内部队列中，直到被消费。

当内部可读缓冲器的大小达到 highWaterMark 指定的阈值时，流会暂停从底层资源读取数据，直到当前 缓冲器的数据被消费 (也就是说， 流会在内部停止调用 readable._read() 来填充可读缓冲器)。
```js
const fs = require('fs');
const readstream = fs.createReadStream('sample.txt', {start: 90, end: 99}); // read the last 10 bytes of a file 100 bytes long
readstream.pipe(process.stdout);
```
可写流通过反复调用 writable.write(chunk) 方法将数据放到缓冲器。 当内部可写缓冲器的总大小小于 highWaterMark 指定的阈值时， 调用 writable.write() 将返回true。 一旦内部缓冲器的大小达到或超过 highWaterMark ，调用 writable.write() 将返回 false 。

stream API 的关键目标， 尤其对于 stream.pipe() 方法， 就是限制缓冲器数据大小，以达到可接受的程度。这样，对于读写速度不匹配的源头和目标，就不会超出可用的内存大小。

Duplex 和 Transform 都是可读写的。 在内部，它们都维护了 两个 相互独立的缓冲器用于读和写。 在维持了合理高效的数据流的同时，也使得对于读和写可以独立进行而互不影响。 例如， net.Socket 就是 Duplex 的实例，它的可读端可以消费从套接字（socket）中接收的数据， 可写端则可以将数据写入到套接字。 由于数据写入到套接字中的速度可能比从套接字接收数据的速度快或者慢， 在读写两端使用独立缓冲器，并进行独立操作就显得很重要了。

API for stream consumers (流 消费者的api)
------------------------
> 几乎所有的 Node.js 应用，不管多么简单，都在某种程度上使用了流。 下面是在 Node.js 应用中使用流实现的一个简单的 HTTP 服务器：  

  ```js
  const http = require('http');
  const PORT = 8080;

  const server = http.createServer((req, res) => {
    // req 是 http.IncomingMessage 的实例，这是一个 Readable Stream
    // res 是 http.ServerResponse 的实例，这是一个 Writable Stream

    let body = '';
    // 接收数据为 utf8 字符串，
    // 如果没有设置字符编码，将接收到 Buffer 对象。
    req.setEncoding('utf8');

    // 如果监听了 'data' 事件，Readable streams 触发 'data' 事件
    req.on('data', (chunk) => {
      body += chunk;
    });

    // end 事件表明整个 body 都接收完毕了
    req.on('end', () => {
      try {
        const data = JSON.parse(body);
        // 发送一些信息给用户
        res.write(typeof data);
        res.end();
      } catch (er) {
        // json 数据解析失败
        res.statusCode = 400;
        return res.end(`error: ${er.message}`);
      }
    });
  })

  server.listen(PORT)
  ```
Writable 流 (比如例子中的 res) 暴露了一些方法，比如 write() 和 end() 。这些方法可以将数据写入到流中。

当流中的数据可以读取时，Readable 流使用 EventEmitter API 来通知应用。 这些数据可以使用多种方法从流中读取。

Writable 和 Readable 流都使用了 EventEmitter API ，通过多种方式， 与流的当前状态进行交互。

Duplex 和 Transform 都是同时满足 Writable 和 Readable 。

对于只是简单写入数据到流和从流中消费数据的应用来说， 不要求直接实现流接口，通常也不需要调用 require('stream')。

Writeable Streams (可写流)
------
可写流（Writeable streams）是对数据写入 目的地 （destination） 的一种抽象。

Writable 的例子包括了：

- HTTP requests, on the client
- HTTP responses, on the server
- fs write streams
- zlib streams
- crypto streams
- TCP sockets
- child process stdin
- process.stdout, process.stderr

注意: 上面的某些例子事实上是 Duplex 流，只是实现了 Writable 接口。

所有 Writable 流都实现了 stream.Writable 类定义的接口。

尽管特定的 Writable 流的实现可能略有差别， 所有的 Writable streams 都可以按一种基本模式进行使用，如下面例子所示：

```js
const mystream = getWritableStreamSomehow();
mystream.write('some data');
mystream.write('some more data');
mystream.end('done writing data');
```
- Event: 'close'
- Event: 'drain'
- Event: 'error'
- Event: 'finish'
- Event: 'pipe'
- Event: 'unpipe'
- writable.cork()
- writable.end([chunk][, encoding][, callback])
- writable.setDefaultEncoding(encoding)
- writable.uncork()
- writable.writableHighWaterMark
- writable.writableLength
- writable.write(chunk[, encoding][, callback])
- writable.destroy([error])

Readable Streams (可读流)
------
可读流（Readable streams）是对提供数据的 源头 （source）的抽象。

Readable 的例子包括：

- HTTP responses, on the client
- HTTP requests, on the server
- fs read streams
- zlib streams
- crypto streams
- TCP sockets
- child process stdout and stderr
- process.stdin

所有的 Readable 都实现了 stream.Readable 类定义的接口。
对于大多数用户，建议使用 readable.pipe() 方法来消费流数据，因为它是最简单的一种实现。开发者如果要精细地控制数据传递和产生的过程，可以使用 EventEmitter 和 readable.pause()/readable.resume() 提供的 API 。

- Event: 'close'
- Event: 'data'
- Event: 'end'
- Event: 'error'
- Event: 'readable'
- readable.isPaused()
- readable.pause()
- readable.pipe(destination[, options])
- readable.readableHighWaterMark
- readable.read([size])
> 一般来说，建议开发人员避免使用'readable'事件和readable.read()方法，使用readable.pipe()或'data'事件代替。
- readable.readableLength
- readable.resume()
- readable.setEncoding(encoding)
- readable.unpipe([destination])
- readable.unshift(chunk)
- readable.wrap(stream)
- readable.destroy([error])


Duplex and Transform Streams
------
