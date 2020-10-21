#### hystrix 超时配置

- 全局配置 

``` properties
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds = 3000
```

- 接口级别配置

接口定义如下

```java
@FeignClient(value = "user-service", fallbackFactory = UserRemoteClientFallbackFactory.class)
public interface UserRemoteClient {
	
	@GetMapping("/user/get")
	public ResponseData<UserDto> getUser(@RequestParam("id") Long id);
	
}
```

那么配置如下：

```properties
hystrix.command.UserRemoteClient#getUser(Long).execution.isolation.thread.timeoutInMilliseconds = 300
```

- 服务级配置

```properties
hystrix.command.<serviceName>.execution.isolation.thread.timeoutInMilliseconds = 60000
```

#### rebbon 超时配置

- 全局配置

~~~properties
# 请求连接的超时时间
ribbon.ConnectTimeout = 2000
# 请求处理的超时时间
ribbon.ReadTimeout = 5000
~~~

- 服务级配置

~~~properties
# 请求连接的超时时间
<serviceName>.ribbon.ConnectTimeout = 2000
# 请求处理的超时时间
<serviceName>.ribbon.ReadTimeout = 5000
~~~

#### 线程并发配置

- 基本原则

~~~ 
1.线程的执行，是由CPU进行调度的，一个CPU在同一时刻只会执行一个线程，我们看上去的线程A 和 线程B并发执行。为了让用户感觉这些任务正在同时进行，操作系统利用了时间片轮转的方式，CPU给每个任务都服务一定的时间，然后把当前任务的状态保存下来，在加载下一任务的状态后，继续服务下一任务。任务的状态保存及再加载，这段过程就叫做上下文切换。线程的上下文切换也会消耗CPU资源，因此并不是线程数越多越好，线程数越多相应线程的上下文切换消耗的CPU资源越多。

2、CPU密集型：操作内存处理的业务，一般线程数设置为：CPU核数 + 1 或者 CPU核数*2。核数为4的话，一般设置 5 或 8

3、IO密集型：文件操作，网络操作，数据库操作，一般线程设置为：cpu核数 / (1-0.9)，核数为4的话，一般设置 40

 说明：实际值线程数的设置根据这个参考值进行动态调整。
~~~

- hystrix 线程池配置

~~~ properties
# 说明:
# 1. 核心线程数量的线程被占满，之后的任务加入到阻塞队列当中
# 2. 当核心线程数和阻塞队列都被占满，之后的任务到达线程池，线程池则会创建更多的线程，直到存在的线程数量达到最大线程配置的数量
# 3. 当最大线程数量的线程和队列都被占满，之后的任务到达线程池，那么线程池会根据拒绝策略执行相关逻辑

# 核心线程数
hystrix.threadpool.default.coreSize=10
# 最大现场数
hystrix.threadpool.default.maximumSize=15
# 允许线程池大小自动动态调整，设置为true之后，maxSize就生效了，此时如果一开始是coreSize个线程，随着并发量上来，那么就会自动获取新的线程，但是如果线程在keepAliveTimeMinutes内空闲，就会被自动释放掉。
hystrix.threadpool.default.allowMaximumSizeToDivergeFromCoreSize=true
# 任务队列大小
hystrix.threadpool.default.maxQueueSize=1000
~~~

- rbbon

~~~properties
# 最大连接数
ribbon.MaxTotalConnections=500
# 每个host最大连接数
ribbon.MaxConnectionsPerHost=500
~~~



- Tomcat配置

~~~properties
# 最大连接数配置
server.tomcat.max-connections = 1000
# 最大线程数配置
server.tomcat.max-threads = 200

~~~



#### 知识拓展

~~~
根据任务的不同，CPU 的上下文切换就可以分为几个不同的场景，也就是进程上下文切换、线程上下文切换以及中断上下文切换。

Linux 按照特权等级，把进程的运行空间分为内核空间和用户空间。
内核空间（Ring 0）具有最高权限，可以直接访问所有资源；
用户空间（Ring 3）只能访问受限资源，不能直接访问内存等硬件设备，必须通过系统调用陷入到内核中，才能访问这些特权资源。
进程既可以在用户空间运行，又可以在内核空间中运行。进程在用户空间运行时，被称为进程的用户态，而陷入内核空间的时候，被称为进程的内核态。
从用户态到内核态的转变，需要通过系统调用来完成。比如，当我们查看文件内容时，就需要多次系统调用来完成：首先调用 open() 打开文件，然后调用 read() 读取文件内容，并调用 write() 将内容写到标准输出，最后再调用 close() 关闭文件。
而系统调用结束后，CPU 寄存器需要恢复原来保存的用户态，然后再切换到用户空间，继续运行进程。所以，一次系统调用的过程，其实是发生了两次 CPU 上下文切换。

根据 Tsuna 的测试报告，每次上下文切换都需要几十纳秒到数微秒的 CPU 时间。这个时间还是相当可观的，特别是在进程上下文切换次数较多的情况下，很容易导致 CPU 将大量时间耗费在寄存器、内核栈以及虚拟内存等资源的保存和恢复上，进而大大缩短了真正运行进程的时间。
~~~

