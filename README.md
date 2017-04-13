# java-interview
Java面试重难点剖析（不断更新）

1、高并发访问数据库优化方法

    一、服务器配置优化

    我们需要根据应用服务器的性能和并发访问量的大小来规划应用服务器的数量。有一个使用原则是：单台应用服务器的性能不一定要求最好，但是数量一定要足够，最好能有一定的冗余来保障服务器故障。特别是，在高并发访问峰期间，适当增加某些关键应用的服务器数量。比如在某些高峰查询业务上，可以使用多台服务器，以满足用户每小时上百万次的点击量。

    二、使用负载均衡技术

    负载均衡技术是解决集中并发访问的核心技术,也是一种较为有效的解决网站大规模并发访问的方法。实现负载均衡技术的主要设备是负载均衡器服务器。例如，我们把网站部署到在两台不同的服务器之上（前提是要保证这2台或者多台服务器都可以正常运行网站程序），这几台服务器之间通过安装特定的软件实现负载均衡。那么，某个时刻，当网站面临大规模访问时，用户的请求会通过负载均衡程序，根据不同服务器的繁忙和资源情况，自动分配到处理性能最优的服务器上，从而将大规模用户产生的高并发访问均衡地分流到各个服务器上。这样就能大大减轻单台服务器处理高并发请求，确保整个网站系统面临高负载时的可靠性。

    三、数据库结构设计

    这部分是程序层的问题，通常是由软件工程师进行负责，对SQL语句进行优化。我们可以采取的措施包括：对经常查询的数据库字段做索引、对数据库表进行分区操作（如对海量数据进行分区操作十分必要，例如针对按年份存取的数据，我们可以按年进行分区）、对数据库查询语句-SQL（减少冗余的数据库操作，提高查询效率）进行优化等。

    四、中间件的优化

    所谓的中间件，听起来会有点像很深的技术，其实就在我们身边，各位站长朋友经常在网站部署的时候用到的Apache、IIS、Tomcat、WebLogic都是中间件。中间件主要位于客户端/服务器的操作系统之上，负责计算机的资源管理和网络通讯。举个简单的例子，我们在部署JAVA项目的时候，通常都是用Tomcat中间件，那么Tomcat在默认情况下是不优化的，当在高并发的情况下，非常容易当机。关于Tomcat的优化给出以下几个建议（本人在实际项目开发过程中觉得较为重要的几点）：①线程池优化；②启动占用内存优化；③日志输出优化；④HTTP压缩优化；⑤配置文件优化。

    上面举例的Tomcat中间件（也就是WEB服务器）只是一个例子，不同的网站采用不同的架构，那么对相应的中间件的优化也会有不同的方法，比如微软的IIS有相应的配置参数，所以具体的优化方法可以根据项目的需要，查阅中间件的官方文档说明进行参数设置，这样才能实现中间件的最优设置。

    五、数据缓存技术的使用

    现在大多数大型网站都有使用缓存技术，把用户经常使用到的数据通过缓存(Cache)技术进行管理，从而减轻服务器重新请求的压力，提高网站的访问速度。缓存技术有很多，这里我个人根据实际的项目经验，可以将其分成2种，即数据缓存和页面缓存。

    ①所谓的是数据缓存，指的是数据库的数据不是直接传输，而是将数据调用到内存，然后从内存中读取，从而可以大大提高读取速度。数据缓存技术有很多的方案，这里由于开源、高性能等特点，建议使用Memcache来设置数据缓存技术来加速动态web应用程序，减轻数据库负载。

    ②页面缓存一定程度上是针对公共页面，静态化也是页面缓存的一种，将用户经常访问的页面在服务器的相应目录下生成静态页面，当用户再次访问时，不需要对服务器进行动态请求，而只需要对缓存下来的html页面直接读取，这样访问的效率就可以得到有效的提高。

2、 java finalize方法总结、GC执行finalize的过程

    1. finalize的作用
    finalize()是Object的protected方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法。
    finalize()与C++中的析构函数不是对应的。C++中的析构函数调用的时机是确定的（对象离开作用域或delete掉），但Java中的finalize的调用具有不确定性
    不建议用finalize方法完成“非内存资源”的清理工作，但建议用于：① 清理本地对象(通过JNI创建的对象)；② 作为确保某些非内存资源(如Socket、文件等)释放的一个补充：在finalize方法中显式调用其他资源释放方法。其原因可见下文[finalize的问题]
    2. finalize的问题
    一些与finalize相关的方法，由于一些致命的缺陷，已经被废弃了，如System.runFinalizersOnExit()方法、Runtime.runFinalizersOnExit()方法
    System.gc()与System.runFinalization()方法增加了finalize方法执行的机会，但不可盲目依赖它们
    Java语言规范并不保证finalize方法会被及时地执行、而且根本不会保证它们会被执行
    finalize方法可能会带来性能问题。因为JVM通常在单独的低优先级线程中完成finalize的执行
    对象再生问题：finalize方法中，可将待回收对象赋值给GC Roots可达的对象引用，从而达到对象再生的目的
    finalize方法至多由GC执行一次(用户当然可以手动调用对象的finalize方法，但并不影响GC对finalize的行为)
    3. finalize的执行过程(生命周期)

    (1) 首先，大致描述一下finalize流程：当对象变成(GC Roots)不可达时，GC会判断该对象是否覆盖了finalize方法，若未覆盖，则直接将其回收。否则，若对象未执行过finalize方法，将其放入F-Queue队列，由一低优先级线程执行该队列中对象的finalize方法。执行finalize方法完毕后，GC会再次判断该对象是否可达，若不可达，则进行回收，否则，对象“复活”。
    (2) 具体的finalize流程：
    对象可由两种状态，涉及到两类状态空间，一是终结状态空间 F = {unfinalized, finalizable, finalized}；二是可达状态空间 R = {reachable, finalizer-reachable, unreachable}。各状态含义如下：
    unfinalized: 新建对象会先进入此状态，GC并未准备执行其finalize方法，因为该对象是可达的
    finalizable: 表示GC可对该对象执行finalize方法，GC已检测到该对象不可达。正如前面所述，GC通过F-Queue队列和一专用线程完成finalize的执行
    finalized: 表示GC已经对该对象执行过finalize方法
    reachable: 表示GC Roots引用可达
    finalizer-reachable(f-reachable)：表示不是reachable，但可通过某个finalizable对象可达
    unreachable：对象不可通过上面两种途径可达
    状态变迁图：

    变迁说明：
    新建对象首先处于[reachable, unfinalized]状态(A)
    随着程序的运行，一些引用关系会消失，导致状态变迁，从reachable状态变迁到f-reachable(B, C, D)或unreachable(E, F)状态
    若JVM检测到处于unfinalized状态的对象变成f-reachable或unreachable，JVM会将其标记为finalizable状态(G,H)。若对象原处于[unreachable, unfinalized]状态，则同时将其标记为f-reachable(H)。
    在某个时刻，JVM取出某个finalizable对象，将其标记为finalized并在某个线程中执行其finalize方法。由于是在活动线程中引用了该对象，该对象将变迁到(reachable, finalized)状态(K或J)。该动作将影响某些其他对象从f-reachable状态重新回到reachable状态(L, M, N)
    处于finalizable状态的对象不能同时是unreahable的，由第4点可知，将对象finalizable对象标记为finalized时会由某个线程执行该对象的finalize方法，致使其变成reachable。这也是图中只有八个状态点的原因
    程序员手动调用finalize方法并不会影响到上述内部标记的变化，因此JVM只会至多调用finalize一次，即使该对象“复活”也是如此。程序员手动调用多少次不影响JVM的行为
    若JVM检测到finalized状态的对象变成unreachable，回收其内存(I)
    若对象并未覆盖finalize方法，JVM会进行优化，直接回收对象（O）
    注：System.runFinalizersOnExit()等方法可以使对象即使处于reachable状态，JVM仍对其执行finalize方法
    4. 一些代码示例

    (1) 对象复活
    public class GC {

        public static GC SAVE_HOOK = null;

        public static void main(String[] args) throws InterruptedException {
            SAVE_HOOK = new GC();
            SAVE_HOOK = null;
            System.gc();
            Thread.sleep(500);
            if (null != SAVE_HOOK) { //此时对象应该处于(reachable, finalized)状态
                System.out.println("Yes , I am still alive");
            } else {
                System.out.println("No , I am dead");
            }
            SAVE_HOOK = null;
            System.gc();
            Thread.sleep(500);
            if (null != SAVE_HOOK) {
                System.out.println("Yes , I am still alive");
            } else {
                System.out.println("No , I am dead");
            }
        }

        @Override
        protected void finalize() throws Throwable {
            super.finalize();
            System.out.println("execute method finalize()");
            SAVE_HOOK = this;
        }
    }
    (2)覆盖finalize方法以确保资源释放
    作为一个补充操作，以防用户忘记“关闭“资源，JDK中FileInputStream、FileOutputStream、Connection类均用了此”技术“，下面代码摘自FileInputStream类
        /**
     * Ensures that the <code>close</code> method of this file input stream is
     * called when there are no more references to it.
     *
     * @exception  IOException  if an I/O error occurs.
     * @see        java.io.FileInputStream#close()
     */
    protected void finalize() throws IOException {
        if ((fd != null) &&  (fd != FileDescriptor.in)) {
            /* if fd is shared, the references in FileDescriptor
             * will ensure that finalizer is only called when
             * safe to do so. All references using the fd have
             * become unreachable. We can call close()
             */
            close();
        }
    }
    参考：
    12.6 Finalization of Class Instances https://notendur.hi.is//~snorri/SDK-docs/lang/lang083.htm
    深入理解java的finalize http://zhang-xzhi-xjtu.iteye.com/blog/484934
    The Finalizable Object  http://www.artima.com/interfacedesign/Finalizable.html

3、简单理解Socket及TCP/IP、Http、Socket的区别
    TCP/IP
    要想理解socket首先得熟悉一下TCP/IP协议族， TCP/IP（Transmission Control Protocol/Internet Protocol）即传输控制协议/网间协议，定义了主机如何连入因特网及数据如何再它们之间传输的标准，
    从字面意思来看TCP/IP是TCP和IP协议的合称，但实际上TCP/IP协议是指因特网整个TCP/IP协议族。不同于ISO模型的七个分层，TCP/IP协议参考模型把所有的TCP/IP系列协议归类到四个抽象层中
    应用层：TFTP，HTTP，SNMP，FTP，SMTP，DNS，Telnet 等等
    传输层：TCP，UDP
    网络层：IP，ICMP，OSPF，EIGRP，IGMP
    数据链路层：SLIP，CSLIP，PPP，MTU
    每一抽象层建立在低一层提供的服务上，并且为高一层提供服务，看起来大概是这样子的
                            
    估计有兴趣打开此文的同学都对此有一定了解了，加上我也是一知半解，所以就不详细解释，有兴趣同学可以上网上搜一下资料
    维基百科 
    百度百科
    在TCP/IP协议中两个因特网主机通过两个路由器和对应的层连接。各主机上的应用通过一些数据通道相互执行读取操作
    
    socket
    我们知道两个进程如果需要进行通讯最基本的一个前提能能够唯一的标示一个进程，在本地进程通讯中我们可以使用PID来唯一标示一个进程，但PID只在本地唯一，网络中的两个进程PID冲突几率很大，这时候我们需要另辟它径了，我们知道IP层的ip地址可以唯一标示主机，而TCP层协议和端口号可以唯一标示主机的一个进程，这样我们可以利用ip地址＋协议＋端口号唯一标示网络中的一个进程。
    能够唯一标示网络中的进程后，它们就可以利用socket进行通信了，什么是socket呢？我们经常把socket翻译为套接字，socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。

    socket起源于UNIX，在Unix一切皆文件哲学的思想下，socket是一种"打开—读/写—关闭"模式的实现，服务器和客户端各自维护一个"文件"，在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。
    socket通信流程
    socket是"打开—读/写—关闭"模式的实现，以使用TCP协议通讯的socket为例，其交互流程大概是这样子的

    服务器根据地址类型（ipv4,ipv6）、socket类型、协议创建socket
    服务器为socket绑定ip地址和端口号
    服务器socket监听端口号请求，随时准备接收客户端发来的连接，这时候服务器的socket并没有被打开
    客户端创建socket
    客户端打开socket，根据服务器ip地址和端口号试图连接服务器socket
    服务器socket接收到客户端socket请求，被动打开，开始接收客户端请求，直到客户端返回连接信息。这时候socket进入阻塞状态，所谓阻塞即accept()方法一直到客户端返回连接信息后才返回，开始接收下一个客户端谅解请求
    客户端连接成功，向服务器发送连接状态信息
    服务器accept方法返回，连接成功
    客户端向socket写入信息
    服务器读取信息
    客户端关闭
    服务器端关闭
    三次握手
    在TCP/IP协议中，TCP协议通过三次握手建立一个可靠的连接

    第一次握手：客户端尝试连接服务器，向服务器发送syn包（同步序列编号Synchronize Sequence Numbers），syn=j，客户端进入SYN_SEND状态等待服务器确认
    第二次握手：服务器接收客户端syn包并确认（ack=j+1），同时向客户端发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态
    第三次握手：第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1），此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手
    定睛一看，服务器socket与客户端socket建立连接的部分其实就是大名鼎鼎的三次握手
    <img src="http://images.cnitblog.com/blog/349217/201312/05234946-b80841921eae4d2ab983f26ed9023768.png" alt="">
    socket编程API
    前面提到socket是"打开—读/写—关闭"模式的实现，简单了解一下socket提供了哪些API供应用程序使用，还是以TCP协议为例，看看Unix下的socket API，其它语言都很类似（PHP甚至名字都几乎一样），这里我就简单解释一下方法作用和参数，具体使用有兴趣同学可以看看博客参考中的链接或者上网搜索
    int socket(int domain, int type, int protocol);
    根据指定的地址族、数据类型和协议来分配一个socket的描述字及其所用的资源。
    domain:协议族，常用的有AF_INET、AF_INET6、AF_LOCAL、AF_ROUTE其中AF_INET代表使用ipv4地址
    type:socket类型，常用的socket类型有，SOCK_STREAM、SOCK_DGRAM、SOCK_RAW、SOCK_PACKET、SOCK_SEQPACKET等
    protocol:协议。常用的协议有，IPPROTO_TCP、IPPTOTO_UDP、IPPROTO_SCTP、IPPROTO_TIPC等
    int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    把一个地址族中的特定地址赋给socket
    sockfd:socket描述字，也就是socket引用
    addr:要绑定给sockfd的协议地址
    addrlen:地址的长度
    通常服务器在启动的时候都会绑定一个众所周知的地址（如ip地址+端口号），用于提供服务，客户就可以通过它来接连服务器；而客户端就不用指定，有系统自动分配一个端口号和自身的ip地址组合。这就是为什么通常服务器端在listen之前会调用bind()，而客户端就不会调用，而是在connect()时由系统随机生成一个。
    int listen(int sockfd, int backlog);
    监听socket
    sockfd:要监听的socket描述字
    backlog:相应socket可以排队的最大连接个数 
    int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
    连接某个socket
    sockfd:客户端的socket描述字
    addr:服务器的socket地址
    addrlen:socket地址的长度
    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    TCP服务器监听到客户端请求之后，调用accept()函数取接收请求
    sockfd:服务器的socket描述字
    addr:客户端的socket地址
    addrlen:socket地址的长度
    ssize_t read(int fd, void *buf, size_t count);
    读取socket内容
    fd:socket描述字
    buf：缓冲区
    count：缓冲区长度
    ssize_t write(int fd, const void *buf, size_t count);
    向socket写入内容，其实就是发送内容
    fd:socket描述字
    buf：缓冲区
    count：缓冲区长度
    int close(int fd);
    socket标记为以关闭 ，使相应socket描述字的引用计数-1，当引用计数为0的时候，触发TCP客户端向服务器发送终止连接请求。
    参考
    Linux Socket编程（不限Linux）
    揭开Socket编程的面纱 

4、 java之yield(),sleep(),wait()区别详解-备忘笔记

5、线程本地存储TLS(Thread Local Storage)的原理和实现——分类和原理

6、 JVM系列三:JVM参数设置、分析

7、 虚拟内存机制

8、搜索算法集锦

9、关于B树的一些总结

10、 Redis中5种数据结构的使用场景介绍

11、 Maven如何解决包冲突问题

12、 maven中如何将所有引用的jar包打包到一个jar中

13、Java并发之CountDownLatch、CyclicBarrier和Semaphore

14、使用LinkedHashMap构建LRU的Cache

15、java提高篇(八)----详解内部类

16、谈谈ConcurrentHashMap1.7和1.8的不同实现

17、单例模式和双重检查锁定

18、 java中实现多态的机制是什么?

19、 java提高篇(四)-----理解java的三大特性之多态

20、 java类静态域、块，非静态域、块，构造函数的初始化顺序

21、HashMap底层实现原理/HashMap与HashTable区别/HashMap与HashSet区别

22、几种网络I/O模型

23、 字典树(Trie树)实现与应用

24、从输入url到显示网页，后台发生了什么？

25、网页打开时都发生了什么？我被吓着了

26、 实现一个 能在O(1)时间复杂度 完成 Push、Pop、Min操作的栈

27、 线程返回值的方式介绍

28、MyBatis 拦截器原理探究

29、Java 四种线程池

