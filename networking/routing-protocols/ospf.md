# OSPF - Open Shortest Path First

OSPF is a link state routing protocol, is classless by design, it scales fast and supports authentication for neighbors.

Its administrative distance is 110.  

## OSPF - under the hood

OSPF uses three databases:
- one for adjacencies that holds information about neighbors
- one for link-state messages that holds the topology of the area (**LSDB**)
- one for routes that holds information about routes learned via OSPF (**Forwarding DB**)

Since OSPF messages are encapsulated in IP packets and fraes but are not encapsulated in segments, the value of the protocol field of the IP packet is **89** which makes OSPF, as the other routing protocols, easily distinguishable.  

OSPF packets are sent to neighbors as multicast messages to IPs **224.0.0.5** (_AllSPFRouters_) and **224.0.0.6** (_AllDRouters_) and MAC frames **01-00-5e-00-00-05** and **01-00-5e-00-00-06**.  

### OSPF messages - Link-State Packets

|Type|Packet|Description|
|----|------|-----------|
|1|Hello|Find neighbors and build adjancencies|
|2|Database Description (DBD)|Sync DBs with adjacent routers|
|3|Link-state Request (LSR)|Enquire for specific datas amogn routers|
|4|Link-state Update (LSU)|Answer to LSR, or spontaneous, send own link state to specific router or all routers|
|5|Link-state Acknowledgement (LSAck)|Acknowledge delivery of packets|

Each LSU packet may contain one or more LSAs (Link-state Advertisements) holding specific routing information:

|LSA type|Description|
|--------|-----------|
|1|Router LSAs|
|2|Network LSAs|
|3 or 4|Summary LSAs|
|5|Autonomous LSAs|
|6|Multicast OSPF LSAs|
|7|Defined for Not-So-Stubby-Areas|
|8|External Attributes LSA for BGP|
|9, 10, 11|Opaque LSAs|

**Hello** packets are essentials to build adjacencies with other OSPF routers and to elect **DRs** and **BDRs** routers.  
Hello packets are sent every 10 seconds (**Hello-interval**) on multiaccess networks (Ethernet) and point-to-point (HDLC or PPP) and every 30 seconds on NBMA-Non Broadcast multiaccess.

If a router being part of OSPF adjacencies does not send for a time named **Dead-interval**, which is by default 4 times the defined Hello-interval, it is then removed from neighbors' LSBDs.  

### OSPF algorithm

Brief and fast overview of the algorithm:
- each router receives LSAs from neighbors and builds a local LSDB
- based on the information contained in the local LSDB, the algorithm build an SPF tree which is the complete topology of OSPF 'network'
- from that map, the algorithm extracts the best routes to reach each destination and put them in the local Forwarding database

### OSPF areas

OSPF areas are important because they provide a sort of segmentation.  
For example, a network topology is divided into three OSPF areas. Initially all routers in all areas run the SPF algorithm to build a fowarding database.  
After some time, a topology change occurs in one specific area, well only the routers of that area will run again the algorithm to re-build a forwarding database.  
Note: this is true unless major changes occur which alter the summary routes sent across areas.  

Area 0 is considered to be the _backbone Area_ to which all non-0 areas need to be directly attached to, either physically or virtually.  
Routers with interfaces in more than one area are called **ABR** (Area Border Router). ABRs exchange summary routes information among them in the like of **Distance Vector** protocols.  

**Major pros**:
- small routing tables
- less overhead while computing LSUs
- less SPF algorithm runs

### OSPF - from 0 to Full

As soon as a router is OSPF enabled it sends Hello packets to discover neighbors, build adjacencies, exchange routes information and build its own databases.  

From non-OSPF router to active-OSPF router, there are 7 steps a router must go through:  

**--Establish Neighbor Adjacencies--**  
|#|State|Description|
|-|-----|-----------|
|1|**Down State**| No hello packets are received. The routers starts to send Hello packets. Transition to Init State.|
|2|**Init State**| Hello packets are received from neighbors. They contain the sending router's Router ID. Transition to Two-Way State.|
|3|**Two-Way State**| On Ethernet links, DRs and BDRs are elected. Transition to ExStart State.|  

**--Synchronize OSPF Databases--**  
|#|State|Description|
|-|-----|-----------|
|4|**ExStart State**|Negotiate master/slave relationship and DBD packet sequence number. The master initiates the DBD packet exchange|
|5|**Exchange State**|Routers exchange DBD packets. If additional router information is required then transition to Loading, otherwise transition to Full|
|6|**Loading State**|LSRs and LSUs are used to gain additional route information. Routes are processed using the SPF algorithm. Transition to Full state|
|7|**Full State**|Routers have converged|
