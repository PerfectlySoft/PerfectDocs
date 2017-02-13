# Threading

Perfect provides a core threading library in the PerfectThread package. This package is designed to provide support for the rest of the systems in Perfect. PerfectThread is abstracted over the core operating system level threading package.

PerfectThread is imported by PerfectNet and so it is not generally required that one directly import it. However, if you need to do so you can ```import PerfectThread```.

PerfectThread provides the following constructs:

* Threading.Lock - Mutually exclusive thread lock, a.k.a. a mutex or critical section
* Threading.RWLock - A many reader/single writer based thread lock
* Threading.Event - Wait/signal/broadcast type synchronization
* Threading.sleep - Block/pause; a single thread for a given period of time
* Threading.ThreadQueue - Create either a serial or concurrent thread queue with a given name
* Threading.dispatch - Dispatch a closure on a named queue

These systems provide internal concurrency for Perfect, and are heavily used in the PerfectNet package in particular.

### Locks

PerfectThread provides both mutex and rwlock synchronization objects.

**Mutex**

Mutexes are provided through the Threading.Lock object. These are intended to protect shared resources from being accessed simultaneously by multiple threads at once. It provides the following functions:

```swift
/// A wrapper around a variety of threading related functions and classes.
public extension Threading {
	/// A mutex-type thread lock.
	/// The lock can be held by only one thread. 
	/// Other threads attempting to secure the lock while it is held will block.
	/// The lock is initialized as being recursive. 
	/// The locking thread may lock multiple times, but each lock should be accompanied by an unlock.
	public class Lock {
		/// Attempt to grab the lock.
		/// Returns true if the lock was successful.
		public func lock() -> Bool
		/// Attempt to grab the lock.
		/// Will only return true if the lock was not being held by any other thread.
		/// Returns false if the lock is currently being held by another thread.
		public func tryLock() -> Bool
		/// Unlock. Returns true if the lock was held by the current thread and was successfully unlocked, or the lock count was decremented.
		public func unlock() -> Bool
		/// Acquire the lock, execute the closure, release the lock.
		public func doWithLock(closure: () throws -> ()) rethrows
	}
}
```

The general usage pattern as as follows:

* Created a shared instance of ```Threading.Lock```
* When a shared resource needs to be accessed, call the ```lock()``` function
	* If another thread already has the lock then the calling thread will block until it is unlocked
* Once the call to ```lock()``` returns the resource is safe to access
* When finished, call ```unlock()```
	* Other threads are now free to acquire the lock

Alternatively, you can pass a closure to the ```doWithLock``` function. This will lock, call the closure, and then unlock.

The ```tryLock``` function will lock and return true if no other thread currently holds the lock. If another thread holds the lock, it will return false.

**Read/Write Lock**

Read/Write Locks (RWLock) are provided through the ```Threading.RWLock``` object. RWLocks support many threads accessing a shared resource in a read-only capacity. For example, it could permit many threads at once to be accessing values in a shared Dictionary. When a thread needs to perform a modification (a write) to a shared object it acquires a write lock. Only one thread can hold a write lock at a time, and all other threads attempting to read or write will block until the write lock is released. An attempt to lock for writing will block until any other read or write locks have been released. When attempting to acquire a write lock, no other read locks will be permitted, and threads attempting to read or write will be blocked until the write lock is held and then released.

RWLock is defined as follows:

```swift
/// A wrapper around a variety of threading related functions and classes.
public extension Threading {
	/// A read-write thread lock.
	/// Permits multiple readers to hold the while, while only allowing at most one writer to hold the lock.
	/// For a writer to acquire the lock all readers must have unlocked.
	/// For a reader to acquire the lock no writers must hold the lock.
	public final class RWLock {
		/// Attempt to acquire the lock for reading.
		/// Returns false if an error occurs.
		public func readLock() -> Bool
		/// Attempts to acquire the lock for reading.
		/// Returns false if the lock is held by a writer or an error occurs.
		public func tryReadLock() -> Bool
		/// Attempt to acquire the lock for writing.
		/// Returns false if an error occurs.
		public func writeLock() -> Bool
		/// Attempt to acquire the lock for writing.
		/// Returns false if the lock is held by readers or a writer or an error occurs.
		public func tryWriteLock() -> Bool
		/// Unlock a lock which is held for either reading or writing.
		/// Returns false if an error occurs.
		public func unlock() -> Bool
     /// Acquire the read lock, execute the closure, release the lock.
		public func doWithReadLock(closure: () throws -> ()) rethrows        
     /// Acquire the write lock, execute the closure, release the lock.
		public func doWithWriteLock(closure: () throws -> ()) rethrows
	}
}
```

RWLock supports ```tryReadLock``` and ```tryWriteLock```, both of which will return false if the lock cannot be immediately acquired. It also supports ```doWithReadLock``` and ```doWithWriteLock``` which will call the provided closure with the lock held and then release it when it has completed.

### Events

The ```Threading.Event``` object provides a way to safely signal or communicate among threads about particular events. For instance, to signal worker threads that a task has entered a queue. It's important to note that ```Threading.Event``` inherits from ```Threading.Lock``` as using the ```lock``` and ```unlock``` methods provided therein are vital to understanding thread event behaviour.

```Threading.Event``` provides the following functions:

```swift
public extension Threading {
	/// A thread event object. Inherits from `Threading.Lock`.
	/// The event MUST be locked before `wait` or `signal` is called.
	/// While inside the `wait` call, the event is automatically placed in the unlocked state.
	/// After `wait` or `signal` return the event will be in the locked state and must be unlocked.
	public final class Event: Lock {
		/// Signal at most ONE thread which may be waiting on this event.
		/// Has no effect if there is no waiting thread.
		public func signal() -> Bool
		/// Signal ALL threads which may be waiting on this event.
		/// Has no effect if there is no waiting thread.
		public func broadcast() -> Bool
		/// Wait on this event for another thread to call signal.
		/// Blocks the calling thread until a signal is received or the timeout occurs.
		/// Returns true only if the signal was received.
		/// Returns false upon timeout or error.
		public func wait(seconds secs: Double = Threading.noTimeout) -> Bool
	}
}
```

The general usage pattern is illustrated by using a producer/consumer metaphor:

*Producer Thread*

* Producer thread wants to produce a resource and alert other threads about the occurrence
* Call the ```lock``` function
* Produce the resource
* Call the ```signal``` or ```broadcast``` function
* Call the ```unlock``` function

*Consumer Thread*

* Call the ```lock``` function
* If a resource is available for consumption:
	* Consume the resource and call the ```unlock``` function
* If a resource is not available for consumption:
	* Call the ```wait``` function
	* When ```wait``` returns true:
		* If a resource is available then consume the resource
	* Call the ```unlock``` function

These producer/consumer threads generally operate in a loop performing these steps repeatedly during the life of the program.

The ```wait``` function accepts an optional timeout parameter. If the timeout expires then ```wait``` will return false. By default, ```wait``` does not timeout.

The functions ```signal``` and ```broadcast``` differ in that ```signal``` will alert at most one waiting thread while ```broadcast``` will alert all currently waiting threads.

### Queues

The PerfectThread package provides an abstracted thread queue system. It is based loosely on Grand Central Dispatch (GCD), but is designed to mask the actual threading primitives, and as such, operate with a variety of underlying systems.

This queue system provides the following features:

* Named serial queues - one thread operating, removing and executing tasks
* Named concurrent queues - multiple threads operating, the count varying depending on the number of available CPUs, removing and executing tasks simultaneously
* A default concurrent queue

This system provides the following functions:

```swift
/// A thread queue which can dispatch a closure according to the queue type.
public protocol ThreadQueue {
	/// The queue name.
	var name: String { get }
	/// The queue type.
	var type: Threading.QueueType { get }
	/// Execute the given closure within the queue's thread.
	func dispatch(_ closure: Threading.ThreadClosure)
}

public extension Threading {
	/// The function type which can be given to `Threading.dispatch`.
	public typealias ThreadClosure = () -> ()
	/// Queue type indicator.
	public enum QueueType {
		/// A queue which operates on only one thread.
		case serial
		/// A queue which operates on a number of threads, usually equal to the number of logical CPUs.
		case concurrent
	}
	/// Find or create a queue indicated by name and type.
	public static func getQueue(name nam: String, type: QueueType) -> ThreadQueue
	/// Call the given closure on the "default" concurrent queue
	/// Returns immediately.
	public static func dispatch(closure: Threading.ThreadClosure)
}
```

Calling ```Threading.getQueue``` will create the queue if it does not already exist. Once the queue object has been returned call, its ```dispatch``` function and pass it the closure which will be executed on that queue.

The system will automatically create a queue called "default". Calling the static ```Threading.dispatch``` function will always dispatch the closure on this queue.


