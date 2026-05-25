There is no official library named library-quartx within the [Moqui Ecosystem](https://github.com/moqui) used for Inter-Process Communication (IPC). [1] 
If you are working with Moqui and handling background tasks, clustering, or communication between different application instances, you are likely looking for one of the following official mechanisms:
## 1. Moqui System Messages (SystemMessage)
For asynchronous, persistent communication between separate processes or external systems, Moqui provides a built-in Store-and-Forward Message Queue. [2] 

* Functionality: It persists outgoing messages to allow automated retries upon failure and guarantees incoming message processing.
* Use Case: Reliable system-to-system messaging without needing a heavy external broker.
* Example: The [Moqui Example Component](https://github.com/moqui/example) features pre-configured system message produce and consume workflows to demonstrate this setup. [2, 3] 

## 2. Quartz Scheduler (Background Execution)
If you misremembered the spelling of Quartz, the [Moqui Framework](https://github.com/moqui/moqui-framework) natively integrates Quartz Scheduler for job scheduling and background execution. [4] 

* IPC Context: In a clustered environment, multiple Moqui application server processes connect to a shared database. Quartz coordinates job distribution across these processes using database-backed clustering, preventing identical background tasks from firing simultaneously on different server instances.

## 3. Distributed Caching & Hazelcast
For real-time, low-latency inter-process synchronization or state sharing across different JVM processes, Moqui leverages Hazelcast as its distributed cache provider.

* Functionality: It manages distributed maps, locks, and topic listeners across multiple node clusters.

## 4. EntitySync
If your goal is data replication between completely detached processes or distinct tenants across different networks, Moqui handles this natively via its EntitySync engine. It duplicates records sequentially based on precise update timestamps. [2] 
------------------------------
If you are trying to implement a specific communication workflow between your Moqui instances, could you clarify:

* Are you trying to pass messages, sync database records, or trigger background jobs across servers?
* Is this communication happening on the same physical server or across a network?

Providing these details will help me point you to the exact Moqui API or configuration file needed.

[1] [https://github.com](https://github.com/moqui)
[2] [https://github.com](https://github.com/moqui/moqui/releases)
[3] [https://github.com](https://github.com/moqui/example)
[4] [https://github.com](https://github.com/moqui/moqui-framework)
