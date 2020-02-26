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
### Data Aggregation Basics

* Users push data to Processing service and processing service writes to DB
* To expedite the processing service can cache the writes and purge them to DB from time to time
* What if Processing service dies in this push model. The data in processing service is lost.
* Create Queue or temp service and processing service can poll it. Once processing service has information from the queue it can start acting on it. If Processing service crashes some other service can pick up **remember write ahead log**




 










   





