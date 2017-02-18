---
title: Node.js 文件系统(11)
date: 2016-06-29 13:35:20
tags: node
categories: nodeJS学习笔记
---


### 11、Node.js 文件系统
Node.js 提供一组类似 UNIX（POSIX）标准的文件操作API。 Node 导入文件系统模块(fs)语法如下所示：
``` javascript
  var fs = require("fs");
  //读取文件内容
  fs.readFile(filename, [options], callback)//异步读取文件内容。
  fs.readFileSync(filename, [options])//同步读取文件内容。
```
建议大家是用异步方法，比起同步，异步方法性能更高，速度更快，而且没有阻塞。
<!-- more -->
#### 打开文件
`fs.open(path, flags, [mode], callback)`//异步打开文件。
    path-文件路径；flag-文件打开行为；mode-设置文件模式(默认:0666-可读可写)；callback - 回调函数，带有两个参数如：callback(err, fd)。
flags参数描述：
1  `r` //以读取模式打开文件。如果文件不存在抛出异常。
2  `r+` //以读写模式打开文件。如果文件不存在抛出异常。
3  `rs` //以同步的方式读取文件。
4  `rs+` //以同步的方式读取和写入文件。
5  `w` //以写入模式打开文件，如果文件不存在则创建。
6  `wx` //类似 'w'，但是如果文件路径存在，则文件写入失败。
7  `w+` //以读写模式打开文件，如果文件不存在则创建。
8  `wx+` //类似 'w+'， 但是如果文件路径存在，则文件读写失败。
9  `a` //以追加模式打开文件，如果文件不存在则创建。
10  `ax` //类似 'a'， 但是如果文件路径存在，则文件追加失败。
11  `a+` //以读取追加模式打开文件，如果文件不存在则创建。
12  `ax+` //类似 'a+'， 但是如果文件路径存在，则文件读取追加失败。

#### 读取文件信息
`fs.stat(path, callback)`//通过异步模式获取文件信息.
    path - 文件路径。callback - 回调函数，带有两个参数如：(err, stats), stats 是 fs.Stats 对象。
fs.stat(path)执行后，会将stats类的实例返回给其回调函数。可以通过stats类中的提供方法判断文件的相关属性。
stats类中方法有：
1  `stats.isFile()`//如果是文件返回 true，否则返回 false。
2  `stats.isDirectory()`//如果是目录返回 true，否则返回 false。
3  `stats.isBlockDevice()`//如果是块设备返回 true，否则返回 false。
4  `stats.isCharacterDevice()`//如果是字符设备返回 true，否则返回 false。
5  `stats.isSymbolicLink()`//如果是软链接返回 true，否则返回 false。
6  `stats.isFIFO()`//如果是FIFO，返回true，否则返回 false。FIFO是UNIX中的一种特殊类型的命令管道。
7  `stats.isSocket()`//如果是 Socket 返回 true，否则返回 false。
#### 写入文件
`fs.writeFile(filename, data, [options], callback)`//异步写入文件
    path-文件路径；data-要写入的数据，可以是String或Buffer(流)对象；options-该参数是一个对象，包含{encoding,mode,flag}默认utf8 ,0666,'w'；callback-回调函数，只包含错误信息参数(err),在写入失败是返回。
#### 读取文件
`fs.read(fd, buffer, offset, length, position, callback)`//异步模式下使用文件描述符来读取文件。
    fd-通过fs.open()方法返回文件描述符；buffer-数据写入的缓冲区；offset-缓冲区写入的写入偏移量；length-要从文件中读取的字节数；position-文件读取的起始位置，值为null则会从当前文件指针位置读取；callback-回调函数，有三个参数err错误信息,bytesRead字节数,buffer缓冲区对象.
#### 关闭文件
`fs.close(fd, callback)`//异步模式下关闭文件,该方法使用了文件描述符来读取文件。
    fd - 通过 fs.open() 方法返回的文件描述符; callback - 回调函数，没有参数。
#### 截取文件
`fs.ftruncate(fd, len, callback)`//异步模式下截取文件,该方法使用了文件描述符来读取文件。
    fd - 通过 fs.open() 方法返回的文件描述符; len - 文件内容截取的长度; callback - 回调函数，没有参数。
#### 删除文件
`fs.unlink(path, callback)`
    path - 文件路径; callback - 回调函数，没有参数。
#### 创建目录
`fs.mkdir(path[, mode], callback)`
    path - 文件路径; mode - 设置目录权限，默认为 0777; callback - 回调函数，没有参数。
#### 读取目录
`fs.readdir(path, callback)`
    path - 文件路径; callback - 回调函数，回调函数带有两个参数err, files，err 为错误信息，files 为 目录下的文件数组列表。
#### 文件模块方法参考手册
1  `fs.rename(oldPath, newPath, callback)`//异步 rename().回调函数没有参数，但可能抛出异常。
2  `fs.ftruncate(fd, len, callback)`//异步 ftruncate().回调函数没有参数，但可能抛出异常。
3  `fs.ftruncateSync(fd, len)`//同步 ftruncate()
4  `fs.truncate(path, len, callback)`//异步 truncate().回调函数没有参数，但可能抛出异常。
5  `fs.truncateSync(path, len)`//同步 truncate()
6  `fs.chown(path, uid, gid, callback)`//异步 chown().回调函数没有参数，但可能抛出异常。
7  `fs.chownSync(path, uid, gid)`//同步 chown()
8  `fs.fchown(fd, uid, gid, callback)`//异步 fchown().回调函数没有参数，但可能抛出异常。
9  `fs.fchownSync(fd, uid, gid)`//同步 fchown()
10  `fs.lchown(path, uid, gid, callback)`//异步 lchown().回调函数没有参数，但可能抛出异常。
11  `fs.lchownSync(path, uid, gid)`//同步 lchown()
12  `fs.chmod(path, mode, callback)`//异步 chmod().回调函数没有参数，但可能抛出异常。
13  `fs.chmodSync(path, mode)`//同步 chmod().
14  `fs.fchmod(fd, mode, callback)`//异步 fchmod().回调函数没有参数，但可能抛出异常。
15  `fs.fchmodSync(fd, mode)`//同步 fchmod().
16  `fs.lchmod(path, mode, callback)`//异步 lchmod().回调函数没有参数，但可能抛出异常。Only available on Mac OS X.
17  `fs.lchmodSync(path, mode)`//同步 lchmod().
18  `fs.stat(path, callback)`//异步 stat(). 回调函数有两个参数 err, stats，stats 是 fs.Stats 对象。
19  `fs.lstat(path, callback)`//异步 lstat(). 回调函数有两个参数 err, stats，stats 是 fs.Stats 对象。
20  `fs.fstat(fd, callback)`//异步 fstat(). 回调函数有两个参数 err, stats，stats 是 fs.Stats 对象。
21  `fs.statSync(path)`//同步 stat(). 返回 fs.Stats 的实例。
22  `fs.lstatSync(path)`//同步 lstat(). 返回 fs.Stats 的实例。
23  `fs.fstatSync(fd)`//同步 fstat(). 返回 fs.Stats 的实例。
24  `fs.link(srcpath, dstpath, callback)`//异步 link().回调函数没有参数，但可能抛出异常。
25  `fs.linkSync(srcpath, dstpath)`//同步 link().
26  `fs.symlink(srcpath, dstpath[, type], callback)`//异步 symlink().回调函数没有参数，但可能抛出异常。 type 参数可以设置为 'dir', 'file', 或 'junction' (默认为 'file') 。
27  `fs.symlinkSync(srcpath, dstpath[, type])`//同步 symlink().
28  `fs.readlink(path, callback)`//异步 readlink(). 回调函数有两个参数 err, linkString。
29  `fs.realpath(path[, cache], callback)`//异步 realpath(). 回调函数有两个参数 err, resolvedPath。
30  `fs.realpathSync(path[, cache])`//同步 realpath()。返回绝对路径。
31  `fs.unlink(path, callback)`//异步 unlink().回调函数没有参数，但可能抛出异常。
32  `fs.unlinkSync(path)`//同步 unlink().
33  `fs.rmdir(path, callback)`//异步 rmdir().回调函数没有参数，但可能抛出异常。
34  `fs.rmdirSync(path)`//同步 rmdir().
35  `fs.mkdir(path[, mode], callback)`//S异步 mkdir(2).回调函数没有参数，但可能抛出异常。 mode defaults to 0777.
36  `fs.mkdirSync(path[, mode])`//同步 mkdir().
37  `fs.readdir(path, callback)`//异步 readdir(3). 读取目录的内容。
38  `fs.readdirSync(path)`//同步 readdir().返回文件数组列表。
39  `fs.close(fd, callback)`//异步 close().回调函数没有参数，但可能抛出异常。
40  `fs.closeSync(fd)`//同步 close().
41  `fs.open(path, flags[, mode], callback)`//异步打开文件。
42  `fs.openSync(path, flags[, mode])`//同步 version of fs.open().
43  `fs.utimes(path, atime, mtime, callback)
44  `fs.utimesSync(path, atime, mtime)`//修改文件时间戳，文件通过指定的文件路径。
45  `fs.futimes(fd, atime, mtime, callback)
46  `fs.futimesSync(fd, atime, mtime)`//修改文件时间戳，通过文件描述符指定。
47  `fs.fsync(fd, callback)`//异步 fsync.回调函数没有参数，但可能抛出异常。
48  `fs.fsyncSync(fd)`//同步 fsync.
49  `fs.write(fd, buffer, offset, length[, position], callback)`//将缓冲区内容写入到通过文件描述符指定的文件。
50  `fs.write(fd, data[, position[, encoding]], callback)`//通过文件描述符 fd 写入文件内容。
51  `fs.writeSync(fd, buffer, offset, length[, position])`//同步版的 fs.write()。
52  `fs.writeSync(fd, data[, position[, encoding]])`//同步版的 fs.write().
53  `fs.read(fd, buffer, offset, length, position, callback)`//通过文件描述符 fd 读取文件内容。
54  `fs.readSync(fd, buffer, offset, length, position)`//同步版的 fs.read.
55  `fs.readFile(filename[, options], callback)`//异步读取文件内容。
56  `fs.readFileSync(filename[, options])
57  `fs.writeFile(filename, data[, options], callback)
异步写入`文件内容。
58  `fs.writeFileSync(filename, data[, options])`//同步版的 fs.writeFile。
59  `fs.appendFile(filename, data[, options], callback)`//异步追加文件内容。
60  `fs.appendFileSync(filename, data[, options])`//The 同步 version of fs.appendFile.
61  `fs.watchFile(filename[, options], listener)`//查看文件的修改。
62  `fs.unwatchFile(filename[, listener])`//停止查看 filename 的修改。
63  `fs.watch(filename[, options][, listener])`//查看 filename 的修改，filename 可以是文件或目录。返回 fs.FSWatcher 对象。
64  `fs.exists(path, callback)`//检测给定的路径是否存在。
65  `fs.existsSync(path)`//同步版的 fs.exists.
66  `fs.access(path[, mode], callback)`//测试指定路径用户权限。
67  `fs.accessSync(path[, mode])`//同步版的 fs.access。
68  `fs.createReadStream(path[, options])`//返回ReadStream 对象。
69  `fs.createWriteStream(path[, options])`//返回 WriteStream 对象。
70  `fs.symlink(srcpath, dstpath[, type], callback)`//异步 symlink().回调函数没有参数，但可能抛出异常。

