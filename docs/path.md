path (路径)
====
- Windows 与 POSIX
- path.basename(path[, ext])  

  ```js
  path.basename('/foo/bar/baz/asdf/quux.html');
  // 返回: 'quux.html'

  path.basename('/foo/bar/baz/asdf/quux.html', '.html');
  // 返回: 'quux'
  ```
- path.delimiter
  - POSIX

  ```js
  console.log(process.env.PATH);
  // 输出: '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

  process.env.PATH.split(path.delimiter);
  // 返回: ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']
  ```
  - Windows

  ```js
  console.log(process.env.PATH);
  // 输出: 'C:\Windows\system32;C:\Windows;C:\Program Files\node\'

  process.env.PATH.split(path.delimiter);
  // 返回: ['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\node\\']
  ```
- path.dirname(path)
```js
path.dirname('/foo/bar/baz/asdf/quux');
// 返回: '/foo/bar/baz/asdf'
```
- path.extname(path)

  ```js
  path.extname('index.html');
  // 返回: '.html'

  path.extname('index.coffee.md');
  // 返回: '.md'

  path.extname('index.');
  // 返回: '.'

  path.extname('index');
  // 返回: ''

  path.extname('.index');
  // 返回: ''
  ```
- path.format(pathObject)
- path.isAbsolute(path)
- path.join([...paths])
- path.normalize(path)
- path.parse(path)

  ```js
  path.parse('C:\\path\\dir\\file.txt');
  // 返回:
  // { root: 'C:\\',
  //   dir: 'C:\\path\\dir',
  //   base: 'file.txt',
  //   ext: '.txt',
  //   name: 'file' }
  ```
- path.posix
- path.relative(from, to)
- path.resolve([...paths])
  - 如果处理完全部给定的 path 片段后还未生成一个绝对路径，则当前工作目录会被用上。
  - 生成的路径是规范化后的，且末尾的斜杠会被删除，除非路径被解析为根目录。
  - 长度为零的 path 片段会被忽略。
  - 如果没有传入 path 片段，则 path.resolve() 会返回当前工作目录的绝对路径。
  
  ```js
  path.resolve('/foo/bar', './baz');
  // 返回: '/foo/bar/baz'

  path.resolve('/foo/bar', '/tmp/file/');
  // 返回: '/tmp/file'

  path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
  // 如果当前工作目录为 /home/myself/node，
  // 则返回 '/home/myself/node/wwwroot/static_files/gif/image.gif'
  ```
- path.sep
- path.win32
