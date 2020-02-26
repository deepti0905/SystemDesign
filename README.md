# SystemDesign
I am writing down points from https://www.youtube.com/watch?v=bUHFg8CZFws, for my quick reference later on.

Ask Requirement Clarification questions.
It is impossible to solve the problem in 45 min interview. This is mainly to get insight on where interviewer is heading to.
# Things you should focus on
* Users/Customers -> who and how will they use our system
* Scale(read and write)-> How our system will handle a growing amount of data
  * How many read queries per second?
  * How much data is queried per request??
  * How many video views are processed per second?
  * Can there be spikes in traffic?
      * We must expect response from the interviewer on this or assume some value
* Performance -> How fast our system must be
  * What is expected write to read data delay?
    * can we consider this in batch processing or real time?
  * What is expected p99 latency for the read queries?
    * should we precache data before it is read to reduce the latency
* Cost -> Budget Constraints
  * Minimize cost of development -> use open source technologies
  * Minimize cost of maintenance -> use public cloud
  
**Not getting requirements clarified is a big no sign from any interviewer, basically this helps you set the functional and non functional requirements from now on**
* Functional Requirement
  * What system will support?
    * Here is a hint from the video creator to write these as simple functions
    * This would help you in finalizing the right parameters without much thinking on Http Request and Response syntax (Guideline think simple and build up)
* Non Functional Requirement
 **usually interviewer will not tell us what non functional requirement is. They would say it say it should be scalable and as fast as possible. It is difficult to achieve both together and we would need to achieve the tradeoffs**
    * how a system is supposed to be
    * Scalable 10000 video views per second
    * Highly Performant (10 ms time to view total view count of a video)
    * Highly Available (survives hardware/network failues, no single point of failure)
*Time to mention the CAP Theorm. Write the non functional requirements on the board as well. This will help you choose amongst different technologies*

    
# Now time for a simple design on the board
![Basic Diagram](https://github.com/deepti0905/SystemDesign/blob/master/Basic%20diagram.PNG)

**At this point it is possible that interviewer can go bonkers and start pushing in terms of topic he/she knows the most. It is in your interest to drive this conversation and move interviewer to different direction, which in most cases is data... what data we are saving and how?**

# Enters the Data Modeling section
* What we are storing
* Different tables.
**Be ready with different types of tables, what is the advantage/disadvantage of each. You can even as interviewer's opinion here on what they prefer. If no opinion have you own thought process ready on why you choose one over the other**

## Where we store
* How to **scale writes**?
* How to **scale reads**?
* How to make both **writes and reads fast**?
* How to **not lose data** in case of hardware faults and network partitions?
* How to achieve **strong consistency**? What are the tradeoffs?
* How to **recover data** in case of outage?
* How to ensure **data security**?
* How to make it **extensible** for data model changes in the future?

### Scaling SQL DataBases
Thanks to the creator of the video on how beautifully he explained the steps
* Things are simple when we can save every thing in on one server
* More data implies data needs to be spread across multiple servers. Enters **Sharding**
* We have processing service that stores data in the database and query service that retreives data from the database
* But how do these services know which shard to pick data. Enters a **proxy service** which knows where to pick what data
* This relieves pressure on processing/query service to know where to pick data and puts the entire load on the proxy service.
* Now how will proxy service know if a shard died or a new server is added to the cluster. Enters a **configuration service (Zookeeper)**. 
* Configuration service maintains health check connection to all servers so that it knows what db machines are available
* Now we are maintaining db handles in the cluster proxy. Enters **shard Proxy**, a proxy server sitting on every shard.
* Shard Proxy can do following
  * Cache Results
  * Terminate big queries
  * Do Health checks
  * Send performance metrics
* Till this point we have targeted **scalability** but not **availability**. What if one shard died? Enters **Read Replicas**
* Read Replicas can be placed in another datacenter so that if one datacenter fails we have a backup and a failover strategy.
* Data is either **synchronously or asynchronously synched** between master and read replicas
![SQL DB](https://github.com/deepti0905/SystemDesign/blob/master/SQL_DB.PNG)

**Points to note the above solution is not simple. we have leader/read replicas/ proxies/ config service etc**

### Scaling in NoSQL Databases
* In NoSQL we also divide data into shards. But instead of leaders and followers we say every db is equal.
* We no longer need the **configuration server** to get the state of shards. Instead shards talk to each other, enters **gossip protocol**.
* To reduce network load it is not required that every shard talks to every other shard. Instead every shard talks to few nearby shards. Quickly information on shards is propagated across entire cluster
* Processing Service will make a write request, we can use **round robin** to direct the request to any specific node.
* Node 4 can direct the request to any node which as per consistent hashing is responsible for the specific key.
* Node 4 can also direct requests to multiple nodes and can use quorum consensus to prove if data read/write was successful or not.
* Similar behavior can be used for read as well.
* For replication we can have a cassandra cluster in another data center as well.
![NOSQL DB](https://github.com/deepti0905/SystemDesign/blob/master/NOSQL_DB.PNG)


**Points worth mentioning here is consistency. If we choose availability over consistency we prefer to show stale data instead of not showing data at all. For consistency synchronous replication is used which is slow**

### How we store

* For SQL-->  Data is normalized we keep different tables all of them have different ids as primary keep. There is no data duplication and joins are required for returning combination data.
* For NOSQL--> Every information can stay as a blob or document and for every new info a new column can be added for that specific id.
![SQL vs NSQL](https://github.com/deepti0905/SystemDesign/blob/master/SQLvsNOSQL.PNG)

#### Types of NOSQL
* Column
  * cassandra (is Fault tolerant, wide column, master less, supports cross data center replication)
  * Hbase (has master slave replication)
* Value
* Key Value
  * Couch DB
  * Rocks DB
* Document
  * MongoDB (uses leader based replication)
* Graph
  * Neo4j
  
# Components
## Processing Service
**Rember all the requirements you have listed before and focus on following points**
* How to **scale**
  * Remember partitioning
* How to achieve **high throughput**
  * In Memory
* How **not to lose data** when node crashes occur
  * Replication
* What to do when db is **unavailabe** or **Slow**
![processing service](https://github.com/deepti0905/SystemDesign/blob/master/Partition_Detailed_Design.PNG)

### Data Aggregation Basics

* Users push data to Processing service and processing service writes to DB
* To expedite the processing service can cache the writes and purge them to DB from time to time
* What if Processing service dies in this push model. The data in processing service is lost.
* Create Queue or temp service and processing service can poll it. Once processing service has information from the queue it can start acting on it. If Processing service crashes some other service can pick up **remember write ahead log**
![Data Aggregation](https://github.com/deepti0905/SystemDesign/blob/master/Data_Aggregation.PNG)

#### Check Pointing
* Queue to hold all input requests and then the data is commited in order
* What if data input is large. Enters **partitioning**  more queues adding data to DB

* Partition Shard gets messages
* Partition consumer polls the message
* Redis Cache to track last messages to avoid duplication
* Aggregator to cummulate the messages, again uses some in memory store
* output is then pushed to internal queue
* Database writer polls the queue and writes to the Database.
* If data from the db writer cannot be sent to DB.Either due to faults, network partitioning, delays, they go to dead letter queue.
* The meta information for the processing can be present in Embedded Databases. These databases are present on the same machine as the processing service
* state management--> maintains inmemory information incase of failure of machine which is running Processing service. We can resume from the db responsible for state management

### Data Ingestion path

* multiple queues or partitions
* multiple processing services polling on the partitions
* Partitioner Service --> divides data into partitions
* Load Balancer --> evenly distribute load across multiple partition service
* User APIGatewap to direct request to backend services machines
![Data Ingestion](https://github.com/deepti0905/SystemDesign/blob/master/Data_Ingestion.PNG)

### Ingestion path components
#### Partitioner Service Client
##### Blocking IO
* One request opens one socket on the server. By that one socket is blocked on the server side
* This happens within a single execution thread. So the thread that handles that connection is blocked as well.
* Another client's request will require another thread to be created
* Above is how blocking system works.
* Let's say server starts to observe a slow down as the number of threads have increased on the system.
* This can result in bringing down the entire cluser **death spiral**
* Enters **Rate Limiter**, we can limit the number of request/connections per user.
##### NonBlocking IO
* single thread on the server side to handle multiple concurrent connections
* excessive requests can be piled up on a queue. Piling up queue is far less expensive than pilling up threads.
* Non Blocking systems are more efficient and results in higher throughput
**So why dnt we just use Non Blocking systems?**
* Blocking systems are easy to operate and debug
* Local variables can be used in the blocking systems

##### Buffering & Batching

* Multiple input requests require multiple API Servers
* To compensate for multiple API Servers, we may require multiple workers of processing servers.
* This is expensive, alternate is we add a buffer where we keep adding data and once it reaches a optimum size it is batched to worker service.
* Benefits is it increases throughput, it helps to save on cost, request.
* Drawback, it introduces complexity on both client and server side.
* e.g worker processes batch request, some part of it is success and some is failure. What should happen should request for complete batch or just failed events. I think this is dependent on the system

##### Timeouts

How much time client is ready to wait to get the response from the server. We have two types of Time outs
* connection Timeout
Client is ready to wait this much Time to establish a connection
**this value usually 10ms**
* request timeout
Time client is ready to wait for a response from Request
  * For this we try to analyze latency percentiles of the system
  * 1% latency percentiles implies 1% of the requests to the system will timeout
##### Retries
* Lets say for the first request we hit a bad machine, chances are that for the next request we'll land on another machine increasing out chances of success
* Be cautious of retries if all clients retry this will result in a retry storm event and will bring the system down.

##### Exponential Backoff or jitter algorithm
* With exponential retries, every next retry wait for exponential increase of time till it reaches a maximum cutoff
* With Jitter the retry wait is random
* If the worker service is down almost all request will be retries and irrespective of the algorithm used it will bring the system down.

##### Circuit Breaker
* If we find that total number of errors for a service is beyond a threshold we can stop calling the downstream service.
   * This makes system difficult to test and ste proper thresholds and timers
 
#### Load Balancer
##### H/W LB
* Are network devices we buy from organizations
* Powerfull machines designed to handle high throughput

##### S/W LB
* s/w we install on commodity h/w
* Most of them are open source. We don't need big machines
*  ELB from Amazon is also a LB
50:20 continue

#### Partitioner Service and Partitions











 










   





