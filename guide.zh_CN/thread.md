# 线程

Perfect包含了完整的线程函数库：PerfectThread。该函数库是整个Perfect系统的基础支持。PerfectThread是从操作系统核心级别中抽象出的类库。

由于PerfectThread已经包含在PerfectNet函数中，因此不需要显式声明该库的调用。但是如有必要，您当然也可以通过`import PerfectThread`声明对该库的调用

PerfectThread提供如下功能：

* Threading.Lock - 线程互斥锁，又名互斥或者临界区
* Threading.RWLock - 基于多读单写操作的线程锁
* Threading.Event - 用于线程同步的等待/信号/广播类型
* Threading.sleep - 单线程阻塞/暂停一段时间
* Threading.queue - 为顺序启动或者并发启动的线程创建一个命名队列
* Threading.dispatch - 调度或关闭一个命名队列
* Promise - “承诺”—— 用于处理分布在不同线程上一个或者多个任务，等待所有任务完成并集中返回结果或者错误的函数。

以下系统部件为Perfect提供内部并发支持，而且PerfectNet函数库高度依赖于本函数库。

### Locks 线程锁

PerfectThread同时提供线程互斥锁和多读单写同步锁。

**Mutex线程互斥**

Mutex互斥是通过Threading.Lock线程锁对象实现的。以下内容为在多线程并发环境下保护共享资源的方法：

``` swift
/// 一系列线程有关的函数和类库封装。
public extension Threading {
    /// 线程互斥锁
    /// 该锁只能被一个线程持有
    /// 其它试图解锁的线程都会被阻塞。
    /// 该锁被初始化为递归调用（存在调用堆栈）
    /// 锁定线程可以多次加锁，但是每次加锁必须配对一个对应解锁。
    public class Lock {
        /// 试图加锁
        /// 如果加锁成功则返回真值。
        public func lock() -> Bool
        /// 试图加锁
        /// 只有当前锁未被其它线程锁定时才返回真值。
        /// 如果已经被其它线程锁定，则返回假。
        public func tryLock() -> Bool
        /// 解锁。只有该所被当前线程锁定并成功解锁时才返回真值。否则锁定计数器将递减
        public func unlock() -> Bool
        /// 针对闭包申请加锁，并尝试执行，随后解锁
        public func doWithLock(closure: () throws -> ()) rethrows
    }
}
```

使用方法可参考如下：

* 创建一个```Threading.Lock```实例
* 当需要访问共享资源时，调用```lock()``` 函数
* 如果其它线程已经对此加锁了，则当前进程将被阻塞直至解锁
* 一旦成功调用```lock()```后，当前线程即可对预期资源进行安全的独占访问
* 访问结束后，调用```unlock()```解锁
* 解锁之后，其它线程即可重新对其加锁访问

另一种方式就是直接将需要加锁的内容做成闭包，传递给```doWithLock```函数进行调用。此时该函数回自动加锁、调用闭包，而后自动解锁。

如果没有其它线程锁定资源，则```tryLock```函数回自动加锁并返回真值。如果其它线程已经锁定，则返回假。

**多读单写锁**

调用```Threading.RWLock```对象来实现多读单写锁。该所支持多个线程同时以只读方式读取一个共享资源。比如，可以允许多个线程同时访问一个共享的字典。当其中一个线程需要进行修改（写操作）时，则该线程可以申请一个单写锁。同一时间内只有一个线程可以进行写锁定，其它所有尝试读写该锁的线程都会被阻塞直到该锁（单线程修改操作）释放。当某线程试图获取一个修改锁时，其它线程的读取锁操作也被禁止，所有其它尝试对该所进行读写的线程都会被阻塞直至修改操作完毕为止。

多读单写锁定义如下：

``` swift
/// 一系列线程有关的函数和类库封装。
public extension Threading {
        /// 尝试读取锁信息
        /// 如果有错误发生则返回假。
        public func readLock() -> Bool
        /// 尝试读取锁信息
        /// 如果当前锁已经被某修改线程锁定，或者有错误发生，将返回假。
        public func tryReadLock() -> Bool
        /// 尝试获取锁用于写操作。
        /// 如果有错误发生则返回假。
        public func writeLock() -> Bool
        /// 尝试获取锁用于写操作。
        /// 如果已经被其它读写线程锁定或有错误发生，将返回假。
        public func tryWriteLock() -> Bool
        /// 为读写操作解除锁定
        /// 如果有错误发生将返回假。
        public func unlock() -> Bool
        /// 尝试获取读锁并执行闭包操作，执行结束后自动解锁。
        public func doWithReadLock(closure: () throws -> ()) rethrows
        /// 尝试获取写锁并执行闭包操作，执行结束后自动解锁。
        public func doWithWriteLock(closure: () throws -> ()) rethrows
}
```

RWLock同时支持 ```tryReadLock``` 和 ```tryWriteLock```方法，二者都是在锁无法立刻获取的情况下返回假，同时支持```doWithReadLock``` 和 ```doWithWriteLock``` 方法，用于为闭包自动加锁，以及执行结束自动解锁

### Events线程事件（信号灯）

其中，```Threading.Event```线程事件对象为线程间就特定事件通信提供了一个安全的信号灯机制。比如，向进入任务队列的一个工作线程发出一个信号通知其有事件发生。需要注意的是```Threading.Event```线程事件对象是从```Threading.Lock```线程锁对象继承而来，因此其内部使用了```lock```和```unlock```的加锁／解锁机制，这对理解线程事件对象的行为而言是非常重要的。

线程事件```Threading.Event``` 提供以下函数：

``` swift
public extension Threading {
    /// 一个线程事件对象。从线程锁Threading.Lock继承而来.
    /// 在调用wait等待方法和signal信号灯方法之前，该事件必须加锁。
    /// 在调用wait方法过程内部，该事件会自动处于解锁状态。
    /// 在调用wait等待或者signal信号灯后，该事件会处于锁定状态，因此必须要进行解锁。
    public final class Event: Lock {
        /// 单信号触发：通过信号灯触发和等待这个事件的线程最多只有一个。
        /// 如果没有等待线程，则调用时不会有任何效果。
        public func signal() -> Bool
        /// 广播：触发等待该事件的所有线程。
        /// 如果没有等待线程，则调用不会有任何效果。
        public func broadcast() -> Bool
        /// 等待该事件直到另一个线程触发信号灯。
        /// 阻塞调用线程直至信号灯触发或者超时。
        /// 只有在信号灯触发后才会返回真值。
        /// 如果超时或者有错误发生，则返回假。
        public func wait(seconds secs: Double = Threading.noTimeout) -> Bool
    }
}
```

线程事件的基本使用方法可以用下面的生产者/消费者这个例子进行描述：

*生产者线程*

* 生产者线程将创建一个资源并通知其它线程这个资源已经发生创建的事件
* 调用```lock```函数
* 产生这个可以共享的资源
* 调用```signal```信号灯方法或者```broadcast```广播方法
* 调用```unlock```解锁函数

*消费者线程*

* 调用```lock```函数
* 如果资源已经准备好而且可以被消费
  * 使用资源（进行消费操作）然后调用```unlock```函数
* 如果该资源尚不具备使用条件（无法消费）：
  * 调用```wait```线程等待方法
  * 当```wait```返回为真值时：
    * 如果资源已经处于可以使用状态，则进行消费操作
    * 调用```unlock```线程解锁函数

上述生产者/消费者线程会在程序生命周期内周而复始地重复上述操作。

而```wait```等待函数允许接受一个可选的超时参数。如果已经发生超时则```wait```会返回假。默认情况下```wait```不会超时。

函数```signal```和```broadcast```这两个线程调度函数区别在于```signal```最多唤醒一个等待线程，而```broadcast```会唤醒所有处于等待状态的线程。

### 线程队列

PerfectThread线程函数库提供一个从操作系统中抽象出来的队列系统。该函数库基于（苹果体系）广义中央调度系统（GCD），但是也被设计为概括真实的线程基本操作，因此可以在不同的平台上使用。

该队列系统有以下特点：

* 命名顺序队列：每个时刻只有一个线程处于运行状态，从队列中脱离并启动后执行具体任务。
* 命名并发队列：每个时刻根据实际的CPU数量尽可能多的允许多个线程进行同时操作，从队列中脱离并启动后在同一时刻并发执行任务
* 默认并发队列：由系统自动命名为“default”的并发命名队列

该体系提供以下函数：

``` swift
/// 线程队列可以根据其队列类型调度闭包。
public protocol ThreadQueue {
    /// 队列名称。
    var name: String { get }
    /// 队列类型。
    var type: Threading.QueueType { get }
    /// 在线程队列中执行指定闭包。
    func dispatch(_ closure: Threading.ThreadClosure)
}

public extension Threading {
    /// 该函数类型相当于Threading.dispatch线程调度
    public typealias ThreadClosure = () -> ()
    /// 队列类型指示器。
    public enum QueueType {
        /// 顺序队列，每次只允许一个线程脱离队列进行操作。
        case serial
        /// 并发队列，允许同时执行多个线程，允许的并发线程数量通常等于逻辑CPU的实际数量。
        case concurrent
    }
    /// 根据命名和类型寻找或者创建一个队列。
    public static func getQueue(name nam: String, type: QueueType) -> ThreadQueue
    /// 根据指定类型返回一个匿名队列
    /// 如果没有返回 ThreadQueue对象，则该队列不能使用。
    /// 如果不再需要该队列，则请注销。
    public static func getQueue(type: QueueType) -> ThreadQueue
    /// 返回默认队列
    public static func getDefaultQueue() -> ThreadQueue
    /// 终止并注销队列
    public static func destroyQueue(_ queue: ThreadQueue    /// 在名为“default”的默认的并发队列中调用指定闭包
    /// 会立刻返回。
    public static func dispatch(closure: Threading.ThreadClosure)
}
```

通过```Threading.getQueue```获取命名队列时，如果队列不存在，则会自动创建一个队列。一旦队列对象调用后返回，其调度函数```dispatch```将把闭包传递给队列进行执行操作

系统回自动创建一个默认的队列，名字就叫“default”。调用```Threading.dispatch```静态函数时，总是会通过这个名为“default”的默认队列执行调度闭包的操作。

### Promise “承诺”

承诺是用于一个或者多个线程之间的对象。承诺能够以线程的方式执行指定的闭包，一旦生产者线程完成工作并返回结果，则消费者线程能够获取该结果，或者是该工作执行时产生的错误。

该对象有两种使用方法：

* 链式承诺：首先是执行一个闭包，同时允许闭包不包含任何参数，同时返回类型也允许指定；随后可以进行一系列链式操作，用.then子句进行连接。

举例：三线程串式写法、并发执行、合并等待。

```swift
let v = try Promise { 1 }
	.then { try $0() + 1 }
	.then { try $0() + 1 }
	.wait()
// 如果没问题的话，变量 v 的结果等于3
```

注意上面的例子中，每一个`.then`子句尾随的闭包可接受的函数必须是从前驱线程（前驱线程）返回的结果，或者是检测并处理前驱线程的错误。

* 将承诺以参数方式传递给其他线程的函数或者闭包。该承诺可以在后续执行中触发`.set`或者`.fail`，即返回执行结果，或者抛出一个错误对象。⚠️注意⚠️承诺代码段set/fail是可以多次反复调用的，这种方式提高了编程的灵活性，允许执行一系列异步操作。

举例：暂停并设置逻辑值
Example: Pause then set a Bool value
		
```swift
let prom = Promise<Bool> {
	(p: Promise) in
	Threading.sleep(seconds: 2.0)
	p.set(true)
}
XCTAssert(try prom.get() == nil) // 承诺尚未履行（即预期条件尚未满足）
XCTAssert(try prom.wait(seconds: 3.0) == true) // 等待三秒后是否设置结果
```

无论是上述哪一种方法，承诺声明的闭包函数都会立刻另起线程开始工作。

承诺函数库的完整调用参考：

```swift
public class Promise<ReturnType> {
	/// 以尾随闭包方式初始化一个承诺，闭包回继续传递该承诺，
	/// 用于随时设置承诺的结果值或错误对象。
	/// 该闭包会启动一个新的线程队列并立刻执行。
	public init(closure: @escaping (Promise<ReturnType>) throws -> ())
	
	/// 带尾随闭包初始化一个承诺。
	/// 该闭包回返回一个执行类型结果，一旦结果有效则意味着承诺已经履行。
	/// 该闭包会启动一个新的线程队列并立刻执行。
	public init(closure: @escaping () throws -> ReturnType)

	/// 在现有承诺基础上链式追加新承诺。
	/// 新的尾随闭包回继承前驱线程的结果。
	public func then<NewType>(closure: @escaping (() throws -> ReturnType) throws -> NewType) -> Promise<NewType>
}

public extension Promise {
	/// 获取承诺结果。如果承诺已经履行，则结果有效；
	/// 未履行承诺返回为 nil
	/// 出错时抛出异常。
	/// 该函数应该由消费者线程调用
	public func get() throws -> ReturnType?
	
	/// 获取承诺结果。如果承诺已经履行，则结果有效；
	/// 未履行承诺返回为 nil
	/// 出错时抛出异常。
	/// 阻塞当前线程并直到承诺履行或者超时。
	/// 该函数应该由消费者线程调用
	public func wait(seconds: Double = Threading.noTimeout) throws -> ReturnType?
}

public extension Promise {
	/// 在生产者线程中设置承诺的返回结果，允许消费者调用读取该结果。
	public func set(_ value: ReturnType)
	
	/// 在生产者线程中触发错误并设置错误对象。
	public func fail(_ error: Error)
}
```
