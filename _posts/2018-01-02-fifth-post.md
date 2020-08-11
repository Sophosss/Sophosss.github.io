---
title: 你知道的Java
author: Sophosss
layout: post
---
1. 为什么1.8后永久代中的字符串常量池移到了堆中，然后消除了永久代？

   永久代很容易发生内存泄漏，为了不会遇到永久代存在的内存溢出错误，也不会出现泄漏的数据移到交换区这样的事情。用户可以为元空间设置一个可用空间最大值，如果不进行设置，JVM会自动根据类的元数据大小动态增加元空间的容量。  

2. 方法区的垃圾回收？永久代会垃圾回收吗？
   - 废弃常量：如果一个字符串"abc"已经进入常量池，但是当前系统没有任何一个String对象引用了叫做"abc"的字面量，那么如果发生垃圾回收并且有必要时，"abc"就会被系统移出常量池。
   - 无用的类：该类的所有实例都已经被回收，即Java堆中不存在该类的任何实例；加载该类的ClassLoader已经被回收；该类对应的java.lang.class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

3. 进程间通信机制有哪些？（进程间通信有哪几种方式？）
   - 管道：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
   - 信号量：信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。
   - 消息队列：消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
   - 信号 ：信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。
   - 共享内存：共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC  方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号量，配合使用，来实现进程间的同步和通信。
   - 套接字：套解字也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同及其间的进程通信。

4. 访问一个网站，发生了什么？
   - 输入www.google.com，DNS协议帮助我们将这个网址转换成IP地址。已知DNS服务器为8.8.8.8，于是我们向这个地址发送一个DNS数据包（53端口）。DNS服务器做出响应，告诉我们它的IP地址是172.194.72.105。
   - 已知子网掩码是255.255.255.0，本机用它对自己的IP地址192.168.1.100，做一个二进制的AND运算，结果为192.168.1.0。然后对Google的IP地址172.194.72.105也做一个AND运算，计算结果为172.194.72.0。这两个结果不相等，所以结论是，Google与本机不在同一个子网络。因此，我们要向Google发送数据包，必须通过网关192.168.1.1转发。
   - TCP数据包需要设置端口，接收方（Google）的HTTP端口默认是80，发送方（本机）的端口是一个随机生成的1024-65535之间的整数，假定为51775。
   - TCP数据包嵌入IP数据包，IP数据包需要设置双方的IP地址，这是已知的，发送方是192.168.1.100（本机），接收方是172.194.72.105（Google）。
   - IP数据包嵌入以太网数据包，以太网数据包需要设置双方的MAC地址，发送方为本机的网卡MAC地址，接收方为网关192.168.1.1的MAC地址（通过ARP协议得到）。
   - 经过多个网关的转发，Google的服务器172.194.72.105，收到了这所有以太网数据包。根据IP标头的序号，Google将所有包拼起来，取出完整的TCP数据包，然后读出里面的"HTTP请求"，接着做出"HTTP响应"，再用TCP协议发回来。
   - 本机收到HTTP响应以后，就可以将网页显示出来，完成一次网络通信。

5. ArrayList 和 LinkedList 的区别？

   线程安全，底层数据结构，插入删除，快速随机访问，内存空间占用…

6. RandomAccess？

   源码中`ArrayList`类实现了`RandomAccess`接口，`LinkedList`类中却没有实现这个接口，这个接口仅是一个标识Marker，实现了这个接口的 List 将支持快速随机访问。如果传入的 List 实现了`RandomAccess`接口，采用普通for循环遍历；若传入的 List 未实现`RandomAccess`接口，采用`iterator`遍历

7. ArrayList 的扩容机制？

   先看`add()`方法的源码，当向`ArrayList`对象中添加新元素时，首先会调用`ensureCapacityInternal(size)`方法，`size`为最小扩容量；`ensureCapacityInternal()`方法会首先调用`calculateCapacity`来确定需要的最小容量；最后调用`ensureExplicitCapacity()`方法判断是否需要扩容。最后判断所需最小容量如果大于当前数组的空间大小，则需要扩容，调用`grow()`方法扩容。

   言而言之，`ArrayList`扩容的本质就是计算所需扩容`size`得到新的数组，将原数组中的数据复制到新数组中，最后将原数组指向新数组在堆内存的引用地址即可。

8. HashMap 和 HashTable 的区别？

   线程安全，效率，对 null key 和 null value 的支持，底层数据结构

9. hashmap ？

   JDK1.8: Table数组 + Entry链表 / 红黑树，采用位运算法，相比直接取余，位运算直接对内存中的二进制数据操作，不需要再转换为十进制，因此处理速度很快。默认采用了链地址法解决Hash冲突问题

10. 上下文切换？

    并发编程中实际线程的数量都可能大于CPU核心的个数，而CPU一个核心在任意时刻只能被一个线程使用，CPU为了保证并发的线程都有被执行，采用随机分配时间片并轮转的方式。而一个线程的时间片用户将保存并进入就绪状态直到下次分配时间片再执行，这个任务从保存到再加载的过程就是一次上下文切换。

11. sleep() 方法和 wait() 方法的区别？
    - sleep 方法没有释放锁，wait 方法释放了锁
    - 两者都可以暂停线程的执行
    - `wait()`通常用于线程间交互/通信，`sleep()`通常用户暂停执行
    - `wait()`方法被调用后，线程不会自动苏醒，需要别的线程调用同一对象上的`notify()`或者`notifyAll()`方法。`sleep()`方法执行完成后，线程会自动苏醒。

12. synchronized 关键字？

    - 修饰实例方法，作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁。
    - 修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁，会作用于类的所有对象实例。
    - 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

    synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。 当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权.当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

    synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

    Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

13. ReenTrantLock 和 synchronized 区别？
    - 都是可重入锁，自己可以再次获取自己的内部锁。
    - synchronized 依赖于 JVM 而 ReenTrantLock 依赖于 API。
    - ReenTrantLock 比 synchronized 增加了一些高级功能：1.等待可中断；2.可实现公平锁；3.可实现选择性通知（锁可以绑定多个条件）。

14. Comparable\<T>和Comparator\<T>区别？

    一个类实现Comparable\<T>接口，那么这个类本身就具有了被排序的功能。一个类如果没有实现Comparable\<T>接口，要使该类在数组中能排序，就要另外再写一个针对该类的排序类,新写的类必须实现Comparator\<T>功能。

    也就是说，一个是自己有比较功能，另一个是让第三方类实现比较功能。

15. session 与 cookie 区别？
    - cookie 数据存放在客户的浏览器上，session 数据放在服务器上。
    - cookie 不是很安全，别人可以分析存放在本地的 cookie 并进行 cookie 欺骗，考虑到安全应当使用 session。
    - session 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用 cookie。
    - 单个 cookie 保存的数据不能超过 4K，很多浏览器都限制一个站点最多保存 20 个 cookie。

16. 线程中 run() 和 start() 的区别？

    - run()相当于线程的任务处理逻辑的入口方法，它由Java虚拟机在运行相应线程时直接调用，而不是由应用代码进行调用。

    - 而start()的作用是启动相应的线程。启动一个线程实际是请求Java虚拟机运行相应的线程，而这个线程何时能够运行是由线程调度器决定的。start()调用结束并不表示相应线程已经开始运行，这个线程可能稍后运行，也可能永远也不会运行。

17. 为什么需要双亲委派模型？

    防止内存中出现多份同样的字节码，Java 类随着它的类加载器一起具备了一种带有优先级的层次关系。为了使类对象进行比较有意义，不使用会给虚拟机的安全带来隐患。

18. 类加载的过程？
    - 加载：Java虚拟机将.class文件读入内存，并为之创建一个Class对象。任何类被使用时系统都会为其创建一个且仅有一个Class对象。这个Class对象描述了这个类创建出来的对象的所有信息。
    - 链接：验证阶段。主要的目的是确保被加载的类（.class文件的字节流）满足Java虚拟机规范，不会造成安全错误；准备阶段。负责为类的静态成员分配内存，并设置默认初始值；解析阶段。将类的二进制数据中的符号引用替换为直接引用。
    - 初始化：初始化阶段是执行类构造器`<client>`方法的过程。`<client>`方法是由编译器自动收集类中的类变量的赋值操作和静态语句块中的语句合并而成的。只对static修饰的变量或语句块进行初始化。如果初始化一个类的时候，其父类尚未初始化，则优先初始化其父类。

19. Java如何处理死锁？
    - 使用信号量控制最多一个线程访问资源。
    - 避免在同步代码块中调用外部同步方法。
    - 在嵌套多层synchronized同步块中，对锁进行排序，使得每次获得锁的顺序一致。

20. 实现线程安全的类？

    vector， stack，hashtable， enumeration，StringBuffer，ConcurrentHashMap，blockingQueue

21. MyISAM 和 InnoDB？
    - InnoDB是事务安全的，支持行级锁，适合高并发，支持外键，支持MVCC
    - MyISAM 是非事务安全的，表级锁，性能优先，不支持外键，不支持MVCC
    - ps：应对高并发事务, MVCC比单纯的加锁更高效；MVCC只在 `READ COMMITTED` 和 `REPEATABLE READ` 两个隔离级别下工作；MVCC可以使用 乐观(optimistic)锁 和 悲观(pessimistic)锁来实现。
    
22. Java 中 new 一个对象的步骤?

    - 首先去检查这个指令的参数是否能在常量池中能否定位到一个类的符号引用，并且验证是否是第一次使用该类。

    - 类加载检查通过后，接下来虚拟机将为新生的对象在堆中分配内存（指针碰撞，空闲列表）。

    - 内存分配完后，虚拟机需要将分配到的内存空间中的数据类型都初始化为零。

    - 虚拟机要对对象头进行必要的设置 ，例如这个对象是哪个类的实例（即所属类）、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。

      ps：对象的访问定位：直接指针访问，句柄访问。

23. GC三个算法，分代收集讲下？

    标记清除，复制，标记整理，分代收集老年代标记整理，新生代复制。

24. 泛型的使用？

    泛型类、泛型接口、泛型方法

25. servlet 的生命周期？
    - 在客户端请求servlet时，Tomcat容器会检测是否有请求的servlet的实例存在。
    - 如果servlet实例不存在，则调用其构造方法创建servlet的实例，在实例被创建成功后调用init()方法进行初始化工作，初始化成后将该Servlet保存起来以备后续请求使用，再调用service()方法处理客户端的请求。如果是get请求，service()方法调用doGet()方法处理客户端请求；如果是post请求，service()方法调用doPost()方法处理客户端请求。
    - 如果Servlet实例存在，则调用其service方法，如果是get请求，service()方法调用doGet()方法处理客户端请求，如果是post请求，service()方法调用doPost()方法处理客户请求。
    - 当容器关闭时，调用Servlet的destroy()方法。
      

26. IOC, DI, AOP
    - IOC ：借助于“第三方”(Spring 中的 IOC 容器) 实现具有依赖关系的对象之间的解耦(IOC容易管理对象，你只管使用即可)，从而降低代码之间的耦合度。
    - DI ：是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。

27. Spring 的工厂设计模式？
    - `BeanFactory` ：延迟注入(使用到某个 bean 的时候才会注入),相比于`BeanFactory` 来说会占用更少的内存，程序启动速度更快。
    - `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。`BeanFactory` 仅提供了最基本的依赖注入支持，` ApplicationContext` 扩展了 `BeanFactory` , 除了有`BeanFactory`的功能还有额外更多功能，所以一般开发人员使用` ApplicationContext`会更多。`ClassPathXmlApplication` 是它的一个实现类。