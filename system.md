# system design

###### tags: `interview`
https://vagrantpi.github.io/2019/11/01/understanding-worker-threads-in-Node.js/

linked hash map
https://stackoverflow.com/questions/19738977/how-is-the-internal-implementation-of-linkedhashmap-different-from-hashmap-imple
![Uploading file..._y0x005j7v]()



consistent hash ring -> solve the inefficiency of moving data after removing/adding nodes in the current system.

## System

當系統中有狀況發生的時候，造成服務沒辦法正常運作，或許應該要有個healthcheck的機器可以定期去probe系統的狀態，並且可以在第一時間拒絕接受client request，並且發送錯誤訊息通知顧客

### Service mesh

基本上由各個Side car proxy組成的網路 加上control plane就是service mesh
可以做到的事情包括了傳統proxy和gateway的職責
1. 認證加強服務間溝通的安全性還有Access control
1. load balancing
1. retry, failover, rate-limiting
1. monitor metrics, logs, tracing，包含ingress, egress流量


## Language

### Nodejs

#### async await?

這只是語法糖，他其實就是把await後續的所有程式碼用resolve().then()包起來


#### why nodejs uses event-based networking instead of thread based?

網路應用程式，會有非常大量的Network IO，特別是在今日講求模組化，微服務化，也因此thread-based表示各request會有一個獨立的thread去服務他，當遇到I/O時就會被block住，而thread的資源就會被hang在那裏無法使用

event-based則是在需要I/O時，會不管他，直接把該thread歸還給thread pool供其他人使用，並且在callback回來時，放進callback queue，並在callstack為空的時候，運用event loop將之放回callstack執行後續指令
整個過程只會使用一個或是少數的thread數量

**然而，若為CPU-bound的工作，則適合使用thread-based模式，不然那單個thread被卡在callstack當中一直執行，則在callback queue中的作業便會完全hang在那裏了**

#### Event loop

setTimeout()
call stack -> webAPI Timer-> call back queue
-(if callstack is empty)->eventloop -> callstack(execute)
ref: https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif

#### Worker-thread變數共享
可以使用message passing複製一份變數到message queue當中，以利各個thread撿起來
或是shared memory(SharedBufferArray)來共享一份變數(需要使用atomics operation)

#### event-driven language

Event-driven programming is a programming paradigm in which the flow of the program is determined by events such as user actions (mouse clicks, key presses), sensor outputs, or messages from other programs/threads.


#### Why it is single threaded?
- Thread-based networking is relatively inefficient and very difficult to use. Furthermore, users of Node.js are free from worries of dead-locking the process, since there are no locks. Almost no function in Node.js directly performs I/O, so the process never blocks. Because nothing blocks, scalable systems are very reasonable to develop in Node.js. — Node.js Documentation
- Because Node handles many clients with few threads, if a thread blocks handling one client’s request, then pending client requests may not get a turn until the thread finishes its callback or task. The fair treatment of clients is thus the responsibility of your application. This means that you shouldn’t do too much work for any client in any single callback or task. — Node.js Documentation
- Workers (threads) are useful for performing CPU-intensive JavaScript operations. They will not help much with I/O-intensive work. Node.js’s built-in asynchronous I/O operations are more efficient than Workers can be. — Node.js Documentation


## Common

### hash vs encoding

- hash: checksum，確保data integrity，將長度轉化為固定長度大小的hash值
- encoding: 使資料可以傳輸於不同系統之間

## Web

### http pipelining

將多個HTTP請求（request）整批送出的技術，而在傳送過程中不需先等待伺服器的回應。
但會有一些問題，因為server還是得按照順序回復，因此使用queue採取一個FIFO的回覆，但也會因此有後續的request都卡在第一個request的問題（Head-of-line blocking (HOL blocking) 
因此目前的concurrent request都是採用多個persistent reusable connection一起傳送，而http2可支援更多的consistent connection

ref:https://ishwar-rimal.medium.com/why-does-your-browser-limit-the-number-of-concurrent-network-calls-1ae5d50863dd


http1.0 single request single response connection close
http1.1 persistent connection(connection pool)
http2.0 support more persistent reusable connection

但每一個瀏覽器仍會限制同時開啟的concurrent connection，以避免造成DOS攻擊

### Websocket disadvantage

cannot cache data or response like http

### Web protocol

Here is a framework for thinking about web protocols:

- TCP: low-level, bi-directional, full-duplex, and guaranteed order transport layer. No browser support (except via plugin/Flash).
- HTTP 1.0: request-response transport protocol layered on TCP. The client makes one full request, the server gives one full response, and then the connection is closed. The request methods (GET, POST, HEAD) have specific transactional meaning for resources on the server.
- HTTP 1.1: maintains the request-response nature of HTTP 1.0, but allows the connection to stay open for multiple full requests/full responses (one response per request). Still has full headers in the request and response but the connection is re-used and not closed. HTTP 1.1 also added some additional request methods (OPTIONS, PUT, DELETE, TRACE, CONNECT) which also have specific transactional meanings. However, as noted in the introduction to the HTTP 2.0 draft proposal, HTTP 1.1 pipelining is not widely deployed so this greatly limits the utility of HTTP 1.1 to solve latency between browsers and servers.
- Long-poll: sort of a "hack" to HTTP (either 1.0 or 1.1) where the server does not respond immediately (or only responds partially with headers) to the client request. After a server response, the client immediately sends a new request (using the same connection if over HTTP 1.1).
- HTTP streaming: a variety of techniques (multipart/chunked response) that allow the server to send more than one response to a single client request. The W3C is standardizing this as Server-Sent Events using a text/event-stream MIME type. The browser API (which is fairly similar to the WebSocket API) is called the EventSource API.
- Comet/server push: this is an umbrella term that includes both long-poll and HTTP streaming. Comet libraries usually support multiple techniques to try and maximize cross-browser and cross-server support.
- WebSockets: a transport layer built-on TCP that uses an HTTP friendly Upgrade handshake. Unlike TCP, which is a streaming transport, WebSockets is a message based transport: messages are delimited on the wire and are re-assembled in-full before delivery to the application. WebSocket connections are bi-directional, full-duplex and long-lived. After the initial handshake request/response, there is no transactional semantics and there is very little per message overhead. The client and server may send messages at any time and must handle message receipt asynchronously.
- SPDY: a Google initiated proposal to extend HTTP using a more efficient wire protocol but maintaining all HTTP semantics (request/response, cookies, encoding). SPDY introduces a new framing format (with length-prefixed frames) and specifies a way to layering HTTP request/response pairs onto the new framing layer. Headers can be compressed and new headers can be sent after the connection has been established. There are real world implementations of SPDY in browsers and servers.
- HTTP 2.0: has similar goals to SPDY: reduce HTTP latency and overhead while preserving HTTP semantics. The current draft is derived from SPDY and defines an upgrade handshake and data framing that is very similar the the WebSocket standard for handshake and framing. An alternate HTTP 2.0 draft proposal (httpbis-speed-mobility) actually uses WebSockets for the transport layer and adds the SPDY multiplexing and HTTP mapping as an WebSocket extension (WebSocket extensions are negotiated during the handshake).
- WebRTC/CU-WebRTC: proposals to allow peer-to-peer connectivity between browsers. This may enable lower average and maximum latency communication because as the underlying transport is SDP/datagram rather than TCP. This allows out-of-order delivery of packets/messages which avoids the TCP issue of latency spikes caused by dropped packets which delay delivery of all subsequent packets (to guarantee in-order delivery).
- QUIC: is an experimental protocol aimed at reducing web latency over that of TCP. On the surface, QUIC is very similar to TCP+TLS+SPDY implemented on UDP. QUIC provides multiplexing and flow control equivalent to HTTP/2, security equivalent to TLS, and connection semantics, reliability, and congestion control equivalentto TCP. Because TCP is implemented in operating system kernels, and middlebox firmware, making significant changes to TCP is next to impossible. However, since QUIC is built on top of UDP, it suffers from no such limitations. QUIC is designed and optimised for HTTP/2 semantics.

References:https://stackoverflow.com/questions/14703627/websockets-protocol-vs-http


## MQ

### kafka tx

一個transaction當中，producer可以使用多個thread來同時傳送多筆訊息

commit offset也可以放到transaction裡面來確保atomicity

### kafka exactly once sending
ref: https://www.baeldung.com/kafka-exactly-once
缺點:retry overhead提高

### Kafka isloation
kafka transaction: ensure Atomic multi-partition writes

kafka islotion level: 在consumer那邊設定
When we consume, we can read all the messages on a topic partition in order. Though, we can indicate with isolation.level that we should wait to read transactional messages until the associated transaction has been committed:

```java=
Properties consumerProps = new Properties();
consumerProps.put("bootstrap.servers", "localhost:9092");
consumerProps.put("group.id", "my-group-id");
consumerProps.put("enable.auto.commit", "false");
consumerProps.put("isolation.level", "read_committed");
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
consumer.subscribe(singleton(“sentences”));
```


### Kafka Idempotent Producer

note: 這個sequence number原理類似於tcp的的原理，tcp也是運用這個方法確定資料的ordering和deduplication，而receiver若再依定時間內沒有回覆ack，則會再重送，

可以確保message被送達一次而已 不會多也不會少
原理:每個Producer會有一個ID(transactional.id)，同時送出的訊息也會有sequence number，並隨著訊息送出而遞增
假設斷線要retry重送，會先檢查目前這個訊息的number是否大於最後一個訊息，若是的話，則忽略，表示之前送過了
反之，則表示沒有接受過這筆訊息，開始重送

But with idempotence enabled, each message comes with a PID(transactional.id) and sequence number:
```
M1 (PID: 1, SN: 1) - written to partition. For PID 1, Max SN=1
M2 (PID: 1, SN: 2) - written to partition. For PID 1, Max SN=2
M3 (PID: 1, SN: 3) - written to partition. For PID 1, Max SN=3
M4 (PID: 1, SN: 4) - written to partition. For PID 1, Max SN=4
M5 (PID: 1, SN: 5) - written to partition. For PID 1, Max SN=5
M6 (PID: 1, SN: 6) - written to partition. For PID 1, Max SN=6
M4 (PID: 1, SN: 4) - rejected, SN <= Max SN
M5 (PID: 1, SN: 5) - rejected, SN <= Max SN
M6 (PID: 1, SN: 6) - rejected, SN <= Max SN
M7 (PID: 1, SN: 7) - written to partition. For PID 1, Max SN=7
M8 (PID: 1, SN: 8) - written to partition. For PID 1, Max SN=8
M9 (PID: 1, SN: 9) - written to partition. For PID 1, Max SN=9
M10 (PID: 1, SN: 10) - written to partition. For PID 1, Max SN=10
```
ref: https://www.cloudkarafka.com/blog/apache-kafka-idempotent-producer-avoiding-message-duplication.html

## DB

### sql vs nosql

儘管nosql不提供ACID，但其實可以運用程式來處理達到ACID，只是這樣就會使nosql逐漸偏向SQL的性質，而犧牲了NoSQL本身的優勢

NoSQL systems often provide weak consistency guarantees such as eventual consistency and transactions restricted to single data items, even though one can impose full ACID guarantees by adding a supplementary middleware layer.


### distributed transactions

short-lived:一個transaction可以快速完成的，可以直接使用2-phase-commit來鎖住資源，並統一commit

long-lived: also called saga transactions, 一般會使用compensation transaction
由於整體transaction時程會拉長，如訂機票，確認時間會拉長到1-2天，那這樣一直鎖著資源也不太好，因此才會採用compensation transaction


### isolation level

越高表示越可以確保serializable，也就是說一件事一件事來，絕對不允許有query執行的時間線重疊，但系統資源被Lock住的時間也越長，產生deadlock的機率越高

level越低表示效率越好，但也會有許多concurrent問題像是read uncommited, dirty read, phantom read etc.


### 達到ACID的方法

**note: DB的Atomic和程式中的Atomic operation概念不太相同 程式中的Atomic operation比較像是Isloation，避免race condition，讓各個執行緒序列化**

為何程式中要叫做Atomic?
One thing atomic operations do is take these operations that humans think of as being single operations, but which the computer sees as multiple operations, and makes the computer see them as single operations, too.

This is why they’re called atomic operations. It’s because they take an operation that would normally have multiple instructions—where the instructions could be paused and resumed—and it makes it so that they all happen seemingly instantaneously, as if it were one instruction. It’s like an indivisible atom.

ref: https://hacks.mozilla.org/2017/06/avoiding-race-conditions-in-sharedarraybuffers-with-atomics/


Atomic:  two-phase commit protocol(for short-lived tx), compensation transaction/saga tx(for long-lived transaction), 或是秉持service間溝通全都透過同個DB 
Isolation: Two-phase locking
Consistency: DB constraints, foreign key constraint

note:
- 2pc, strict 2pc(release exclusive lock after tx commit), rigorous 2pc(release both exclusive & shared lock after tx commit, meaning it doesn't have shrink phase)
- **rigorous 2pc can ensure distributed serializability and global serializability**
- isolation == concurrency -> making transactions serliazability, as if they were happened one by one at a time

### 假設2pc failed在commit stage的最後一秒?

- 首先 一般DB無法支援分散式trasnaction，DB要有支援transaction coordinator才能這樣搞

- 由於該node再prepare stage投票是OK的，而且其他node也已經commit了，因此等到該Node重新起來之後, tx coordinator會再叫他重新commit一次

### MySQL 如何實現 Lock？

https://hugh-program-learning-diary-js.medium.com/後端基礎-資料庫-nosql-transaction-acid-與-lock-882f3079fd3d

Uploading file..._395gsp27l

使用for update

若沒有where id = 1，則會變成table lock
lock再commit之後會release lock
lock 後commit前中間可以再做許多query，然後execute()
全部完成不需要這個row了之後就可以commit並且release了
```php=
$conn->autocommit(FALSE);
$conn->begin_transaction();
$conn->query("SELECT amount from products where id = 1 for update");
$conn->commit();
```


### redis sentinel

哨兵模式(sentinel model)

主從模式實現了讀寫分離，並解決了數據備份和單一模式可能引發的效能問題，但其在使用上也有不便之處。因為客戶端連接到不同的 redis instance 時，都得要指定IP端口，若所連接的 instance 故障下線了，主從模式無法通知客戶端切換到另一個可用的 instance，只能靠手動去修改配置並重新取得新的連線。若是故障的是主節點，所有的從節點會因為沒有主節點而同步中斷，進而變成各自獨立的節點，即整個叢集無效，直至手動修改配置或是重啟主節點。

哨兵模式 Redis 的高可用性解決方案，可以管理多組主從實例，其架構如下圖:



哨兵模式主要執行以下四個任務:

1. Monitoring: sentinel 會持續地定期檢查主從節點是否正常運作

2. Notification: 當被監控的節點服務出現問題時，sentinel 會透過 API 向管理者或其他相關應用程序發送通知

3. Automatic failover: 當主節點未如期工作或無法正常服務時，sentinel 會將失效主節點下的其中一個從節點推選為新的主節點，並設定其他從節點使用新的主節點

4. Configuration provider: 為客戶端提供 service discovery，客戶端可以透過 sentinel 取得主節點的連線資訊，包括故障轉移後新的主節點位址