[TOC]

## FS 文件系统
> fs是Node中内置核心模块，实现了文件 I/O操作，通过`require("fs")` 导入模块
> 
> fs 中的所有的方法都有异步和同步的形式，在node中能使用异步的就不要同步，因为同步造成主线程阻塞
> 
> 异步方法的最后一个参数都是一个回调函数。 传给回调函数的参数取决于具体方法，但回调函数的第一个参数都会保留给异常。 如果操作成功完成，则第一个参数会是 null 或 undefined。

> 当使用同步方法时，任何异常都会被立即抛出。 可以使用 try/catch 来处理异常，或让异常向上冒泡。

### exists
语法：**`fs.existsSync(path)`**
> 检测文件是否存在， 如果文件存在，则返回 true，否则返回 false
>- path：文件路径
>- 返回值：布尔类型（true/false）

```javascript
var isExist = fs.exists('1.txt');
console.log(isExist);

//--> 异步方法，已经废弃（个人认为判断文件是否存在不需要使用异步）
fs.existsSync('1.txt'); 
```

### readFile
语法：**`fs.readFile(path[, options], callback)`**
> 异步读取文件(`读取的文件必须存在`)，如果未指定字符编码，默认读取的是 Buffer类型，和readFile相对于的还有同步读取`readFileSync`
> - path：文件名或文件描述符
> - options：文件配置项，可以指定字符编码等
> - callback：回掉函数，回调函数有两个参数 (err, data)，其中 data 是文件的内容

```javascript
//-->内置核心模块 file system
let fs = require("fs");

fs.readFile('1.txt','utf-8',function (err, data) {
    if(err) return console.log(err);
    console.log(data);
});
```
### writeFile
语法：**`fs.writeFile(file, data[, options], callback)`**
> 异步地写入数据到文件，可以是`字符串`或者`Buffer`格式，默认字符编码是utf-8，如果是buffer，则忽略字符编码设置。
> 
> 如果文件不存在会创建，如果文件已经存在，则替代文件内容
> - file：文件名或文件描述符
> - data：写入的数据
> - options：文件配置项，可以指定字符编码等
> - callback：回掉函数，回掉函数拥有 err参数，返回错误信息

```javascript
//-->内置核心模块 file system
let fs = require("fs");
//-->同步写入
fs.writeFileSync('./file/1.txt',new Buffer("珠峰"));
//-->异步写入
fs.writeFile('./file/2.txt','啦啦啦',function (err) {
   if(err) console.log(err);
});
```

### 综合练习文件的复制操作
> 结合读写方法实现文件的复制

```
let fs = require('fs');
//-->同步拷贝
function copySync(source, target) {
    let val = fs.readFileSync(source);
    fs.writeFileSync(target,val);
}
copySync('./file/1.txt','./file/2.txt');

//-->异步拷贝
function copy(source, target,callBack) {
    fs.readFile(source,function (err,data) {
        if(err) return callBack && callBack(err);

        fs.writeFile(target,data,callBack);
    })
}
copy('./file/1.txt','./file/3.txt',(err)=>{
    if(err) console.log(err);
});

//-->缺陷：会淹没可用内存，疯狂的向内存中读取，64k以上的文件不建议这样使用，我们希望的是读一点写一点（文件流）
```

### ReadStream 类
> ReadStream是一个可读流，我们可以使用这个类中提供的方法实现对文件的流动读取

#### createReadStream
语法：**`fs.createReadStream(path[, options])`**
> 创建一个可读流对象
> - path：文件路径
> - options：可读流的配置项，如字符编码、读取范围等，另外还可以设置一次读取的文件大小`highWaterMark` 默认值64k
> 

#### data
> 在开启文件流动模式后，node内部会不停的触发data事件，我们可以听过`监听data事件`来读取文件内容

#### end
> 当文件读取完毕后，node内部会触发end方法，我们可以通过`end事件`来处理文件读取完成后的操作

#### error
>文件读取出错时，会触发此事件

#### pause
> pause() 方法可以暂停data事件的触发

#### resume
> resume() 方法可以恢复data事件的触发

-----------------
综合上述方法，实现文件读取
``` javascript
let fs = require('fs');

//-->highWaterMark: 一次读取的文件大小为1k
let re = fs.createReadStream('1.txt',{highWaterMark:1});

let ary = [];
// -->监听‘data’事件
re.on('data',function (chunk) { // chunk 是buffer类型
	//-->将内容添加到数组中
    ary.push(chunk);

    re.pause(); // 暂停data方法的触发
    setTimeout(()=>{
        re.resume(); //恢复data事件的触发
    },1000);
});

//--> 当文件读取完毕后，会触发end事件
re.on('end',function () {
	//-->合并数组中的buffer数据
    console.log(Buffer.concat(ary).toString());
});

```

### WriteStream 类
> WriteStream 是一个可写流，这个类中提供了一系列方法可以实现对文件的流动写入

#### createWriteStream
语法：**`fs.createWriteStream(path[, options])`**
> 创建一个可写流对象，默认写入的编码格式是 utf-8
> -path：文件路径
> -options：可写流的配置项，如字符编码等，另外还可以设置一次写入的文件大小`highWaterMark` 默认值16k

#### write
> `ws.write()` 写入内容，可以支持文件类型`字符串`或`buffer`
>
>返回值：布尔类型（`true`/`false`），true代表可以继续写入，false则不能继续写入；该方法可以接收一个返回值，代表是否可以继续写入文件，当不能写入文件时，应该暂停文件的读取，等待写入完成后再进行读取，例如：如果读取了64k，写的空间16k 往文件中发现写不下，就不要在继续读取了

#### drain
> 当内存中的内容全部写入文件后触发此事件

#### end
> `ws.end() `关闭文件流，当文件全部写入完成后，调用此方法关闭可写流对象。
> 该方法可以支持一个参数，可以写入文件内容，关闭文件流时会再次调用write方法将没有写入的内容强制写入

-------------
综合上述方法实现文件写入
``` javascript
let fs = require('fs');
//-->创建可写流对象ws
let ws = fs.createWriteStream('2.txt',{highWaterMark:1});

// 写入文件内容
let flag = ws.write("啦啦啦啦啦啦，写入文件啦");
//flag： 此返回值表示是否可以继续写入
console.log(flag);

//-->drain: 抽干方法，当内存中的内容全部写入文件后触发此函数
ws.on('drain',function () {
    console.log('drain');
});

//--> 关闭文件流，再次调用write方法写入内容
ws.end('最后一口了');
```

### 文件流实现文件的复制
```
let fs = require('fs');

function copy(source, target) {
    //--> 创建可读流和可写流对象
    let rs = fs.createReadStream(source,{highWaterMark:4});
    let ws = fs.createWriteStream(target,{highWaterMark:1});

    //-->开启流动模式，读取文件
    rs.on('data',function (thunk) {
        if(ws.write(thunk) === false) { //如果写不下就暂停
            rs.pause(); //暂停data事件触发
        }
    });

    // 当给我的内容写完后触发
    ws.on('drain',function () {
        rs.resume(); //开启data读取
    });

    //文件全部读取完毕后，关闭写入流
    rs.on('end',function () {
        ws.end();
    });
}

copy('1.txt','2.txt');
```

### pipe
> 管理数据流，它就相当于一个数据的管道，可以将可读流中的内容导入到可写流中

```javascript
let fs = require('fs');

function copy(source, target) {
    //--> 创建可读流和可写流对象
    let rs = fs.createReadStream(source,{highWaterMark:4});
    let ws = fs.createWriteStream(target,{highWaterMark:1});

    //-->将可读流中的内容导入到可写流中，会默认调用 ws的write 和 end方法
    rs.pipe(ws);
}

copy('1.txt','2.txt');
```
###mkdir
语法：**`fs.mkdir(path[, mode], callback)`**
> 创建目录
> path：目录
> mode：设置目录权限，默认0o777
> callback：回调函数，回调函数中有一个err参数，返回错误信息
```javascript
let fs = require('fs');

fs.mkdir('node',0o777,function (err) {
    if(err) console.log(err);

    console.log('success');
})
```

### readdir
语法：**`fs.readdir(path, callback)`**
> -path：路径
> -callback：回调函数，有两个参数 (err, files)，其中 files 是目录中不包括 '.' 和 '..' 的文件名的数组。

```javascript
fs.readdir('node',function (err, files) {
    console.log(files);
})
```
###stat
语法：**`fs.stat(path, callback)`**
>  检查一个文件是否存在
>  -path：路径
>  -callback：回调函数，有两个参数 (err, stats) 其中 stats 是一个 fs.Stats 对象

```javascript
fs.stat('node',function (err,stat) {
	//-->是否是一个文件
    console.log(stat.isFile());
    //-->是否是一个目录
    console.log(stat.isDirectory());
})
```