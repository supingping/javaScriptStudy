[TOC]


## Buffer 缓存区
> JS语言本身只有字符串数据类型，没有二进制数据类型，而在处理TCP和文件流的时候，必须要处理二进制数据，
> 
> 在NodeJs中提供了`Buffe`r对象来操作二进制数据，是一个表示`固定内存分配`的全局对象（global.Buffer）
> 
> Buffer好比由一个`八位字节组成的数组`，可以有效的表示二进制数据
> - node 中最小单位是字节，一个字节等于8位二进制的数
> - 一个字节的最大值是11111111，用十进制表示是255（2的8次方 - 1）
> - parsetInt(string, radix) 方法可以将字符串按照指定数字的基数进行转换，当第一个字符串参数的前缀是`0X`或者 `0x` 时，parsetInt会将其解释为16进制，第二个参数可以指定转换的基数，合理的取值范围是2-36，例如： `parsetInt('1011',2)`
> - 将十进制的数字转换为16进制，可以使用toString方法，例如`(255).toString(16)`
> 
> 参考文档：http://nodejs.cn/api/buffer.html


### 定义Buffer

####1.  new Buffer(size) 
> 创建一个指定长度的Buffer
``` javascript
var buff = new Buffer(3);
console.log(buff);
```
#### 2. new Buffer(string[, encoding])
> 创建一个包含指定字符串的Buffer
> - encoding ：可选参数，字符串的编码格式，默认utf-8
``` javascript
var buff = new Buffer("buffer");
console.log(buff);
```
#### 3. new Buffer(array)
> 创建一个指定八位字节数组的Buffer，数组里的数字必须在`0-255`之间
```
var buff = new Buffer([0x16,0x17]);
console.log(buff);
```

### isBuffer
语法：**`Buffer.isBuffer(obj)`** 
> 判断一个对象是否是 Buffer，如果是一个 Buffer 则返回 true ，否则返回 false
> - obj：对象
> - 返回值：布尔类型（true / false） 

```javascript
var buff = new Buffer([0x16,0x17]);
console.log(Buffer.isBuffer(buff));
console.log(Buffer.isBuffer('buff'));
```

### byteLength
语法：**`Buffer.byteLength(string[, encoding])`**
> 获取一个字符串的字节长度，一个汉字是3个字节
> - string：要计算长度的字符串
> - encoding：字符串的编码格式，默认utf-8
> - 返回值： string 包含的字节数

``` javascript
console.log(Buffer.byteLength('啦啦啦')); //一个汉字是3个字节，12
```

### slice
语法：**`buf.slice([start[, end]])`**
> 对Buffer进行截取
> - start：开始位置
> - end：结束位置，默认: buf.length
> - 返回值：新的Buffer
> 
> 负索引值是相对于buf 的末尾

``` javascript
var buff = new Buffer([0x16,0x17]);
buff = buff.slice(0,1);
console.log(buff);

var buf = new Buffer([0x16,0x17,0x18]);
//--> 负索引值是相对于buf的末尾,相当于buf长度+负索引值  <=>buff1.slice(2,3)
buf = buf.slice(-1,3);
console.log(buf);
```

### copy
语法：**`buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])` **
> 拷贝 `buf 的指定位置`数据到`目标but 的指定位置`
>- target 要拷贝进的 Buffer
>- targetStart  target 中开始拷贝进的索引， 默认为0
>- sourceStart  buf 中开始拷贝的索引，当 targetStart 为 undefined 时忽略，默认为 0
>- sourceEnd  buf 中结束拷贝的索引，当  sourceStart 为 undefined 时忽略， 默认为buf.length
>- 返回值:  被拷贝的字节数

``` javascript
var b = new Buffer(12);
var buff1 = new Buffer("珠峰");
var buff2 = new Buffer("培训");

//-->后面两个参数省略不写，默认是全部拷贝
buff1.copy(b,0);
buff2.copy(b,buff1.length,0,buff2.length);
//-->toString可以将buffer转换为字符串
console.log(b.toString());
```

### concat
语法：**`Buffer.concat(list[, totalLength])`**
> 合并Buffer
> - list 要合并的 Buffer
>- totalLength 合并时 list 中 Buffer 实例的总长度
>- 返回值 Buffer

```javascript
var buff1 = new Buffer("珠峰");
var buff2 = new Buffer("培训");

console.log(Buffer.concat([buff1,buff2]).toString());
``` 

> 自己实现concat方法
``` javascript
Buffer.myConcat = function (list, totalLength) {
    // 1.先判断totalLength是否传递，如果传递用传的,没传递自己循环数组算一个总长
    if(typeof totalLength === 'undefined') {
        totalLength = list.reduce((prev,next)=> prev + next.length,0);
    }
    // 2.创建一个大buffer，将每一个小buffer拷贝到大buffer中
    var buffer = new Buffer(totalLength);
    var start = 0;
    list.forEach(buff=>{
        buff.copy(buffer,start);
        start += buff.length;
    });
    //3.截取有效长度 (防止第二个参数传的过大，导致数据有问题，截取到list长度即可)
    return buffer.slice(0,start);
}
var buff1 = new Buffer("珠峰");
var buff2 = new Buffer("培训");
console.log(Buffer.myConcat([buff1,buff2],2000).toString());
```



