# Implement Sql Database

## 1. Understanding the 8 Fallacies of Distributed Systems

>https://dzone.com/articles/understanding-the-8-fallacies-of-distributed-syste

### 1.0 Introduction

* Working in distributed system: Microservices, Web APIs, SOA, Web Server, Application server, Database server, Cache server, load balancer. Distributed systems are comprised of many computers that coordinare to achieve a common goal.

* Fallacies of distributed computing are *false assumptions* that many developers make about distributed systems.

* **8 Fallicies are**
    * The network is reliable.
    * Latency is zero.
    * Bandwidth is infinite.
    * The network is secure.
    * Topology doesn't change.
    * There is one administrator.
    * Transport cost is zero.
    * The network is homogeneous.

### 1.1 The Network Is Reliable

* **Problem**
    * `Call over a network will fail.`
    * Systems make calls to other systems (payment gateways, accouting systems, CRMs, web services), What happens if a call fails? HTTP time out exceptions? => **Retry** but **not repeat** charge request.

* **Solutions**
    * If network failed, we could **automatically retry**. Queuing systems are very good at this, store and forward partern, [MSMQ](http://www.simpleorientedarchitecture.com/msmq-basics/) is an example.
    * Moving from request/response model to fire and forget.
* **Conclusion**
    * Network are more reliable. But stuff happens. Hardware and software can fail - power supplies, routers, failed updates or patches, weak wireless signals, network congestion, rodents or sharks. People can start DDOS attacks => Need to consider failure when desigining distributed systems.

### 1.2  Latency Is Zero

* **Problem**
    * `Calls over a network are not instant`

    * Difference between in-memory calls and calls over the internet. 
    
    ```js
    var viewModel = new ViewModel();
    var documents = new DocumentsCollection();
    foreach (var document in documents)
    {
        var snapshot = document.GetSnapshot();
        viewModel.Add(snapshot);
    }
    ```
    * There are two remote call at line 2 and 5. In line 5 can have n + 1 remote call. Repeated remote calls will lead to high latency.
    * **Solutions**
      * Bring Back All the Data You Might Need
      * Move the Data Closer to the Clients
      * Invert the Flow of Data

* **Conlustion**
    * You shouldn't just replace local calls with remote calls. Chances are this will turn your system into a distributed big ball of mud.

### 1.3 Bandwith is Infinite

* **Problems**
    * `Bandwidth is limited.`
    * Although bandwidth has improved over time, the amount of data that we send has increased too. Video streaming or VoIP will need more bandwidth than apps that are passing simple DTOs over the network. 
* **Solutions**
    * Domain-Driven Design Patterns
    * Command and Query Responsibility Segregation
        * The **write model** will ensure invariant hold true and the data is consistent. Since the write model doesn't care about view concerns, it can be kept small and focused.
        * The **read model** is optimized for view concerns, so we can retrieve all the data that is required for a specific view (e.g. a screen in our app).
* **Conclusion**
    * Less data is easier to understand. Less data means less coupling. So transfer only the data that you might need.

### 1.4 The Network Is Secure

* **Problem**
    * `The network is insecure.`
    * Your system is Security Mindset only as secure as your weakest link. The bad news is that there are a lot of links in a distributed system.
    * The attackers of today have a lot of computing power in their hands and a lot of patience. So the question is not if they're going to attack your system, but **when**.
* **Solutions**
    * *Defense in Depth*: Should use a layered approach to secure your system.
    * *Security Mindset*: Keep security in mind when designing your system. The [top ten vulnerabilities](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project) list has not changed that much in the last 5 years. 
    * *Threat Modeling*: You first identify all the assets in your system (user data in the database, files, etc) and how they are accessed.
* **Conclusion** 
    * So it's a good idea to make this clear to the business, decide together how much to invest in security and have a plan for when a security breach does happen.

### 1.5 Topology Doesn't Change

* **Problem**
    * `Network topologies change constantly.`
* **Solutions**
    * *Abstract the Physical Structure of the Network*
      * Stop hardcoding IPs
      * When DNS is not enough (e.g. when you need to map an IP and a port)
      * Service Bus frameworks can also provide location transparency.
    * *Cattle, Not Pets*:  Ensuring that no server is irreplaceable
    * *Test* : Stop a service or shut down a server and see if your system is still up and running
* **Conclusion**
    * Nowadays, with cloud and containers on the rise, it's hard to ignore this fallacy. You need to be prepared for failure and test for it. Don't wait for it to happen in production!

### 1.6 There Is One Administrator

* **Problem**
    * `There is no one person who knows everything.`
    * Many apps interact with 3rd party systems. This means that, if they go down, there isn't much that you can do. So, even if you had one administrator for your system, you still can't control 3rd party systems.
* **Solution**
    * *Everyone Should Be Responsible for the Release Process*
    * *Logging and Monitoring*
    * *Decoupling*
    * *Isolate Third-Party Dependencies*
* **Conclusion**
    * To work around this fallacy, you need to make your system easy to manage. DevOps, logging and monitoring can help.

### 1.7 Transport Cost Is Zero

* **Problem**
    * `Transport cost is not zero.`
      * The Cost of the Networking Infrastructure
      * The Cost of Serialization/Deserialization
* **Solutions**
    * Make sure that you're using it as efficiently as possible. SOAP or XML is more expensive than JSON. JSON is more expensive than binary protocols like Google's Protocol Buffers.
* **Conclusion**
    * You should benchmark and monitor resource consumption and decide if transport cost is a problem for you.

### 1.8 The Network Is Homogeneous

* **Problem**
    * `The network is not homogenous.` (A homogenous network is a network of computers using similar configurations and the same communication protocol.)
* **Solution**
    * You should choose standard formats in order to avoid vendor lock-in: XML, JSON or Protocol Buffers. There are plenty of options to choose from.
* **Conclusion**
    * You need to ensure that the system's components can talk with each other.

### *Designing Distributed Systems Is Hard*

* We must accept these as facts: the network is unreliable, insecure and costs money. Bandwidth is limited. The network's topology will change. Its components are not configured the same way. Being aware of these limitations will help us design better distributed systems.

## 2. FLP Imposibility

> https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf

* In April 1985, FLP Theory, demonstrated by Fischer, Lynch and Patterson, is one of the most important distributed system theories. They also won the Dijkstra Paper Award, the most influential award in the field of distributed computing, for their paper on FLP theory.
* FLP theory proves that in asynchronous communication systems, no consensus protocol can completely solve the consensus issue in spite of one faulty process.
* In a synchronous system, consensus can be achieved. This is because in a synchronous system, when a process fails or a response times out, we can assume that it has crashed.
* While in an asynchronous system, it is impossible for one process to tell whether another has died or is just running very slowly. In this case, if there is a problem with any of the processes, there is no distributed algorithm that can make all non-faulty processes reach a consensus.
*  The problem of consensus - that is, getting a distributed network of processors to agree on a common value - was known to be solvable in a synchronous setting, where processes could proceed in simultaneous steps. In particular, the synchronous solution was resilient to faults, where processors crash and take no further part in the computation. Informally, synchronous models allow failures to be detected by waiting one entire step length for a reply from a processor, and presuming that it has crashed if no reply is received.
* This kind of failure detection is impossible in an asynchronous setting, where there are no bounds on the amount of time a processor might take to complete its work and then respond with a message. Therefore it’s not possible to say whether a processor has crashed or is simply taking a long time to respond. The FLP result shows that in an asynchronous setting, where only one processor might crash, there is no distributed algorithm that solves the consensus problem.

## 3. Consensus (đồng thuận)

* Consensus means multiple servers agreeing on same information, something imperative to design fault-tolerant distributed systems.
* A consensus protocol tolerating failures must have the following features :
  * **Validity** : If a process decides(read/write) a value, then it must have been proposed by some other correct process
  * **Agreement** : Every correct process must agree on the same value
  * **Termination** : Every correct process must terminate after a finite number of steps.
  * **Integrity** : If all correct processes decide on the same value, then any process has the said value.

* There can be two types of systems assuming only one client(for the sake of understandability):

  * **Single Server system** : The client interacts with a system having only one server with no backup. There is no problem in achieving consensus in such a system.

<p align="center">
  <img src="https://cdncontribute.geeksforgeeks.org/wp-content/uploads/single-server-1-raft-visual.png">
</p>

  * **Multiple Server system** : The client interacts with a system having multiple servers. Such systems can be of two types :
    * **Symmetric** :- Any of the multiple servers can respond to the client and all the other servers are supposed to sync up with the server that responded to the client’s request, and
    * **Asymmetric** :- Only the elected leader server can respond to the client. All other servers then sync up with the leader server.

<p align="center">
  <img src="https://cdncontribute.geeksforgeeks.org/wp-content/uploads/multiple-server-1-raft-visual.png">
</p>    

### 3.1 1PC (1 Phase Commit)

* A one-phase commit protocol can be described in just three famous words:

<p align="center">
  <img src="https://vignette.wikia.nocookie.net/cardfight/images/9/97/Star-trek-make-it-so-450x270.jpg/revision/latest?cb=20160325065605">
</p>    

That's it. You're just telling the remote nodes "these are the changes I want implemented, and no backchat from any of you." It's an acceptable protocol when you're prepared to deal with failure after the fact. [more...](https://www.quora.com/What-is-a-one-phase-commit-protocol)

### 3.2 2PC (Two Phase Commit)

>https://www.the-paper-trail.org/post/2008-11-27-consensus-protocols-two-phase-commit/


<p align="center">
  <img src="https://i.imgur.com/A81m84ym.png">
</p>

<p align="center">
  <img src="https://i.imgur.com/DYzgzf8m.png">
</p>

- Distributed two-phase commit reduces the vulnerability of one-phase commit protocols.

#### Steps in commit:
  - Phase 1: `Prepare phase`
      - After each slave node complete its transaction, it will send a "DONE" message to coordinator node. When coordinator node has received "DONE" message from all slaves, it sends a "Prepare" message to the slaves node.
      - The slaves node will vote. If a slave want to commit, its will send a "Ready" message. If a slave want not to commit, its will send a "Not Ready" message.

      <p align="center">
        <img src="https://i.imgur.com/ZKyjU46l.png">
      </p>

  - Phase 2: `Commit/Abort Phase`
      - After all slave sent vote, they will be blocked until receive message from coordinator node.There are 2 cases:
        - Case 1: 
            - Coordinator node received "Ready" message from all the slaves:
                - The coordinator node will send a "Global Commit" message to the slaves node.
                - The slaves apply the transaction and send a "Commit ACK" message to the coordinator node.
                - After the controloling received "Commit ACK" from all slaves node, it considers the transaction as committed.
        - Case 2:
            - The coordinator node received "Not Ready" message from any the slaves:
                - The coordinator node will send a "Global Abort" message to the slaves node.
                - The slaves node abort the transaction and send a “Abort ACK” message to the coordinator node.
                - After the controloling received "Abort ACK" from all slaves node, it considers the transaction as abort.

        <p align="center">
          <img src="https://i.imgur.com/v3twia4l.png">
        </p>

#### The problems:
  - Phase 1:
      - Problem 1:
          - The coordinator node could crash before it send "Prepare" message to slaves node.
          - Solution:
              -  This doesn’t cause us too much worry, as it simply means that 2PC never gets started.
      - Problem 2:
          - The coordinator node could crash after it sent "Prepare" message some slaves. We will have some slaves node received and start 2PC round, and some slaves node never gets start. And if the coordinator doesn’t recover for a long time, the nodes that received the proposal are going to be blocked waiting for the outcome of a protocol that might never be finished.
          - These nodes will have sent back their votes to the coordinator - unaware that it has failed - and therefore can’t simply timeout and abort the protocol since there’s a possibility the coordinator might reawaken, see their ‘commit’ votes and start phase two of the protocol with a commit message.
          - Solution: 
              - We can get another participant node to take over the job of the coordinator node. This node can contact all the slaves node to find vote of them.
              ->  This requires all nodes to keep in persistent storage the results of all 2PC executions.
          
          <p align="center">
            <img src="https://i.imgur.com/9tuuOAAl.png">
          </p>

      - Problem 3:
          - The coordinator node did not receive enought vote message from all slaves node. It mean is some slaves is crashed.
          - Solution:
              - coordinator will have timeout, if time out, it will broadcast "Not ready" message to all slaves node except for crashed node.

      - Problem 4: 
          - The slave is die after sent vote.
          - Solution:
              - If the coordinator received the vote, it waits for other votes and go to phase 2.
              - ⇒ Otherwise: wait for the participant to recover and respond (keep querying it).

  - Phase 2:
      - Problem 1: 
          - The slaves node does not receive commit or abort message from coordinator node. It may the coordinator is crashed before it send.
          - Solution: the same solution solved problem 2 of phase.
      - Problem 2: 
          -  Another slave node crashes before the recovery node can finish the protocol, the state of the protocol cannot be recovered. The recovery node will not know crashed slave node is sent "Ready" or "Not Ready".
          - Solution: use local log of slave node.

      - Problem 3:
          - The coordinator node is timeout when wait ACK message from slaves node.
          - Solution: use local log of slave node.


- The co-ordinator typically will log the result of any succesful protocol in persistent storage, so that when it recovers it can answer inquiries about whether a transaction committed. This allows periodic garbage collection of the logs at the participant nodes to take place: the coordinator can tell nodes that no-one will try to recover a mutually committed transaction and that they can erase its existence from their log (that said, the log might be kept around to be able to recover a node’s state after a crash).
 
- Adding a recovery coordinator:
    - Another system can take over for the coordinator:
        - Could be a participant that detected a timeout to the coordinator.
    - Recovery node needs to find the state of the protocol:
        - Contact ALL participants to see how they voted:
            - If we get voting results from all participants:
                - We know that Phase 1 has completed.
                - If all participants voted to commit ⇒ send commit request.
                - Otherwise send abort request.
    - If ANY participant states that it has not voted:
        - We know that Phase 1 has not completed.
        -> Restart the protocol.
    - But if any participant node also crashes, we’re stuck!
        - Have to wait for recovery

- The other problems:
    - Blocking: if coordinator node fails, the slaves is blocked until coordinator node recovery.
    - There is no way to finish with commit until coordinator and all participants are available. In other words the participant can’t reach a final state in presence of a failure.
    - If the coordinator node and slaves node are crashed:
        - The system has `no way of knowing the result of the transaction`.
        - It might have committed for the crashed participant – hence all others must block. 
    - `The protocol cannot pessimistically abort because some participants may have already committed`
    When a participant gets a commit/abort message, it does not know if every other participant was informed of the result.

### 3.3 3PC (Three Phase Commit)

> https://developer.jboss.org/wiki/Three-phaseCommitProtocol3PC

![Imgur](https://i.imgur.com/k4fKt6Al.png)

- The three-phase commit protocol is said to be non-blocking.
- 3PC splits the prepare state in two: pre-commit and commited.
- 3PC as non-blocking does not mean that the participants are not blocked in processing.  The database has to start a local transaction and put locks on appropriate places when process the transaction and when agree to commit. Thus other operations could be blocked by this effort and needs to wait till the whole 3PC ends.

`The non-blocking means that protocol can proceed despite of existence of failures.`

![Imgur](https://i.imgur.com/ztq0WrU.png)

- Purpose: `let every participant know the state of the result of the vote so that state can be recovered if anyone dies.`
- When any participant is in the pre-commit state the transactions is about to commit - participants can be sure that coordinator decided to commit before. 
- When there is no participant in pre-commit state the transaction is about to abort - participants know it’s possible that coordinator decided to abort.

- Solutions solve problems:
    - The failure of participant is observed by timeouting the coordinator while waiting the participant respons:
        - If timeout occurs for waiting state we know that there are some participants in initial state or/and waiting state. Coordinator commands to abort.
        - If timeout occurs for pre-commit state we know that there are some participants in waiting state or/and in pre-commit state. Coordinator commands to abort.
        - If there is a participant which does not receive the abort message we are fine as upon the recovery, the participant decides depending on the state of other participants.

    - If timeout occurs there is need to find a new coordinator which verifies what is the state of the participants:
        - The new coordinator is state:
            -  Already committed:
                - That means that every other participant has received a `Prepare to Commit`.
                - Some participants may have committed.
                - Send Commit message to all participants (just in case they didn’t get it).
            - Not committed but received a Prepare message:
                - That means that all participants agreed to commit; some may have committed.
                - Send `Prepare to Commit` message to all participants (just in case they didn’t get it).
                - Wait for everyone to acknowledge; then `commit`.
            - Not yet received a Prepare message:
                - This means no participant has committed; some may have agreed.
                - Transaction can be `aborted` or the commit protocol can be `restarted`
- Disavantage:
    - Cost
    - Problems when the network gets partitioned:
        - Partition A: nodes that received Prepare message
            - Recovery coordinator for A: allows commit
        - Partition B: nodes that did not receive Prepare message
            - Recovery coordinator for B: aborts.
        
        `But when the network merges back, the system is inconsistent`
    - Not good when a crashed coordinator recovers:
        - It needs to find out that someone took over and stay quiet.
        - Otherwise it will mess up the protocol, leading to an inconsistent state.
    - Suppose
        - A coordinator sent a Prepare message to all participants.
        - All participants acknowledged the message.
        - BUT the coordinator died before it got all acknowledgements.

### 3.4 Raft protocol

> https://raft.github.io/
> https://www.geeksforgeeks.org/raft-consensus-algorithm/


* Some terms used to refer individual servers in a distributed system.
  * **Leader** – Only the server elected as leader can interact with the client. All other servers sync up themselves with the leader. At any point of time, there can be at most one leader(possibly 0, which we shall explain later)
  * **Follower** – Follower servers sync up their copy of data with that of the leader’s after every regular time intervals. When the leader server goes down(due to any reason), one of the followers can contest an election and become the leader.
  * **Candidate** – At the time of contesting an election to choose the leader server, the servers can ask other servers for votes. Hence, they are called candidates when they have requested votes. Initially, all servers are in the Candidate state.

    <p align="center">
    <img src="https://cdncontribute.geeksforgeeks.org/wp-content/uploads/multiple-server-labelled-raft-visual.png">
    </p>

* Raft is a consensus algorithm that is designed to be easy to understand. 
* It’s equivalent to Paxos in fault-tolerance and performance. The difference is that it’s decomposed into relatively independent subproblems, and it cleanly addresses all major pieces needed for practical systems.

* **How raft work ?**
  <p align="center">
    <img src="https://cdncontribute.geeksforgeeks.org/wp-content/uploads/node-status-transitions.png">
    </p>

* Only a leader can interact with the client; any request to the follower node is redirected to the leader node. A candidate can ask for votes to become the leader. A follower only responds to candidate(s) or the leader.

* To maintain these server status(es), the Raft algorithm divides time into small terms of arbitrary length. Each term is identified by a monotonically increasing number, called **term number**.

* This term number is maintained by every node and is passed while communications between nodes. Every term starts with an election to determine the new leader. The candidates ask for votes from other server nodes(followers) to gather majority. If the majority is gathered, the candidate becomes the leader for the current term. If no majority is established, the situation is called a **split vote** and the term ends with no leader. Hence, a term can have **at most** one leader.

* **Purpose of maintaining term number**
Following tasks are executed by observing the term number of each node:
  * Servers update their term number if their term number is less than the term numbers of other servers in the cluster. This means that when a new term starts, the term numbers are tallied with the leader or the candidate and are updated to match with the latest one(Leader’s)
  * Candidate or Leader demotes to the Follower state if their term number is out of date(less than others). If at any point of time, any other server has a higher term number, it can become the Leader immediately.
  * As we said earlier that the term number of the servers are also communicated, if a request is achieved with a stale term number, the said request is rejected. This basically means that a server node will not accept requests from server with lower term number

* Raft algorithm uses two types of Remote Procedure Calls(RPCs) to carry out the functions :
  * **RequestVotes** RPC is sent by the Candidate nodes to gather votes during an election
  * **AppendEntries** is used by the Leader node for replicating the log entries and also as a heartbeat mechanism to check if a server is still up. If heartbeat is responded back to, the server is up else, the server is down. Be noted that the heartbeats do not contain any log entries.

**Leader election**
* In order to maintain authority as a Leader of the cluster, the Leader node sends heartbeat to express dominion to other Follower nodes. A leader election takes place when a Follower node times out while waiting for a heartbeat from the Leader node. At this point of time, the timed out node changes it state to Candidate state, votes for itself and issues RequestVotes RPC to establish majority and attempt to become the Leader. The election can go the following three ways:

  * The Candidate node becomes the Leader by receiving the majority of votes from the cluster nodes. At this point of time, it updates its status to Leader and starts sending heartbeats to notify other servers of the new Leader.
  * The Candidate node fails to receive the majority of votes in the election and hence the term ends with no Leader. The Candidate node returns to the Follower state.
  * If the term number of the Candidate node requesting the votes is less than other Candidate nodes in the cluster, the AppendEntries RPC is rejected and other nodes retain their Candidate status. If the term number is greater, the Candidate node is elected as the new Leader.

<p align="center">
<img src="https://cdncontribute.geeksforgeeks.org/wp-content/uploads/raft-leader-election.png">
</p>

> Raft uses randomized election timeouts to ensure that split votes are rare and that they are resolved quickly. To prevent split votes in the first place, election timeouts are chosen randomly from a fixed interval (e.g., 150–300ms). This spreads out the servers so that in most cases only a single server will time out; it wins the election and sends heartbeats before any other servers time out. The same mechanism is used to handle split votes. Each candidate restarts its randomized election timeout at the start of an election, and it waits for that timeout to elapse before starting the next election; this reduces the likelihood of another split vote in the new election.

**Log Replication**

* For the sake of simplicity while explaining to the beginner level audience, we will restrict our scope to client making only write requests. Each request made by the client is stored in the Logs of the Leader. This log is then replicated to other nodes(Followers). Typically, a log entry contains the following three information :

  * **Command** specified by the client to execute
  * **Index** to identify the position of entry in the log of the node. The index is 1-based(starts from 1).
  * **Term Number** to ascertain the time of entry of the command.

* The Leader node fires AppendEntries RPCs to all other servers(Followers) to sync/match up their logs with the current Leader.The Leader keeps sending the RPCs until all the Followers safely replicate the new entry in their logs.

* There is a concept of entry commit in the algorithm. When the majority of the servers in the cluster successfully copy the new entries in their logs, it is considered committed. At this point, the Leader also commits the entry in its log to show that it has been successfully replicated. All the previous entries in the log are also considered committed due to obvious reasons. After the entry is committed, the leader executes the entry and responds back with the result to the client.

* It should be noted that these entries are executed in the order they are received.

* If two entries in different logs(Leader’s and Followers’) have identical index and term, they are guaranteed to store the same command and the logs are identical upto that point(Index).

> In Raft, the leader handles inconsistencies by forcing the followers’ logs to duplicate its own. This means that conflicting entries in follower logs will be overwritten with entries from the leader’s log.

* The Leader node will look for the last matched index number in the Leader and Follower, it will then overwrite any extra entries further that point(index number) with the new entries supplied by the Leader. This helps in Log matching the Follower with the Leader. The AppendEntries RPC will iteratively send the RPCs with reduced Index Numbers so that a match is found. When the match is found, the RPC succeeds.

**Safety**

* In order to maintain consistency and same set of server nodes, it is ensured by the Raft consensus algorithm that the leader will have all the entries from the previous terms committed in its log.

* During a leader election, the RequestVote RPC also contains information about the candidate’s log(like term number) to figure out which one is the latest. If the candidate requesting the vote has less updated data than the Follower from which it is requesting vote, the Follower simply doesn’t vote for the said candidate. The following excerpt from the original Raft paper clears it in a similar and profound way.

> Raft determines which of two logs is more up-to-date by comparing the index and term of the last entries in the logs. If the logs have last entries with different terms, then the log with the later term is more up-to-date. If the logs end with the same term, then whichever log is longer is more up-to-date.

**Rules for Safety in the Raft protocol**

The Raft protocol guarantees the following safety against consensus malfunction by virtue of its design :

  * **Leader election safety** – At most one leader per term)
  * **Log Matching safety**(If multiple logs have an entry with the same index and term, then those logs are guaranteed to be identical in all entries up through to the given index.
  * **Leader completeness** – The log entries committed in a given term will always appear in the logs of the leaders following the said term)
  * **State Machine safety** – If a server has applied a particular log entry to its state machine, then no other server in the server cluster can apply a different command for the same log.
  * **Leader is Append-only** – A leader node(server) can only append(no other operations like overwrite, delete, update are permitted) new commands to its log
  * **Follower node crash** – When the follower node crashes, all the requests sent to the crashed node are ignored. Further, the crashed node can’t take part in the leader election for obvious reasons. When the node restarts, it syncs up its log with the leader node

**Cluster membership and Joint Consensus**

* When the status of nodes in the cluster changes(cluster configuration changes), the system becomes susceptible to faults which can break the system. So, to prevent this, Raft uses what is known as a two phase approach to change the cluster membership. So, in this approach, the cluster first changes to an intermediate state(known as joint consensus) before achieving the new cluster membership configuration. Joint consensus makes the system available to respond to client requests even when the transition between configurations is taking place. Thus, increasing the availability of the distributed system, which is a main aim.

**What are its advantages/Features**

  * The Raft protocol is designed to be easily understandable considering that the most popular way to achieve consensus on distributed systems was the Paxos algorithm, which was very hard to understand and implement. Anyoone with basic knowledge and common sense can understand major parts of the protocol and the research paper published by Diego Ongaro and John Ousterhout
  * It is comparatively easy to implement than other alternatives, primarily the Paxos, because of a more targeted use case segment, assumptions about the distributed system. Many open source implementations of the Raft are available on the internet. Some are in Go, C++, Java
  * The Raft protocol has been decomposed into smaller subproblems which can be tackled relatively independently for better understanding, implementation, debugging, optimizing performance for a more specific use case
  * The distributed system following the Raft consensus protocol will remain operational even when minority of the servers fail. For example, if we have a 5 server node cluster, if 2 nodes fail, the system can still operate.
  * The leader election mechanism employed in the Raft is so designed that one node will always gain the majority of votes within a maximum of 2 terms.
  * The Raft employs RPC(remote procedure calls) to request votes and sync up the cluster(using AppendEntries). So, the load of the calls does not fall on the leader node in the cluster.
  * Raft was designed recently, so it employs modern concepts which were not yet understood at the time of the formulation of the Paxos and similar protocols.
  * Any node in the cluster can become the leader. So, it has a certain degree of fairness.
  * Many different open source implementations for different use cases are already out there on GitHub and related places
  * Companies like MongoDB, HashiCorp, etc. are using it!

**Raft Alternatives**

* Paxos – Variants :- multi-paxos, cheap paxos, fast paxos, generalised paxos
* Practical Byzantine Fault Tolerance algorithm (PBFT)
* Proof-of-Stake algorithm (PoS)
* Delegated Proof-of-Stake algorithm (DPoS)

**Limitations**

* Raft is strictly single Leader protocol. Too much traffic can choke the system. Some variants of Paxos algorithm exist that address this bottleneck.
* There are a lot of assumptions considered to be acting, like non-occurrence of Byzantine failures, which sort of reduces the real life applicability.
* Raft is a more specialized approach towards a subset of problems which arise in achieving consensus.
* Cheap-paxos(a variant of Paxos), can work even when there is only one node functioning in the server cluster. To generalise, K+1 replicated servers can tolerate shutting down of/ fault in K servers.


