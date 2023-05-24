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

**Hello** packets are essentials to build adjacencies with other OSPF routers and to elect DRs and BDRs routers.  
Hello packets are sent every 10 seconds (**Hello-interval**) on multiaccess networks (Ethernet) and point-to-point (HDLC or PPP) and every 30 seconds on NBMA-Non Broadcast multiaccess.

If a router being part of OSPF adjacencies does not send for a time, which is by default 4 times the defined Hello-interval (**Dead-interval**), is removed from neighbors' LSBD.

