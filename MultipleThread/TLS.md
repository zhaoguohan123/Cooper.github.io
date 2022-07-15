## 1. TLS概念
**线程局部存储（Thread Local Storage，TLS）用来将数据与一个正在执行的指定线程关联起来**。

## 2. TLS应用场景
如果需要在一个线程内部的各个函数调用都能访问、但其它线程不能访问的变量（被称为static memory local to a thread 线程局部静态变量），就需要新的机制来实现，这就是TLS。当多个线程同时访问以及修改同一全局变量或者静态变量锁导致的冲突。
## 3. TLS原理
### 3.1 动态TSL
系统中每个进程都有一组正在使用标志（in-use flags），每个标志可以被设为FREE或INUSE，表示该slot是否正在被使用。

进程中的线程是通过使用一个数组来保存与线程相关联的数据的，这个数组由TLS_MINIMUM_AVAILABLE个slot组成，在WINNT.H文件中该值被定义为64个。也就是说当线程创建时，系统给每一个线程分配了一个数组，这个数组共有TLS_MINIMUM_AVAILABLE个slot，并且将这个数组的各个slot初始化为0，之后系统把这个数组与新创建的线程关联起来。每一个线程中都有它自己的数组，数组中的每一个slot都能保存一个32位的值。在使用这个数组前首先要判定，数组中哪个slot可以使用，这将使用函数TlsAlloc来判断。函数TlsAlloc判断数组中一个slot可用后，就把这个slot分配给调用的线程，并保留给调用线程。要为数组中的某个slot赋值可以使用函数TlsSetValue，要得到某个slot的值可以使用TlsGetValue。

## 4. TLS应用
### 4.1 windows
下面的四个函数就是对TLS进行操作的：  

- TlsAlloc  
当你需要使用一个TLS slot 的时候，你就可以用这个函数将相应的TLS slot位置１。  

- TlsSetValue  
TlsSetValue 可以把数据放入先前配置到的TLS slot 中。两个参数分别是TLS slot 索引值以及欲写入的数据内容。TlsSetValue 就把你指定的数据放入64 DWORDs 所组成的数组（位于目前的thread database）的适当位置中。  

- TlsGetValue 
这个函数几乎是TlsSetValue 的一面镜子，最大的差异是它取出数据而非设定数据。和TlsSetValue 一样，这个函数也是先检查TLS 索引值合法与否。如果是，TlsGetValue 就使用这个索引值找到64 DWORDs 数组（位于thread database 中）的对应数据项，并将其内容传回。  

- TlsFree
TlsFree 先检验你交给它的索引值是否的确被配置过。如果是，它将对应的64 位TLS slots 位关闭。然后，为了避免那个已经不再合法的内容被使用，TlsFree 巡访进程中的每一个线程，把0 放到刚刚被释放的那个TLS slot 上头。于是呢，如果有某个TLS 索引后来又被重新配置，所有用到该索引的线程就保证会取回一个0 值，除非它们再调用TlsSetValue。

注意：TlsSetValue/TlsGetValue在MSDN文档中指出，index这个参数可以忽略，微软并没有做错误校验，这两个函数操作的数据与调用他们的线程是对应的

windows的代码链接:
https://github.com/zhaoguohan123/Blog/tree/master/TLS/windows


