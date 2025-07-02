## Contents
	- ((68614990-3dbd-4959-be19-5f9e031313f0))
		- ((68614af0-123a-46f0-bcbd-bc74b090269d))
			- ((68614d97-e9c4-4b37-9335-5f0c5343099a))
			- ((68615015-b31d-497b-af8b-42cf6ffb7c9c))
			- ((686153da-24d7-471a-adc0-fed5ca2dd341))
		- ((68623c4a-2442-4469-9b92-56be02484c63))
			- ((686248a8-ef0f-4c27-b07f-9726c5e69286))
			- ((68624d65-6665-41e6-a4e7-0e57d00077f0))
		- ((6862a1ba-2a04-4cec-8bcd-a4c6dc84c1f0))
		- ((6862a860-2a7c-4a74-8e9a-8156813ca8b9))
	- ((6858aace-5258-46d5-a174-b5a66b83dd01))
		- ((6858ac12-7876-40d7-abfa-36e7a54b46a8))
		- ((6858ae64-e3b7-445f-a2ee-eaf46585869e))
		- ((6858b0e0-689c-4a07-921c-b8e22a6a851d))
		- ((6858b605-a9b9-4a28-b872-9af2603f7f51))
		- ((685eb786-0397-4e80-b466-f9ed2f2d511f))
		- ((685eba40-848b-4303-ab56-86429e511bbc))
		- ((685ebcd5-758e-4a06-bc12-3c0f218210ac))
		- ((6858aa8c-6f04-43d3-9452-cb14db7a74cf))
		- ((685f4077-2835-43e5-a212-34ca89ee8af7))
		- ((685f4327-261f-43bf-9587-41c848881460))
- # Introduction
- # Distributed Objects and File System
	- ## Remote Procedure Call
		- **Implementation**
			- The client calls the client stub. This is a local procedural call.
			  logseq.order-list-type:: number
			- The client stub packs the procedure parameters in to a message (marshalling) and makes a system call to send the message. 
			  logseq.order-list-type:: number
			- The clients host os sends the message to remote server's machine
			  logseq.order-list-type:: number
			- The dispatcher selects one of the server stub procedures according to procedure identifier in request message
			  logseq.order-list-type:: number
			- The server stub unmarshalls the parameters from the message and calls the corresponding service procedure
			  logseq.order-list-type:: number
			- The service procedure executes the operations and returns the result to server stub which then marshalls the result into a message
			  logseq.order-list-type:: number
			- the server stub then hands the message to transport layer
			  logseq.order-list-type:: number
			- The transport layer than sends the message to client's transport layer which hands the message to client stub
			  logseq.order-list-type:: number
			- The client stub unmarshalls the message to obtain the result and returns to the caller
			  logseq.order-list-type:: number
		- **Advantages**
			- can be used in distributes as well as local environments
			- hides internal message passing mechanism
			- supports process as well as thread oriented model
			- requires only minimal efforts to rewrite and redevelop the code
			- provides abstraction
		- **Disadvantages**
			- not always suited for transferring large amount of data
			- highly vulnerable to failures
			- no uniform standard
	- ## Remote Method Invocation
		- similar to RPC but extended to objects where a calling object can invoke method on a potentially remote object
		- ### Components
			- **Proxy/Client Stub**
				- a proxy for an actual object that resides on a remote server
				- it lives in the client side and acts as an actual object. when a method is called on the stub, it forwards the request to actual object on remote server
				- When the client invokes a method on the stub, the stub marshalls the method's name, parameters, other metadatas into a message and sends it across the network and waits for the response
				- upon receiving the response, it unmarshalls the return values from the response message and returns the result to the caller
			- **Remote Reference Module**
				- it is a part of the runtime system that manages remote references, interprets remote reference to route method call to actual object, handle communication logic, connection handling stc
				- A remote reference is a special type of reference that is used to identify actual object on a remote system. It contains enough information like ip address of server, port number of the server process, object identifier to uniquely identify the object required to reach actual object
				- The client stub uses RMM to actually send the message request to remote server.
				- On server side, the RMM receives the request message, and sends it to the dispatcher using the object id
			- **Dispatcher**
				- A server has one dispatcher for each class
				- receives remote call request and identifies which method is being called based on method ID and forwards the call to the skeleton
			- **Skeleton**
				- acts as a server side stub
				- it unmarshalls the arguments in request message and invokes corresponding methods on servants
				- it then waits for the invocation to finish after which it marshalls the result in a reply message
			- **Servants**
				- instance of actual class responsible for actually handling the remote request
	- ## Distributed File System
		- file system over a network that allows users to access files stored across multiple servers as if they were stored locally
		- Eg: Andrew File System, Google File System, SUN's NFS etc
		- **Characteristics**
			- Transparent access to remote files
			- location transparency
			- concurrency control
			- fault tolerance
		- **Why**
			- scalability
			- fault tolerance
			- data sharing
			- performance
			- decentralization
		- ### Architecture
			- **Client Module**
				- runs on each client's computer and provides integrated servers through a single API to application programs
				- holds info about flat-file directory server processes
				- implements cache to improve performance
				- provides interactions between user and the file system
				- provides syscalls for open, read, write etc
			- **Directory Service**
				- mapping between file names and their Unique File Identifiers (UFIDs)
				- functions needed to generate new directories, add new files to directories, obtain UFIDs from directories
			- **Flat File Service**
				- implementations of operations on the content of the files
				- responsible for reading writing files
- # Time and State in Distributed System
  id:: 68614990-3dbd-4959-be19-5f9e031313f0
	- no concept of a global clock in distributed system
	- this causes problems since lots of operation are dependent on the precise time or the relative ordering of event on different nodes in a system matters
	- we can use physical clock synchronization techniques to sync. the physical clocks on each nodes to some degree of accuracy or if only the relative ordering of events are of concern then we can use logical clock to maintain logical time
	- ## Physical Clocks and Synchronization
	  id:: 68614af0-123a-46f0-bcbd-bc74b090269d
		- physical clocks are devices that tells the time using crystal oscillator
		- **External synchronization** is the process of using external source of time to synchronize the process's time
		- **Internal synchronization** is the process of synchronizing the clocks of nodes in a system among each
		- The general form of internal clock synchronization is as follows: one process sends its time **t** in an message **m** to another process. The receiving process could then set its clock to $time(t) + T_{trans}$ where $T_{trans}$ is the time taken to transmit the message **m**.
		- Since the $T_{trans}$ is not a fixed measurement and subject to significant variations, the receiving process could instead set the time to $time(t) + \frac{max + min}{2}$, where max and min are the upper and lower bound on the time taken to send any message.
	- ## Cristian's Method
	  id:: 68614d97-e9c4-4b37-9335-5f0c5343099a
		- external clock synchronization technique
		- client that wants to corrects its time makes request to server. the server then send its time to the client
		- **Algorithm**
			- A process on a client $p$ requests the time in a message $m_r$ to server $S$ at time $T_0$ via RPC.
			- It receives the server time $T_{server}$ on message $m_t$ from server at time $T_1$
			- client $p$ can then set its time to: $T_{server} + \frac{T_1 - T_0}{2}$
		- The single time server could fail rendering the synchronization temporarily unavailable.
	- ## Berkeley's Method
	  id:: 68615015-b31d-497b-af8b-42cf6ffb7c9c
		- internal physical clock synchronization technique
		- A coordinator, called the **master**, is chosen among the nodes that periodically polls other computers on the network, called the **slaves**, whose clocks are to be synchronized. After the slaves send back their time, the master calculates the average time after adjusting for transmission using round trip time. It also includes its own time.
		- The **master** sends back the adjustments to each **slaves** instead of corrected current time.
		- **Algorithm**
			- A **master** is chosen using the election process
			- the master sends requests for timestamps to all other nodes including itself
			- slave respond back with their timestamp
			- master observers the $RTT$ of the message and estimates the actual slave time.
				- $Adjusted\ Time\ =\ Slave's\ reported\ time + \frac{RTT}{2}$
			- Master computes the average time
				- $T_{avg} = \frac{T_1 + T_2 + ... + T_n}{n}$
				- master may ignore the timestamps with $RTT$ greater than some predefined limit
			- Master calculates the adjustments for each slaves and itself and sends them
			- Each slave applies the adjustments immediately
	- ## Network Time Protocol
	  id:: 686153da-24d7-471a-adc0-fed5ca2dd341
		- global, robust and scalable way to synchronize the clocks of devices across the internet
		- defines an architecture for time service and a protocol to distribute the time information
		- **enables clients across the internet to be synchronized**
			- for this NTP uses hierarchical system of time servers, organized in layers called strata. The highly precise time sources like GPS clocks, atomic clocks, radio clocks are in Stratum 0. Stratum 1 time servers are those that are directly connected to Stratum 0 source. The stratum level denote how far the node is from the stratum 0 time sources
			- clients communicate with one or more NTP servers and use algorithms to calculate time offsets and network delay
		- **provides reliable service**
			- in case a device loses connection with an NTP server, it still maintains reasonably accurate time
			- NTP uses local clock oscillator and clock discipline algorithm to calculate the drift rate of local clock and adjust accordingly
		- **regular synchronization to correct drift**
		- **protection against interference with time servers**
			- NTP uses cryptographic authentication using PKI or symmetric keys
			- clients can check for suspicious time shifts, high delays, inconsistent data
			- using multiple NTP servers to eliminate outliers
		- ### Server Synchronization
			- In a high speed LAN network, NTP may use **Multicast** mode where one or more servers periodically multicasts time to the servers running in other computers. This mode can only achieve small accuracy (due to one way communication), but one that is sufficient for multiple purposes
			- In **Procedure-Call** mode, one server accepts the time requests from other computers and replies with its current timestamp. Achieves higher accuracy than multicast mode
			- In **Peer-to-Peer** mode, two or more servers on same strata exchange time information with each other to achieve more accuracy.
		- ### Clock Synchronization
			- NTP synchronizes client's clock with that of a server using a 4-timestamp exchange after which it calculates
				- Clock Offset($\theta$): difference between server's clock and client's clock
				- Round-trip delay($\delta$): how long it took for sending/receiving message
			- Client **sends request** to server
			  logseq.order-list-type:: number
				- client notes the timestamp($T_1$) when it send the request
				  logseq.order-list-type:: number
				- the request message packet is sent to server over UDP (port 123)
				  logseq.order-list-type:: number
			- Server **receives the requests**
			  logseq.order-list-type:: number
				- server also notes the timestamp($T_2$) when it received the request
				  logseq.order-list-type:: number
			- Server **sends response** to client
			  logseq.order-list-type:: number
				- the server records the time($T_3$) of sending the response and sends all three timestamps($T_1, T_2, T_3$) in reply
				  logseq.order-list-type:: number
			- Client **receives the response**
			  logseq.order-list-type:: number
				- client again notes the timestamp($T_4$) when it received the response
				  logseq.order-list-type:: number
			- **Compute** the $\theta$ and $\delta$
			  logseq.order-list-type:: number
				- $\theta = \frac{(T_2 - T_1) + (T_3 - T_4)}{2}$
				  logseq.order-list-type:: number
				- $\delta = (T_4 - T_1) - (T_3 - T_2)$
				  logseq.order-list-type:: number
			- **Adjust** the clock
			  logseq.order-list-type:: number
				- the client can now adjust the clock with new time = current clock time + $\theta$
				  logseq.order-list-type:: number
				- gradual adjustment, called the slewing, may be preferred to avoid time jumps
				  logseq.order-list-type:: number
	- ## Logical Clock
	  id:: 68623c4a-2442-4469-9b92-56be02484c63
		- virtual clock that records the relative ordering of events
		- assigns a timestamp to each event
		- **Why logical clocks**
			- physical clocks can't be perfectly synchronized
			- physical clocks can't capture causality
			- not all systems have physical clocks
	- ## Lamport's Logical Clock
	  id:: 686248a8-ef0f-4c27-b07f-9726c5e69286
		- Ordering is based on **two simple rules**
			- if two events happened in the same process, then they occurred in order observed following the process
			- whenever a message is sent between the process, the sending of the message occurred before the receiving of the message
		- Ordering in lamport is based on happened-before relation (denoted $\rightarrow$)
			- $a \rightarrow b$: if a and b are events in the same process and $a$ occurred before $b$
			- $a \rightarrow b$: if $a$ is the event of sending a message $m$ in a process and $b$ is the even of receiving the same message $m$ in another process
			- if $a \rightarrow b$ and $b \rightarrow c$ then $a \rightarrow c$
		- **Algorithm**
			- each event issued at process $P_i$ is timestamped with value obtained after incrementing the local clock $C_{Pi} :=  C_{Pi} + 1$
			- when $a$ is the event of sending the message $m$ from process $P_i$, the timestamp $t_m = C_{Pi}(a)$ is included in the message $m$
			- on receiving message $m$ by process $P_j$, its logical clock $C_{Pj}$ is updated as follows
				- $C_{Pj} := max(C_{Pj}, t_m)$
		- **Problems**
			- imposes only partial order on events, which is insufficient for certain applications
			- just by looking at the timestamps we cannot say whether two events are causal or concurrent
	- ## Vector Clock
	  id:: 68624d65-6665-41e6-a4e7-0e57d00077f0
		- assigns a vector of timestamps for each events, one for each process in the system
		- **Algorithm**
			- initially set $V_i[j] = 0, for\ i, j = 1, 2, .. N$
			- just before $P_i$ timestamps an event, it sets $V_i[i] = V_i[i] + 1$
			- $P_i$ includes the value $t = V_i$ in every message it sends
			- when $P_i$ receives a timestamp $t$ in  message it sets
				- $V_i[j] = max(V_i[j], t[j]), for j = 1, 2, ... N$
		- for any two vector timestamps $u\ and\ v$, we have
			- $u = v$ if and only if $\forall i, u[i] = v[i]$
			- $u \le v$ if and only if $\forall i, u[i] \le v[i]$
			- $u \lt v$ if and only if $u \le v\ and\ u \ne v$
			- $u \parallel v$ if and only if $\neg (u \lt v) \land \neg(v \lt u)$
	- ## Global State
	  id:: 6862a1ba-2a04-4cec-8bcd-a4c6dc84c1f0
		- The set of all the local states and the set of states of communicating channels.
		- global state is required for a lot of things like taking a snapshot of the system, reasoning about the system, decision making, fault tolerance, debugging etc.
		- but there is no easy way to acquire a global state due to lack of global clock or central point of control
		- **Definations**
			- $LS_i$ is the local state of process $P_i$
				- it includes all the messages sent and received by the process as well as other informations
			- Global state of the system is the collection of the local states
				- $GS = \{LS_1, LS_2, LS_3, ... LS_n\}$
			- $\textbf{send}(m^k_{ij})$ denotes the event of sending the message $m^k_{ij}$ from $P_i$ to $P_j$
			- $\textbf{rec}(m^k_{ij})$ denotes the event of receiving the message $m^k_{ij}$ by $P_j$
			- $send(m^k_{ij}) \in LS_i$ if and only if the sending event occurred before the local state was recorded
			- $rec(m^k_{ij}) \in LS_j$ if and only if the receiving event occurred before the local state was recorded
			- $\textbf{transit}(LS_i, LS_j) = \{ m^k_{ij} | send(m^k_{ij}) \in LS_i \land  rec(m^k_{ij}) \notin LS_j \}$
			- $\textbf{inconsistent}(LS_i, LS_j) = \{ m^k_{ij} | send(m^k_{ij}) \notin LS_i \land  rec(m^k_{ij}) \in LS_j \}$
		- **Consistent**
			- A global state $GS = \{ LS_1, LS_2, ..., LS_n \}$ is consistent if and only if:
				- $\forall i, \forall j: 1 \le i, j \le n, \textbf{inconsistent}(LS_i, LS_j) = \phi$
		- **Transitless**
			- A global state $GS = \{ LS_1, LS_2, ..., LS_n \}$ is transitless if and only if:
				- $\forall i, \forall j: 1 \le i, j \le n, \textbf{transit}(LS_i, LS_j) = \phi$
		- A global state is strongly consistent if it is consistent and transitless.
	- ## Chandy Lamport Distributed Snapshot Algorithm
	  id:: 6862a860-2a7c-4a74-8e9a-8156813ca8b9
		- A snapshot is record of the global state at a specific point in time
		- **Algorithm**
			- Initiation
			  logseq.order-list-type:: number
				- $P_i$ records it own local state
				  logseq.order-list-type:: number
				- for any outgoing channel, $P_i$ sends a **marker** message before sending any other message
				  logseq.order-list-type:: number
			- Marker Receiving Rule
			  logseq.order-list-type:: number
				- When a process $P_j$ receives a **marker** message on channel $C_{i \rightarrow j }$
				  logseq.order-list-type:: number
				- If its the first marker message received on that channel
				  logseq.order-list-type:: number
					- it records its own local state
					  logseq.order-list-type:: number
					- records the state of channel $C_{i \rightarrow j }$ as empty
					  logseq.order-list-type:: number
					- for each incoming channel, start recording the message until a marker is received
					  logseq.order-list-type:: number
					- send marker message on all outgoing channel
					  logseq.order-list-type:: number
				- Marker received on channel where marker was already received
				  logseq.order-list-type:: number
					- stop recording message on that channel
					  logseq.order-list-type:: number
					- the set of recorded message is the channel state
					  logseq.order-list-type:: number
			- Continue this until all processes have recorded their local state and all channel have been accounted for
			  logseq.order-list-type:: number
- # Coordination and Agreement
  id:: 6858aace-5258-46d5-a174-b5a66b83dd01
	- ## Mutual Exclusion
	  id:: 6858ac12-7876-40d7-abfa-36e7a54b46a8
		- ensure that only one process at a time can access the shared resource
		- more difficult in distributed system due to no shared memory or clock
		- why mutual exclusion?
			- prevent data inconsistency
			- maintain integrity
			- avoid race conditions
			- coordinate access to resource
		- Requirements
			- mutual exclusion (obviously)
			- progress (if no one in CS and some want to enter, one of them must be allowed to enter)
			- bounded waiting
			- no deadlock
			- no starvation
			- fault tolerance
		- There are two types of algorithm for implementing mutual exclusion in distributed system.
			- Non-token based
			- Token based
	- ## Central Coordination Algorithm
	  id:: 6858ae64-e3b7-445f-a2ee-eaf46585869e
		- simple non-token based algorithm for achieving mutual exclusion
		- A central coordination is present in the distributed system. Only it can grant or deny access to the critical section
		- **Algorithm**
			- Let, $P_1, P_2, ... P_n$ be the processes and $C$ be the coordination
			- Request Phase
				- A process $P_i$ sends a REQUEST message to the coordination when it wants to enter CS
			- Grant Phase
				- If no other processes are in CS, the coordination sends the GRANT message back to the requesting process.
				- If the CS is busy, the request is queued.
			- CS Execution
				- After receiving GRANT from the coordination, the process enters the CS
			- Release Phase
				- after process finishes its work in CS, it sends a RELEASE message to the coordinator. The coordinator then grants permission to next process in there are any in the queue
		- Advantages
			- simple and easy
			- requires only 3 messages per use of CS
			- fairness
		- Disadvantages
			- coordinator can become performance bottleneck
			- coordinator is a point of failure
	- ## Ricart-Agrawala Non Token Based Algorithm
	  id:: 6858b0e0-689c-4a07-921c-b8e22a6a851d
		- based on distributed agreement not central coordinator
		- the process that wants to enter CS sends request message to all other processes and only when it has received messages from all other processes can it enter CS
		- the request message is of the form $[T(P_i), i]$, where $T(P_i)$ is the process's timestamp and $i$ is its identifier
		- Each process is in one of three state with respect to CS: Released, Requested, Held
		- **Algorithms/Rules**
			- **Rule for Process Initialization**
				- performed by all processes
				- State($P_i$) := RELEASED
			- **Rule for access request**
				- when process $P_i$ wants to access the CS. it does the following
				- State($P_i$) := REQUESTED
				- $P_i$ sends a request message to all other processes
				- it then waits until it receives replies from all other processes
			- **Rule for executing the CS**
				- when process $P_i$ receives replies from all other process, it
				- State($P_i$) := HELD
				- $P_i$ enters CS
			- **Rule for handling incoming requests**
				- when process $P_i$ receives a request message from process $P_j$ (the message being $[T(P_j), j]$)
				- if $P_i$ is in the CS, that is State($P_i$) := HELD then queue the request without replying
				- if $P_i$ has also requested for the CS (State($P_i$) := REQUESTED) then it checks the priority. if $[T(P_i), i] \lt [T(P_j), j]$ (basically if $P_i$ made the request earlier than $P_j$) then also the request is queued without replying
				- otherwise reply immediately to $P_j$
			- **Rule for releasing CS**
				- after $P_i$ finishes its work in CS
				- State($P_i$) := RELEASED
				- reply to all queued requests
		- **Problems**
			- expensive in terms of message traffic: 2(n-1) messages required for each CS entry
			- failure of any process requires special recovery measures
	- ## Ricart-Agrawala Token Based Algorithm
	  id:: 6858b605-a9b9-4a28-b872-9af2603f7f51
		- process is allowed to enter CS when it has the token
		- when a process wants to enter CS (and it doesnt have the token) it sends request(consisting of process's timestamp) to all other processes. only after getting the token as a reply will the process enter the critical region
		- **Algorithm**
			- **Rule for process initiaization**
				- State($P_i$) = NO_TOKEN, for all processes except on process $P_x$
				- whose State($P_x$) = TOKEN_PRESENT
				- token[k] initialized 0 for all elements k = 1..n
				- requestPi[k] initialized 0 for all processes $P_i$  and elements k=1..n
			- **Rule for access request and execution of the CS**
				- if State($P_i$) = NO_TOKEN then $P_i$ sends a request message to all processes
				- $P_i$ then waits for token
				- After receiving the token, it enters the CS by setting State($P_i$) = TOKEN_HELD
			- **Rule for handling incoming request**
				- When a process $P_i$ receives a request from process $P_j$, it sets
				- request($P_i$[j]) = max(request($P_i$[j], T($P_j$))
				- if State($P_i$) == TOKEN_PRESENT, it releases the token
			- **Rule for releasing CS**
				- When $P_i$ finishes its work in the CS, its sets
				- State($P_i$) = TOKEN_PRESENT
				- for k = [i+1, i+2, ...n, 1, 2, ..., i-2, i-1]
				- if request($P_i$[k]) > token[k] then
					- State($P_i$) = NO_TOKEN
					- token[i] = C($P_i$)
					- $P_i$ sends token to process $P_k$
	- ## Token Ring Algorithm
	  id:: 685eb786-0397-4e80-b466-f9ed2f2d511f
		- The n process are arranged in a logical ring with each process having the address of next process
		- **Algorithm**
			- The token is first given to a random process
			- when a process wants to enter a CS, it waits for the token. Upon receiving the token, it retains it and enters the CS. After completing its work in the CS, it passes the token to the next process in the ring
			- When a process receives a token but doesn't require to enter the CS, it immediately passes the token to the next process
	- ## Bully Algorithm
	  id:: 685eba40-848b-4303-ab56-86429e511bbc
		- All processes start with the State ELECTION-OFF
		- **Rule for election process initiator**
			- When a process triggers an election message (after detecting the failure of current coordinator) or when it receives the election message for the first time
			- State($P_i$) = ELECTION-ON
			- it then sends election message to all other processes with identifier higher than its and waits for an answer message
			- if it gets no answer before a timeout, then $P_i$ is the coordinator so it sends coordinator message to all processes
			- if it get an answer, it then waits for a coordinator message. If the coordinator messages doesn't arrive before a timeout, it restarts the election process
		- **Rule for handling incoming election message**
			- When a process $P_i$ receives an election message from $P_j$
			- it replies with an answer message to $P_j$
			- if its State == ELECTION-OFF, it starts the election process
	- ## Ring Based Election Algorithm
	  id:: 685ebcd5-758e-4a06-bc12-3c0f218210ac
		- Initially each process's State = NON-PARTICIPANT
		- **Rule for election process initiator**
			- When process $P_i$ notices that the coordinator is down, it triggers the election process
			- State($P_i$) = PARTICIPANT
			- $P_i$ sends the election message with its identifier to its neighbor
		- **Rule for handling incoming election message**
			- when process $P_j$ receives an election message
			- if message.id > j, it forwards the election message to its neighbor and sets the State($P_j$ ) = PARTICIPANT
			- if message.id < j then
				- if State($P_j$) = NON-PARTICIPANT, it forwards the election message after changing message.id = j. It also changes its State to PARTICIPANT
				- else if $P_j$'s state is already PARTICIPANT, it discards or ignores the message
			- if message.id == j, the $P_j$ is the coordinator. it sends the ELECTED message with id = j to its neighbor and State($P_j$) = NON-PARTICIPANT
		- **Rule for handling incoming ELECTED message**
			- when process $P_i$ receives an ELECTED message
			- if message.id != i, it forwards the message
			- else the election process ends
	- ## Unicast Communication
	  id:: 6858aa8c-6f04-43d3-9452-cb14db7a74cf
		- message sent from exactly one process to another single process
		- can be categorized into two types
		- **Best Effort**
			- if message is delivered, it is guaranteed to be intact
			- there is no guarantee about reliability
			- no retries, acknowledgement, no error correction
		- **Reliable**
			- guarantees the delivery of message
			- in case of failure, there are retries, acknowledgement receipt, buffer and resent
	- ## Broadcast and Multicast
	  id:: 685f4077-2835-43e5-a212-34ca89ee8af7
		- **Broadcast**
			- one process sends to all other processes
		- **Multicast**
			- one process sends to selected group of processes
			- used in group coordination, replicated databases, voting and consensus algorithms
			- often  it is necessary to send the same information to multiple nodes in a distributed coordination tasks (like Leader Election, Mutual Exclusion, Consensus, Atomic Broadcast). Hence Multicast is important
		- **Multicast Requires Coordination and Agreement**
			- **Multiple recipients = Shared State**
			  id:: 685f41a8-7c33-4ca1-8f00-a0d7ca91203b
				- when message is sent to multiple processes, they must agree on what the message was, the order of the messages and how to act on it
			- **Challenges**
				- Message Lost: Some processes may get the message while some may not
				- Out Of Order Delivery: Different processes may receive message in different order
				- Duplicates: same message may be delivered more than once
				- Process Failures: some processes may crash during message exchange
			- These things require coordination (for handling failures) and agreement (for maintaining consistent state)
		- **Why Multicast Communication?**
			- efficiency, consistency, simplicity, scalability
	- ## Consensus
	  id:: 685f4327-261f-43bf-9587-41c848881460
		- task of getting all the processes to agree on a specific value based on voting
		- the value has to be one of the ones submitted by the processes
		- **Requirements**
			- Termination: Every correct process must decide on a value eventually
			- Agreement: No two correct processes decides on two different values
			- Validity: If a process decided on value v, then v must have been proposed by some process
		- **Why Consensus**
			- to maintain a consistent state about some data across the distributed system. this includes the same updates applied to the data on all the nodes
			- to elect leader for coordination
			- to work correctly even in case of failures
			- to agree on the order of events/updates
			- to maintain atomicity
- #bct #semester-seven