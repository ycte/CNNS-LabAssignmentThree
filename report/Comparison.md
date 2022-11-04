#### Part 2c: Comparison of Designs

* Netty provides multiple event loop implementations. In a typical server channel setting, two event loop groups are created, with one typically called the boss group and the second worker group. What are they? How does Netty achieve synchronization among them?

```
boss group and worker group:
	两个从类 io.netty.channel.nio.NioEventLoopGroup 创建的 NioEventLoopGroup 对象。NioEventLoopGroup 是用来处理I/O操作的多线程事件循环器，‘boss’ 用来接收进来的连接，‘worker’ 用来处理已经被接收的连接，一旦‘boss’接收到连接，就会把连接信息注册到‘worker’上。
```

```
achieve synchronization:
	NioEventLoop 是NIO框架的 Reactor 线程, 它聚合了一个多路复用器对象，它的 Selector 定义如下图。boss group 和 worker group 里面每个 NioEventLoop 都绑定一个多路复用器，不管是 boss group 同时接受多个连接，还是 worker group 里有多个已经被接受的连接，都由 NioEventLoop 的 Selector 来决定 Group 里面的连接如何映射到已经创建的 Channels 上。从而解决竞争问题，实现 synchronization。
```

![img](https://img-blog.csdnimg.cn/20190430170059413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoZW5jaGFvaGFvMTIzMjE=,size_16,color_FFFFFF,t_70)

参考自：

1. Netty user's guide

![image-20221104194336350](/home/yunxi/.config/Typora/typora-user-images/image-20221104194336350.png)

2. CSDN https://blog.csdn.net/shenchaohao12321/article/details/89713518
3. 知乎 https://zhuanlan.zhihu.com/p/367218937

![image-20221104194448854](/home/yunxi/.config/Typora/typora-user-images/image-20221104194448854.png)



* Method calls such as bind return ChannelFuture. Please describe how one may implement the sync method of a future.

```
	ChannelFuture 实例提供IO操作的结果信息或状态, 即将 Netty 异步计算的状态回调，sync（）使程序阻塞等待任务结束，如果任务失败，将“导致失败的异常”重新抛出来。
	sync() 方法一般用于需要等待异步非阻塞任务结束时，在 Future 和继承自 Future 的类中使用即可。Netty 官方相关使用如下，其中，第一个 sync() 用于等待 bind() 的完成执行，第二个用于等待 NioEventLoop 的正确关闭。
```

```java
 // Bind and start to accept incoming connections.
	ChannelFuture f = b.bind(port).sync(); // (7)

// Wait until the server socket is closed.
// In this example, this does not happen, but you can do that to gracefully
// shut down your server.
	f.channel().closeFuture().sync();
} finally {
	workerGroup.shutdownGracefully();
	bossGroup.shutdownGracefully();
}
```



* Instead of using ByteBuffer, Netty introduces a data structure called ByteBuf. Please give one key difference between ByteBuffer and ByteBuf.

