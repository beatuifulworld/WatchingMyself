# Reading



- 无符号加减法需要从补码相加的角度去看待；
- 无符号不能有符号相加，golang不支持隐式类型转换；
- 选择不同的类型的变量时, 要特别注意该类型的取值范围, 防止数值越界；
- const类型默认为int、float64、string，但是可以给无符号类型变量赋值，即打破了不能做隐式类型转换的规定，所以在做同族类型之间的运算时容易出错；
- 推荐使用int类型，默认为int32，数值范围为[-2147483648,2147483647]。



- 软件原子操作离不开硬件指令的支持；
- CAS（Compare And Swap）即为原子操作，可用于在多线程编程中实现不被打断的数据交换操作，从而**避免多线程同时改写某一数据时由于执行顺序不确定性以及中断的不可预知性产生的数据不一致问题**。
- Golang中Mutex互斥锁就是利用CAS原子操作实现；
- LOCK是一个指令前缀，其后必须跟一条“**读-改-写**”性质的指令，它们可以是ADD, ADC, AND, BTC, BTR, BTS, CMPXCHG, CMPXCH8B, CMPXCHG16B, DEC, INC, NEG, NOT, OR, SBB, SUB, XOR, XADD,  XCHG。该指令是一种锁定协议，用于封锁**总线**，禁止其他 CPU 对内存的操作来保证原子性。在汇编代码里给指令加上 LOCK 前缀，这是CPU 在**硬件层面支持的原子操作**。但这样的锁**粒度太粗**，其他无关的内存操作也会被阻塞，大幅**降低系统性能**，核数越多愈发显著。为了提高性能，Intel 从 Pentium 486 开始引入了粒度较细的**缓存锁：MESI协议**。
- 在Go提供的原子操作库atomic中，除了CAS还有许多有用的原子方法，**它们共同筑起了Go同步原语体系的基石**。





- CPU从主存中读取数据至Cache时，并非单个字节形式进行读取，而是以连续内存块的方式进行拷贝，拷贝块内存的单元被称为缓存行（Cache Line）。这样做的理论依据是著名的**局部性原理**。

    - 时间局部性（temporal locality）：如果一个信息项正在被访问，那么在近期它很可能还会被再次访问。
    - 空间局部性（spatial locality）：在最近的将来将用到的信息很可能与现在正在使用的信息在空间地址上是临近的。

- 利用CPU缓存特性能够潜在帮助设计更高效的算法，尽量避免与主存交互。

- MESI缓存锁协议基于总线嗅探的写传播机制：

    - Modified（被修改的）：处于这一状态的数据只在本核处理器中有缓存，且其数据已被修改，但还没有更新到内存中。
    - Exclusive（独占的）：处于这一状态的数据只在本核处理器中有缓存，且其数据没有被修改，与内存一致。
    - Shared（共享的）：处于这一状态的数据在多核处理器中都有缓存。
    - Invalid（无效的）：本CPU中的这份缓存已经无效了。

  但在多线程同时读写同一个cacheline的不同变量会导致缓存失效，即造成伪共享的状态。规避伪共享可以利用内存填充（空间换时间）的方式，保证两个变量之间填充足够多的空间，以保证它们属于不同的缓存行。



[《Go 语言并发之道》读后感 - 第二章](https://mp.weixin.qq.com/s/cmCyYBhwA0g1ZWxsuwdUtw)

- 并发与并行的区别：并发属于代码，并行属于一个运行中的程序。

- ![image-20201224110829357](http://img.zhengyua.cn/img/20201224110829.png)

  并发 = 两个队列一个咖啡机，两个队列的人交替使用咖啡机，可能会遇见如下情况：

  A 队列被 B 队列小伙伴插队了，这就是**条件竞争**。

  A 队列有一位英俊小伙和 B 队列一位女神并排，轮到他们俩的时候两人互看对方一眼，互让一步同时彬彬有礼的说了一句：你先。这个时候**活锁**出现了。

  B 队列一位小伙伴，是代替其他朋友来打咖啡，他带了一箱子杯子，终于轮到他了。这时**饥饿**就出现了。

  A 队列一位小伙伴喜欢摩卡，卡布奇诺混合咖啡，B 队列一位小伙伴喜欢拿铁，摩卡混合咖啡。当他们互相等待对方摩卡出杯的时候后，**死锁**来了。

  并行 = 两个队列两台咖啡机。

- 编写正确的并发逻辑越难，越需要我们将很简单的并发原语组合起来使用。如果你想写并发代码，你需要对你的程序按照线程同步以及内存访问同步来建模。如果你有一大堆需要并发建模的东西，而你的计算机又不能处理那么多的线程，就需要创建一个线程池，并将你的操作在线程池中复用。

- CSP：进程微积分。

  一个进程的输出应该直接流量另一个进程的输入。

  一个由守护的命令仅仅是一个带有左和右倾向的语句，由 → 来分隔。左侧服务是有运行条件的，或者是守护右侧服务，如果左侧服务运行失败，或者在一个命令执行后，返回 false 或者退出，右侧服务永远不会被执行。

- ![image-20201224110813417](http://img.zhengyua.cn/img/20201224110813.png)

  **你是否需要转让数据所有权？**

  如果需要将计算结果共享，则需要使用channel，好处如下：

    - 创建一个带缓存channel来实现低成本的在内存中的队列来解耦你的生产者和消费者
    - channel确保你的并发代码可以和其他并发代码进行组合

  **你是否试图在保护某个数据结构的内部状态？**

  当你需要保护某一个数据结构的时候这是原子操作，临界区进入最小状态，我们需要将这个行为锁起来，所以这时使用 sync 包。

  **你是否试图协调多个逻辑片段？**

  channel 比内存同步原语更具有可组合性。

  Go 团队鼓励漫天飞的 channel ，如果满篇锁对于任何语言可能都是灾难。

  **这是一个性能要求很高的临界区吗？**

  channel 使用做内存访问同步来操作，因此它只能更慢。

  追求简洁，尽量使用 channel ，并且认为 goroutine 的使用是没有成本的。





- Go中的gignal能够接收系统中的信号机制，通过channel且借助两个协程（一个用来处理信号，一个用来循环接收信号）来实现。



- panic取代err!=nil的错误处理方式



[Go: 关于锁的1234](https://studygolang.com/articles/30028)

- 锁的实现原理是原子操作，CPU引入硬件支持的原子操作，例如x86体系下的LOCK信号，通过锁定总线，禁止其他CPU对内存的操作来保证原子性。但这种锁粒度太粗，容易阻塞其他操作且大幅度降低系统性能，后面Intel再Pentium486开始引入用于保证缓存一致性的MESI协议，通过锁定cacheline达到缓存锁的原子操作。

- 真正的CAS是硬件级别的指令支持，由于不用锁定总线即这样的原子操作指令不会限制其余CPUCore操作非锁定内存，虽然不会影响整体系统的吞吐量，但是由于原子操作指令仍然需要在 CPU 之间传递消息用于对 cache line 的锁定，其**性能仍有一定损耗**，具体来说大概就相当于一个未命中 cache 的 Load Memory 指令。

- Go中atomic.AddInt32的实现是直接使用汇编LOCK_XADDL完成而不是CAS和循环。

- **自旋锁**通过反复检测锁变量是否可用来完成加锁。在加锁过程中 CPU 是在忙等待，因此**仅适用于阻塞较短时间的场合**；其优势在于**避免了线程切换的开销**。

- 研究人员发现，如果锁冲突比较频繁，在 CAS 失败时使用**指数退避算法**（Exponential Backoff）往往能得到更好的整体性能。

- Mutex为互斥锁，即未获得锁时会释放对CPU的占用，其底层也是原子操作来实现，Mutex针对实际应用场景做了许多优化，是一个从轻量级锁逐渐升级到重量级锁的过程，从而平衡各种场景下的需求和性能：

    - fastpath：在简单场景下直接使用 CAS 加锁和解锁，缩短执行路径；
    - spin：当自旋有意义时（多核、GOMAXPROCS > 1 、尝试不超过4次），优先使用自旋；
    - **饥饿** & **公平**：当等待超过 1ms 时，进入饥饿模式，新竞争者需要排队；

  这里提到的“**公平**”，指的是先到先得，这意味着每一个竞争者都需要进入等待队列，而这意味着**CPU控制权的切换和对应的开销**；而**非公平**锁，指的是在进入等待队列之前先尝试加锁，如果加锁成功，可以**减少排队从而提高性能**，但代价是队列中的竞争者可能会处于“**饥饿**”状态。




[Golang 实现 Redis(1): Golang 编写 Tcp 服务器](https://www.cnblogs.com/Finley/p/11070669.html)

- 结合goroutine优化多路复用模型

- 对于解决粘包拆包问题的方案策略
    - 定长消息；
    - 在消息尾部添加特殊分隔符，如示例中的Echo协议和FTP控制协议。bufio 标准库会缓存收到的数据直到遇到分隔符才会返回，它可以帮助我们正确地分割字节流；
    - 将消息分为 header 和 body, 并在 header 中提供 body 总长度，这种分包方式被称为 LTV(length，type，value) 包。这是应用最广泛的策略，如HTTP协议。当从 header 中获得 body 长度后, io.ReadFull 函数会读取指定长度字节流，从而解析应用层消息。
- TCP 服务器的优雅关闭模式通常为: 先关闭listener阻止新连接进入，然后遍历所有连接逐个进行关闭。



[Golang 实现 Redis(2): 实现 Redis 协议解析器](https://www.cnblogs.com/Finley/p/11923168.html)

- Redis 自 2.0 版本起使用了统一的协议 RESP (REdis Serialization Protocol)，该协议易于实现，计算机可以高效的进行解析且易于被人类读懂。

  RESP 是一个二进制安全的文本协议，工作于 TCP 协议上。客户端和服务器发送的命令或数据一律以 `\r\n` （CRLF）结尾。

  RESP 定义了5种格式：

    - 简单字符串(Simple String): 服务器用来返回简单的结果，比如"OK"。非二进制安全，且不允许换行。
    - 错误信息(Error): 服务器用来返回简单的结果，比如"ERR Invalid Synatx"。非二进制安全，且不允许换行。
    - 整数(Integer): `llen`、`scard`等命令的返回值, 64位有符号整数
    - 字符串(Bulk String): 二进制安全字符串, `get` 等命令的返回值
    - 数组(Array, 旧版文档中称 Multi Bulk Strings): Bulk String 数组，客户端发送指令以及`lrange`等命令响应的格式

  RESP 通过第一个字符来表示格式：

    - 简单字符串：以"+" 开始， 如："+OK\r\n"
    - 错误：以"-" 开始，如："-ERR Invalid Synatx\r\n"
    - 整数：以":"开始，如：":1\r\n"
    - 字符串：以 `$` 开始
    - 数组：以 `*` 开始



[Golang 实现 Redis(3): 实现内存数据库](https://www.cnblogs.com/Finley/p/12590718.html)

- Concurrent Hash Map即并发哈希表，保证并发安全，常见的设计有如下几种：

    - sync.map：golang官方提供的并发哈希表，性能优秀但结构复杂不便于拓展；
    - juc.ConcurrentHashMap: java 的并发哈希表采用分段锁实现。在进行扩容时访问哈希表线程都将协助进行 rehash 操作，在 rehash 结束前所有的读写操作都会阻塞。因为缓存数据库中键值对数量巨大且对读写操作响应时间要求较高，使用juc的策略是不合适的。
    - memcached hashtable: 在后台线程进行 rehash 操作时，主线程会判断要访问的哈希槽是否已被 rehash 从而决定操作 old_hashtable 还是操作 primary_hashtable。
      这种策略使主线程和rehash线程之间的竞争限制在哈希槽内，最小化rehash操作对读写操作的影响，这是最理想的实现方式。

- LockMap即需要保证锁定一个或者一组key的锁，最直接的做法就是锁过程分为两步: 初始化对应的锁 -> 加锁， 解锁过程也分为两步: 解锁 -> 释放对应的锁。但这种做法容易出现并发安全问题。若要避免这种情况就需要解锁时不释放锁，但容易造成使用过的锁无法释放造成内存泄漏问题。

  注意到哈希表的长度远少于可能的键的数量，反过来说多个键可以共用一个哈希槽。若我们不为单个键加锁而是为它所在的哈希槽加锁，因为哈希槽的数量非常少即使不释放锁也不会占用太多内存。

- Time To Live (TTL) 的实现方式非常简单，其核心是 string -> time 哈希表。当访问某个 key 时会检查是否过期，并删除过期key。



[Golang 实现 Redis(4): AOF 持久化与AOF重写](https://www.cnblogs.com/Finley/p/12663636.html)

- AOF 持久化是典型的异步任务，主协程(goroutine) 可以使用 channel 将数据发送到异步协程由异步协程执行持久化操作。

  在进行持久化时需要注意两个细节:

    1. get 之类的读命令并不需要进行持久化
    2. expire 命令要用等效的 expireat 命令替换。举例说明，10:00 执行 `expire a 3600` 表示键 a 在 11:00 过期，在 10:30 载入AOF文件时执行 `expire a 3600` 就成了 11:30 过期与原数据不符。

- 若我们对键a赋值100次会在AOF文件中产生100条指令但只有最后一条指令是有效的，为了减少持久化文件的大小需要进行AOF重写以删除无用的指令。

  重写必须在固定不变的数据集上进行，不能直接使用内存中的数据。Redis 重写的实现方式是进行 fork 并在子进程中遍历数据库内的数据重新生成AOF文件。由于 golang 不支持 fork 操作，我们只能采用读取AOF文件生成副本的方式来代替fork。

  在进行AOF重写操作时需要满足两个要求:

    1. 若 AOF 重写失败或被中断，AOF 文件需保持重写之前的状态不能丢失数据
    2. 进行 AOF 重写期间执行的命令必须保存到新的AOF文件中, 不能丢失

  因此我们设计了一套比较复杂的流程：

    1. 暂停AOF写入 -> 更改状态为重写中 -> 准备重写 -> 恢复AOF写入
    2. 在重写过程中，持久化协程在将命令写入文件的同时也将其写入内存中的重写缓存区
    3. 重写协程读取 AOF 文件中的前一部分（重写开始前的数据，不包括读写过程中写入的数据）并重写到临时文件（tmp.aof）中
    4. 暂停AOF写入 -> 将重写缓冲区中的命令写入tmp.aof -> 使用临时文件tmp.aof覆盖AOF文件（使用文件系统的mv命令保证安全）-> 清空重写缓冲区 -> 恢复AOF写入



[Golang 实现 Redis(5): 使用跳表实现 SortedSet](https://www.cnblogs.com/Finley/p/12854599.html)

- ![image-20201228111736146](http://img.zhengyua.cn/img/20201228111744.png)



[Golang最细节篇— struct{} 空结构体究竟是啥？](https://mp.weixin.qq.com/s/Rd1kUFK0F4Z-UTfl5wBcPw)

- 空结构体的变量的内存地址都是一样的，golang 使用 `mallocgc` 分配内存的时候，如果 size 为 0 的时候，统一返回的都是全局变量 `zerobase` 的地址。
- receiver 本质上是非常简单的一个通用思路，就是把对象值或地址作为第一参数传入函数；
- 函数参数压栈方式从前往后（可以调试看下）；
- 对象值作为 receiver 的时候，涉及到一次值拷贝；
- golang 对于值做 receiver 的函数定义，会根据现实需要情况可能会生成了两个函数，一个值版本，一个指针版本（思考：什么是“需要情况”？就是有 `interface` 的场景 ）；
- 空结构体在编译期间就能识别出来的场景，编译器会对既定的事实，可以做特殊的代码生成；
- 空结构体也是结构体，只是 size 为 0 的类型而已；
- 所有的空结构体都有一个共同的地址：`zerobase` 的地址；
- 空结构体可以作为 receiver ，receiver 是空结构体作为值的时候，编译器其实直接忽略了第一个参数的传递，编译器在编译期间就能确认生成对应的代码；
- `map` 和 `struct{}` 结合使用常常用来节省一点点内存，使用的场景一般用来判断 key 存在于 `map`；
- `chan` 和 `struct{}` 结合使用是一般用于信号同步的场景，用意并不是节省内存，而是我们真的并不关心 chan 元素的值；
- `slice` 和 `struct{}` 结合好像真的没啥用。



[《Go 语言并发之道》读后感 - 第一章](https://talkgo.org/t/topic/776)

- 并发代码问题
- 竞争条件：当两个或多个操作必须按正确的顺序执行，而程序并未保证这个顺序，就会发生竞争。
- 原子性：不可分割和不可中断的，需要注意其上下文，操作的原子性可以根据当前定义的范围而改变。
- 内存访问同步：两个并发进程视图访问相同的内存区域，他们访问内存的方式不是原子的，就会出现竞争，通过锁住其临界区就可以保证内存访问同步。
- 死锁、活锁、饥饿。
- 确定并发安全，作者希望每一个负责并发的团队，或人，把每一个并发函数，接口(类)，注释清楚。
    - 谁负责并发？
    - 如何利用并发原语解决这个问题？
    - 谁负责同步？



[《 Go 语言并发之道》读后感 - 第三章](https://talkgo.org/t/topic/1065)

- Go遵循fork-join的并发模型，fork指程序在任意节点，可以将子节点于父节点同时运行；join将来在某个节点时，分支将会合并在一起。

  ![image-20201229162226649](http://img.zhengyua.cn/img/20201229162226.png)

- 上下文切换，线程需要花费 2s 左右的时间，goroutine 上下文切换只需要 0.002s。

- 内存方面goroutine需要占用几kb。

- WaitGroup：当你不关心并发操作的结果，或者你有其他方法来收集它们的结果时，WaitGroup 是等待一组并发操作完成的好方法。

- 互斥锁和读写锁：Mutex 是 “互斥” 的意思，是保护程序中临界区的以重方式。它提供了一种安全的方式来表示对这些共享资源的独占访问。Mutex 互斥锁，对临界区强限制，goroutine 必须先获得锁然后再进行临界区操作。读写锁主要是限制其他 goroutine 写，但不限制读。

- Cond：一个 goroutine 的集合点，等待或发布一个 event。

- sync 包为我们提供了一个专门的方案解决一次性初始化的问题： sync.One。

- Pool 池 是并发安全实现。用于约束创建昂贵的场景，例如： 链接 Redis，MySQL，或其他调用远端服务的时候。只创建固定数量的实例，保障对端服务可用。

  当你使用 Pool 工作是，记住以下几点：

    - 当实例化 sync.Pool，使用 new 方法创建一个成员变量，在调用时时线程安全的。
    - 当你收到一个来自 Get 的实例时，不要对所接收的对象的状态做出任何假设。
    - 当你用完一个从 Pool 中取出来的对象时，一定要调用 Put,否则，Pool 就无法复用这个实例了。通常情况下，这是用 defer 完成的。
    - Pool 内的分布必须大致均匀。

- channel：goroutine 是被动调度的，没有办法保证它会在程序退出之前运行。Go 语言中的 channel 是阻塞的，这样在不同的 goroutine 操作同一个 channel 的时候就会被 channel 阻塞，我们还需要注意，不要试图从一个空 channel 中读取数据，如果只读取将会触发死锁，读数据的 goroutine 将等待至少一条数据被写入 channel 后才行。

  个人对于缓冲 channel 的一些看法

    - 当生产者速度远大于消费者速度，创建缓冲 channel 是一种正向优化
    - 当消费者具有阻塞性质或 syscall 时（例如：数据写入磁盘，请求外部接口，远端服务）
    - 当消费者速度大于生产者速度，消费者侧无阻塞性质，设置缓冲 channel 可能是一种负优化

  对于只读只写 channel 的一些个人经验：

  我们的函数往往是一层一层的调用的，当我们需要使用 channel 构建并发的时候，我们需要知道当前操作的函数对需要操作的 channel 是生产者，或消费者。这样构建时就可以防止一些死锁，channel 未关闭的问题。这是我个人的使用经验。

    - channel 的输入向都需要一个 goroutine.
    - 在函数内部定义 channel 返回一个只读 channel ，有效的管理了临界区
    - 全局定义的channel ，在传入函数时转换了性质，防止在一个 goroutine 种对同一 channel 既读又写。

  channel状态机：

  ![image-20201229164644030](http://img.zhengyua.cn/img/20201229164644.png)

  从 channel 的所有者说起。当一个 goroutine 拥有一个 channel 时应该：

    1. 初始化该 channel
    2. 执行写入操作，或将所有权交给另一个 goroutine
    3. 关闭该通道
    4. 将此前列入的三件事封装在一个列表中，并通过订阅 channel 将其公开

  通过将这些责任分配给 channel 的所有者，会发生一些事情：

    - 因为我们是初始化 channel 的人，所以我们要了解写入空 channel 会带来死锁的风险
    - 因为我们是初始化 channel 的人，所以我们要了解关闭空 channel 会带来 panic 的风险
    - 因为我们是决定 channel 何时关闭的人，所以我们要了解写入已关闭的 channel 会带来 panic 的风险
    - 因为我们是决定何时关闭 channel 的人，所以我们要了解多次关闭 channel 会带来 panic 的风险
    - 我们在编译时使用类型检查器来防止对 channel 进行不正确的写入

  作为一个消费者，需要只需要担心两件事：

    - channel 什么时候会被关闭
    - 处理基于任何原因出现的阻塞

- Select：channel 将 goroutine 粘合在一起，让我们构建起一条非常健壮，高性能的生产线。
