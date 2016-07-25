# Threading

Perfect provides a core threading library in the PerfectThread package. This package is designed to provide support for the rest of the systems in Perfect. PerfectThread is abstracted over the core operating system level threading package.

PerfectThread is imported by PerfectNet and so it is not generally required that one directly import it. However if you need to do so you can ```import PerfectThread```.

PerfectThread provides the following constructs:

* Threading.Lock - Mutually exclusive thread lock, aka critical section.
* Threading.RWLock - A many reader/single writer based thread lock.
* Threading.Event - Wait/signal/broadcast type synchronization.
* Threading.sleep - Block/pause a single thread for a given period of time.
* Threading.ThreadQueue - Create either a serial or concurrent thread queue with a given name.
* Threading.dispatch - Dispatch a closure on a named queue.

These systems provide internal concurrency for Perfect and are heavily used in the PerfectNet package in particular.

### Locks


### Events


### Queues

