## Buffer的创建和转换

**buffer**主要是用来处理**二进制文件流**和**TCP流**的文件缓存区。我们可以将**二进制流**和**string**，**json**，**int**进行转换，也可以进行复制，或者通过自带的函数进行判断buffer的一些状态。

***<u>Buffer类的类方法 :</u>***

* `Buffer.isBuffer(obj)` 判断是否是一个buffer对象
* `Buffer.byteLength(string,[encoding]) ` 判断**string转为buffer的长度**
* `Buffer.concat(list,[totalLength]) ` list是一个数组，将几个buffer合为一个的orgin方法
* `Buffer.isEncoding(encoding)` 判断当前是否是一个有效的编码格式

### 1. 创建Buffer对象

   ***注意：*** **<u>字符串的长度和缓冲区的长度是不一样的，因为字符串是文字为单位，而缓冲区是以字节数为单位</u>**

1. 使用`var buffer = new Buffer(size)`创建对象，然后用`buffer.fill(value,[offset],[end])`来初始化对象。

2. 使用`var buffer = new Buffer(array)`创建对象
3. 使用`var buffer＝ new Buffer(string,[encoding])`创建对象

### 2. Buffer⇦⇨String的相互转换

***注意：***  我们通常会切割或者分开生产buffer，那么一段话就会被切开，这样的话使用**buffer->string**则会生成乱码，所以使用Stringdecoder类的**`decoder.write(buf);`**则会避免这个问题

1. `buf.toString([encoding],[start],[end]);` buffer转换为字符串
2. `buf.write(string,[offset],[length],[encoding]);` buffer将string转换为buf并且写入现有的buffer中（这里wirte实际功能是替换！！）

### 3. Buffer⇦⇨Int的相互转换

1. `buf.readUInt8(offset,[noAssert])` 该函数用来读取第offset位置上的buffer数据，如果noAssert设置为true，则会判断是否offset没有越界，并且抛出异常。
2. `buf.wirteUInt8(value,offset,[noAssert])` 该函数用来替换第offset位置上的值

### 4. Buffer⇦⇨JSON的相互转换

1. `var json = JSON.strify(buf)`  将buf转换为json格式数据
2. `var array = JSON.parse(json)` 将json转化为array数组

### 5.buf1复制到buf2

​	`buf.copy(targetBuffer,[targetstart],[sourcestart],[sourceend]);` 将buffer1复制到bufer2之中

