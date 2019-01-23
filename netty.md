### ByteBuf
  1.有三种缓冲区类型
  heap(堆缓冲区):jvm堆上的缓冲区，将实际数据存放到byte array中来实现，优点：由于数据时存储在jvm的堆中，因此可以快速的创建与快速的释放，并且它提供了直接访问内部字节数组的方法
  ，缺点：每次读写数据时，都需要先将数据复制到直接缓冲区中在进行网络传输
  direct(直接缓冲区):由操作系统管控的缓冲区不占用堆的容量空间,在jvm中通过一个long型的address指向操作系统的这块缓冲区(0拷贝关键)，优点：在使用Socket进行数据传递时，性能非常好，
  因为数据直接位于操作系统的本地内存中，所以不需要从jvm将数据复制到直接缓冲区中，性能很好，缺点：因为Direct Buffer是直接在操作系统内存中的，所以内存空间的分配与释放要比堆空间更加复杂，而且速度要慢一些
  
  Netty通过提供内存池来解决这个问题。直接缓冲区并不支持通过字节数组的方式来访问数据
  
  选择：对于后端的业务消息的编解码来说，推荐使用HeapByteBuf；对于I/O通信线程在读写缓冲区时，推荐使用DirectByteBuf
  
  composite(复合缓存区):是一种虚拟的缓冲区，可以组合多个上边提到的两种缓冲区
  
  JDK的ByteBuffer与Netty的ButeBuf之间的差异对比
 1.Netty的ByteBuf曹勇了读写索引分离的策略(rederIndex与writerIndex),一个初始化(里面尚未有任何数据)的ByteBuf的readerIndex与writerIndex值都为0
 
 2.当读索引与写索引处于同一个位置时，如果我们继续读取，name就会抛出IndexOutOfBoundsException。
 
 3.对于ByteBuf的任何读写操作都会分别单独维护读索引与写索引。maxCapacity最大容量默认的限制就是Integer.MAX_VALUE
 
 JDK的 ByteBuffer的缺点
 
 1.final byte[] hb;这是JDK的ByteBuffer对象中用于存储对象数据的对象声明；可以看到，其字节数组是被声明为final的，也就是长度是对定不变的。一旦分配好后不能动态扩容与收缩；而且当待存储的数据字节很大时就很可能出现IndexOutOfBoundsException。
 如果要预防这个异常那就需要在存储之前完全确定好待存储的字节大小。
 如果ByteBuffer的空间不足，我们只有一种解决方案：创建一个全新的ByteBuffer对象，然后再将之前的ByteBuffer中的数据复制过去，这一切操作都需要由开发者自己来手动完成
 
 2.ByteBuffer只使用一个position指针来表示位置信息，在进行读写切换时就需要调用flip方法或是rewind方法，使使用起来很不方便
 
 Netty的ByteBuf的优点
 
 1.存储字节的数组是动态的，其最大值默认是Integer.MAX_VALUE.这里的动态性是体现在write方法中的，write方法在执行时会判断buffer容量，如果不足则自动扩容
 
 2.ByteBuf的读写索引是完全分开的，使用起来就很方便
