# 深入浅出Node.js学习笔记（六）

## 理解Buffer

## 1. Buffer结构

Buffer是一个像Array的对象，但它主要用于操作字节。

### 1.1 模块结构

Buffer是一个典型的JavaScript与C++结合的模块，它将性能相关的部分用C++实现，将非性能相关的部分用JavaScript实现。

Buffer所占用的内存不是通过V8分配的，属于堆外内存。

由于Buffer太过常见，Node在进程启动时就已经加装了它，并将其放在全局对象(global)上。

### 1.2 Buffer对象

Buffer对象类似于数组，它的元素为16进制的两位数，即0到255的数值。

```
var str = "深入浅出node.js";
var buf = new Buffer(str, 'utf-8');
console.log(buf);
// => <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
```

### 1.3 Buffer内存分配

Buffer对象的内存分配不是在V8的堆内存中，而是在Node的C++层面实现内存的申请的。因为 处理大量的字节数据不能采用需要一点内存就想操作系统申请一旦内存的方式，这可能造成大量的内存申请的系统调用，对操作系统有一定的压力。为此Node在内存的使用上应用的是在C++层面申请内存、在JavaScript中分配内存的策略。

为了高效地使用申请来的内存，Node采用了**slab**分配机制。

slab是一种动态内存管理机制。

slab是一块申请好的固定大小的内存区域。

slab的3中状态：

1. full:完全分配状态；
2. partial:部分分配状态；
3. empty:没有被分配状态；

new一个指定大小的Buffer对象：

```
new Buffer(size)
```

Node以8KB为界限来区分Buffer是大对象还是小对象。

8KB的值也就是每个slab的大小值，在JavaScript层面，以它作为单位单元进行内存分配。

1. **分配小Buffer对象**

   如果指定Buffer的大小少于8KB，Node会按照小对象的方式进行分配。Buffer的分配过程中主要使用一个局部变量作为中间处理对象，处于分配状态的slab单元都指向它。

2. **分配大buffer对象**

   如果需要超过8KB的Buffer对象，将会直接分配一个SlowBuffer对象作为slab作为slab单元，这个slab单元将会被这个大Buffer对象独占。

3. **小结**

   真正的内存是在Node的C++层面提供的，JavaScript层数只是使用它。当进行小而频繁的Buffer操作时，采用slab的机制进行预先申请和事后分配，使得JavaScript到操作系统之间不必有过多的内存申请方面的系统调用。对于大Buffer而言，则直接使用C++层面提供的内存，而无需细腻的分配操作。

##  2. Buffer的转换

Buffer对象可以也字符串之间相互转换。支持的字符串编码有：

1. ASCII
2. UTF-8
3. UTF-16LE/UCS-2
4. Base64
5. Binary
6. Hex

### 2.1 字符串转Buffer

字符串转Buffer对象主要通过构造函数完成的：

```
new Buffer(str,[encoding]);
```

### 2.2 Buffer转字符串

Buffer对象的toString()可以将Buffer对象转换为字符串。

```
buf.toString([encoding],[start],[end]);
```

### 2.3 Buffer不支持的编码类型

Buffer提供了一个isEncoding()函数来判断编码是否支持转换。

```
Buffer.isEncoding(encodinh)
```

##  3. Buffer的拼接

Buffer在使用场景中，通常是以一段一段的方式传输。

### 3.1 乱码是如何产生的

对于任意长度的Buffer而言，宽字节字符串都有可能存在被截取的情况，这是导致乱码产生的原因。

### 3.2 setEncoding()与string_decoder()

setEncoding()：设置编码的方法，让data事件中传递的不再是一个Buffer对象，而是编码后的字符串。

string_decoder模块是StringDecoder的实例对象。

### 3.3 正确拼接Buffer

正确的拼接方式是用一个数组存储接收到的所有Buffer片段的总长度，然后调用Buffer.concat()方法生成一个合并的Buffer对象。Buffer。concat()方法封装了从小Buffer对象向大Buffer对象的复制过程。

##  4. Buffer与性能

通过预先转换静态内容为Buffer对象，可以有效地减少CPU的重复使用，节省服务器资源。在Node构建的Web应用中，可以选择将页面的动态内容和静态内容分离，静态内容部分可以通过预先转换为Buffer的方式，使得性能得到提升。

- 文件读取

  Buffer的使用在文件读取时，有一个highWaterMark设置对性能的影响至关重要。

  读取一个相同的大文件时，highWaterMark值的大小与读取速度的关系：该值越大，读取速度越快。