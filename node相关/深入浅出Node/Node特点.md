# Node特点
- 单线程
- 异步 I/O

## 单线程
- Node 保持了 JS 在浏览器中单线程的特点;
- Node 中, JS 与其余线程无法共享状态;

#### 好处: 
- 不需要像多线程编程那样在意状态的同步问题;
- 没有死锁的存在;
- 没有线程上下文交换所带来的性能开销。

#### 弱点:
- 无法利用多核 CPU;
- 错误会引起整个应用退出, 应用的健壮性值得考验;
- 大量计算占用 CPU 导致无法继续调用异步 I/O。(可以通过子进程弥补)

## 异步 I/O
- 实现并行 I/O;
- 非阻塞 I/O 返回的不是业务需要的数据, 而是当前调用状态;
- 为了获取完整的数据, 应用程序需要重复调用(轮询) I/O 操作来确认是否完成;
- Node 自身的执行模型: 事件循环;
- 事件循环, 观察者, 请求对象, I/O 线程池四者共同构成了 Node 异步 I/O 模型的基本要素。

#### 事件循环
- 进程启动时, Node 会创建一个类似于 while(true) 的循环, 每执行一次循环体的过程称为 Tick;
- 每个 Tick 的过程就是查看是否有事件待处理, 有就取出事件及相关回调函数,  
    如果存在关联的回调函数, 就执行;
- 然后进行下个循环, 如果不再有事件处理, 就退出进程。

#### 观察者
- 根据观察者判断是否有事件需要处理;
- 每个事件循环中有一个或多个观察者, 判断是否有事件需要处理的过程, 就是向观察者查询;
- 浏览器采用了类似的机制, 事件可能产生于用户点击或加载某些文件, 这些产生的事件都有对应的观察者;
- Node 中事件主要来源于网络请求, 文件 I/O 等;
- 每次 Tick 执行中, 事件循环的 I/O 会调用 IOCP 相关方法检查线程池中是否有执行完毕的请求。

#### 请求对象
- 从发起异步函数调用, 到回调函数执行, 过程中存在中间产物: 请求对象;
- JS 调用 Node 核心模块, 调用过程中创建一个请求对象;
- JS 传入的参数, 当前方法, 回调函数都包装在这个请求对象中; 
- 对象包装完成后, 将其推入线程池等待执行;
- JS 调用立即返回, 由 JS 发起的异步调用第一阶段结束, JS 线程继续执行后续操作,  
    当前的 I/O 操作在线程池中等待执行;
- 回调通知是异步 I/O 第二部分,  
    线程池中的 I/O 执行完毕后, 向 IOCP 提交执行状态, 并将线程归还线程池。
- I/O 观察者回调函数的行为是取出请求对象中的 result 属性作为参数, 取出 JS 回调函数作为方法, 调用执行。

### 非 I/O 的异步 API
#### setTimeout, setInterval:
- 实现原理和异步 I/O 类似, 只是不需要 I/O 线程池的参与;
- 调用 setTimeout() 创建的定时器会被插入到定时器观察者内部的一个红黑树中;
- 每次 Tick 执行, 从该红黑树中取出定时器对象, 检查是否超过定时事件;
- 定时器并不精准, 如果某次循环用时较多, 下次循环可能会超时。

#### process.nextTick()
- 将回调函数放入队列, 下一轮 Tick 时取出执行;
- 与 setTimeout(fn, 0) 相比更高效, 因为 setTimeout 需要动用红黑树, 创建定时器对象和迭代等操作。

#### setImmediate
- 同 process.nextTick 类似, 优先级低于 process.nextTick;

## 事件驱动和高性能服务器
- 事件驱动的实质: 通过 主循环 + 事件触发 的方式运行程序;

## Node 服务器优缺点
### 优点:
- 事件驱动, 解决高并发问题
- 单线程避免不必要的内存开销和切换上下文开销。

### 缺点:
- 单线程对多核 CPU 利用率不足;
- 进程的健壮性。
