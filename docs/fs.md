fs模块快速指南
=============
---
> fs 模块包含以下几个类class  

* fs.Stats  
* fs.FSWatcher  
* fs.ReadStream  
* fs.WriteStream  

fs.Stats类
--------
> fs.Stats的实例由这几种方法返回*fs.stat()*, *fs.lstat()* 和 *fs.fstat()*   

  ```js
  const fs = require('fs');
  const sep = require('path').sep;

  const files = [
    'C:\\Users\\Nick_Nie\\Downloads\\lyxz.rar',
    'C:\\Users\\Nick_Nie\\Downloads',
    'C:\\Users\\Nick_Nie\\Downloads\\lyxz.rar - Shortcut.lnk',
  ];

  files.forEach(el => {
    // fs.stat(path, callback(error, stats))
    // path could be string|buffer|url
    fs.stat(el, (err, stats) => {
      if (err) throw err;
      console.log(`${el.split(sep).pop()} stats : \n${JSON.stringify(stats, null, 2)}`);
    });
  });
  ```
fs.Stats的一个实例：
  ```js
  {
    "dev": 2921160410,
    "mode": 33206,
    "nlink": 1,
    "uid": 0,
    "gid": 0,
    "rdev": 0,
    "ino": 11821949022163888,
    "size": 644728,
    "atimeMs": 1520302008445.7664, // access time
    "mtimeMs": 1520302013926.8044, // when file data last modified
    "ctimeMs": 1521692075051.1086, // when file status last changed
    "birthtimeMs": 1520302010799.1658, // time of file creation
    "atime": "2018-03-06T02:06:48.446Z",
    "mtime": "2018-03-06T02:06:53.927Z",
    "ctime": "2018-03-22T04:14:35.051Z",
    "birthtime": "2018-03-06T02:06:50.799Z"
  }
  ```

fs.FSWatcher类
-----------
> fs.FSWatcher的实例由 *fs.watch()* 方法返回

* 写法1：
  ```js
  const fs = require('fs');
  const sep = require('path').sep;
  const { tmpdir } = require('os');

  const watchlistener = (eventType, filename) => {
    console.log(`event type is: ${eventType}`);
    filename ? console.log(`filename provided: ${filename}`): console.log('filename not provided');
    // watcher.close();
  };
  const watcher = fs.watch(tmpdir(), watchlistener);
  ```
* 写法2：  

  ```js
  const fs = require('fs');
  const path = require('path');
  const sep = require('path').sep;
  const { tmpdir } = require('os');

  var original = process.argv[2];
  const watchlistener = (eventType, filename) => {
    console.log(`event type is: ${eventType}`);
    filename ? console.log(`filename provided: ${filename}`): console.log('filename not provided');
    // watcher.close();
  };

  const watcher = fs.watch(tmpdir());

  watcher.on('error', (error) => {
    console.error(error);
    process.exit(1);
  });

  watcher.on('change', watchlistener);

  // 每10秒重命名一次
  setInterval(() => {
    var newFilePath = path.join(tmpdir(), `${Date.now()}`);
    fs.renameSync(original, newFilePath);
    original = newFilePath;
  }, 10000)
  ```
相关方法：
* watcher.close() 停掉watch

fs.ReadStream类
-------------
> fs.ReadStream类的实例由fs.createReadStream()创建。它的父类为stream.Readable。

* fs.createReadStream()
  - input
    - path
    - options. string|object
  - output
    - ReadStream

fs.WriteStream类
----------------
> fs.WriteStream类的实例由fs.createWriteStream()创建。它的父类为stream.Writeable。

File Descriptor
---------------
> To simplify things for users, Node.js abstracts away the specific differences between operating systems and assigns all open files a numeric file descriptor.
The *fs.open()* method is used to allocate a new file descriptor. Once allocated, the file descriptor may be used to read data from, write data to, or request information about the file.

Node.js中File Descriptor 由两种方式产生：
1. fs.open(path, flags[, mode], callback)
2. fs.openSync(path, flags[, mode])

  ```js

  ```

异步方法
-----------------
* fs.access(path[, mode], callback)
  ```js

  ```
* fs.appendFile(file, data[, options], callback)
  ```js

  ```
* fs.chmod(path, mode, callback)
  ```js

  ```
* fs.chown(path, uid, gid, callback)
  ```js

  ```
* fs.close(fd, callback)
  ```js

  ```
* fs.copyFile(src, dest[, flags], callback)
  ```js

  ```
* fs.fchmod(fd, mode, callback)
  ```js

  ```
* fs.fchown(fd, uid, gid, callback)
  ```js

  ```
* fs.fdatasync(fd, callback)
  ```js

  ```
- fs.fstat(fd, callback)
  ```js

  ```
- fs.fsync(fd, callback)
  ```js

  ```
- fs.ftruncate(fd[, len], callback)
  ```js

  ```
- fs.futimes(fd, atime, mtime, callback)
  ```js

  ```
- fs.lchmod(path, mode, callback)
  ```js

  ```
- fs.lchown(path, uid, gid, callback)
  ```js

  ```
- fs.link(existingPath, newPath, callback)
  ```js

  ```
- fs.lstat(path, callback)
  ```js

  ```
- fs.mkdir(path[, mode], callback)
  ```js

  ```
- fs.mkdtemp(prefix[, options], callback)
  ```js

  ```
- fs.open(path, flags[, mode], callback)
  ```js

  ```
- fs.read(fd, buffer, offset, length, position, callback)
  ```js

  ```
- fs.readdir(path[, options], callback)
  ```js

  ```
- fs.readFile(path[, options], callback)
  ```js

  ```
- fs.readlink(path[, options], callback)
  ```js

  ```
- fs.realpath(path[, options], callback)
  ```js

  ```
- fs.realpath.native(path[, options], callback)
  ```js

  ```
- fs.rename(oldPath, newPath, callback)
  ```js

  ```
- fs.rmdir(path, callback)
  ```js

  ```
- fs.stat(path, callback)
  ```js

  ```
- fs.symlink(target, path[, type], callback)
  ```js

  ```
- fs.truncate(path[, len], callback)
  ```js

  ```
- fs.unlink(path, callback)
  ```js

  ```
- fs.utimes(path, atime, mtime, callback)
  ```js

  ```

同步方法
-------------------
> 被调用的同步方法时要放在 *try {} catch (error) {}* 中。

  ```js
  try {
    fs.accessSync('demo-fs-asscessSync.js', fs.constants.R_OK);  
  } catch (error) {
    // error handling
    console.error(error);
  }
  ```
- fs.accessSync(path[, mode])
- fs.appendFileSync(file, data[, options])
- fs.chmodSync(path, mode)
- fs.chownSync(path, uid, gid)
- fs.closeSync(fd)
- fs.constants
- fs.copyFileSync(src, dest[, flags])
- fs.createReadStream(path[, options])
  ```js
  const defaults = {
    flags: 'r',
    encoding: null,
    fd: null,
    mode: 0o666,
    autoClose: true,
    highWaterMark: 64 * 1024
  };
  fs.createReadStream('file.txt', defaults) // return <ReadStream>
  ```
- fs.createWriteStream(path[, options])
  ```js
  const default = {
    flags: 'w',
    encoding: 'utf8',
    fd: null,
    mode: 0o666,
    autoClose: true
  }
  fs.createWriteStream('file.txt', defaults) // return <WriteStream>
  ```
- fs.existsSync(path)
- fs.fchmodSync(fd, mode)
- fs.fchownSync(fd, uid, gid)
- fs.fdatasyncSync(fd)
- fs.fstatSync(fd)
- fs.fsyncSync(fd)
- fs.ftruncateSync(fd[, len])
- fs.futimesSync(fd, atime, mtime)
- fs.lchmodSync(path, mode)
- fs.lchownSync(path, uid, gid)
- fs.linkSync(existingPath, newPath)
- fs.lstatSync(path)
- fs.mkdirSync(path[, mode])
- fs.mkdtempSync(prefix[, options])
- fs.openSync(path, flags[, mode])
- fs.readdirSync(path[, options])
- fs.readFileSync(path[, options])
- fs.readlinkSync(path[, options])
- fs.readSync(fd, buffer, offset, length, position)
- fs.realpathSync(path[, options])
- fs.realpathSync.native(path[, options])
- fs.renameSync(oldPath, newPath)
- fs.rmdirSync(path)
- fs.statSync(path)
- fs.symlinkSync(target, path[, type])
- fs.truncateSync(path[, len])
- fs.unlinkSync(path)
- fs.unwatchFile(filename[, listener])
- fs.utimesSync(path, atime, mtime)
- fs.watch(filename[, options][, listener])
