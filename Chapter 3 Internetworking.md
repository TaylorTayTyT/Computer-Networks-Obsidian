**Problem: Not All Networks are Directly Connected**

To build a global network, we need a way to interconnect different types of links and multi-access networks - this is called internetworking. 

We can divide this problem up into subproblems. Firstly, we need a way to interconnect links. Devices that interconnect links of the same type are often called switches, or layer 2 (L2) switches. 

The point of a switch is to take packets that arrive on an input and forward them to the right output. 

*Routers* - or *gateways* - deal with interconnection of disparate network type - like the IP.

Once everything is connected through switches and routers, there could be multiple ways to get from one point to another. Such paths should be dynamic and loop free. 

# Switching Basics

A switch is a mechanism that allows us to interconnect links to form a larger network.  It is a multi-input, multi-output devices that transfers packets from an input to one or more outputs. This allows a star topology to the set of possible network structures. This structure allows for several attractive properties: 
- large networks can be bult by interconnecting a number of switches
- we can connected switches to each other and to hosts using point-to-point links
- adding a new host does not reduce performance for other hosts already connected
![[Pasted image 20250203111503.png]]
A switch's primary job is to forward packets, and resides in layer 2 of the OSI. How does the switch know where to forward the packet? It looks at the header of the packet, and uses one of two common approaches. The first is the *datagram* or the *connectionless* approach. The second is the *virtual circuit* or *connection-oriented* approach. 

One thing common to all networks is that we need a way to identify end nodes. Hence, all Ethernet cards are assigned a globally unique identifier. 

We also need a way to identify input and output ports of each switch. For now, we use a numbering approach. 

## Datagrams

The idea behind datagrams is simple: in every packet, you include enough information to enable any switch to decide how to get to its destination. A switch will consult a *forwarding table* - also known as a *routing table* - to decide where to send the packet. This table is easy to generate when we have a static map, but typically the map is changing, leading to a harder problem known as *routing*. 
![[Pasted image 20250203112328.png]]
![[Pasted image 20250203112346.png]]

Datagram networks have the following characteristics: 
- A host can send a packet anywhere at any time, since any packet that turns up can be immediately forwarded. For this reason, datagram networks are often called *connectionless*, contrasting with a *connection-oriented* network where a *connection state* must be established before the first data packet is sent
- When a host sends a packet, it has now way of knowing if the network is capable of delivering it or if the destination host is even up and running
- Each packet is forwarded independently of previous packets that might have been sent to the same destination
- A switch or link failure might not have any serious effect on communication if it is possible to find an alternate route around the failure and update the routing table accordingly.
## Virtual Circuit Switching 

A second technique for packet switching uses the concept of a *virtual circuit*. This approach is also known as a *connection-oriented model*. It requires setting up a virtual connection from the source host to the destination host before any data is sent.  Hence, this approach requires two step if a host A wants to send packets to host B - the connection step and the data transfer step.
![[Pasted image 20250203114125.png]]
In the connection setup phase, it is necessary to establish a "connection state" in each of the switches between the source and destination hosts. This means that there will be an entry in a "VC table" for each switch that the connection passes. One entry in the VS tables on a single switch contains: 
- a *virtual circuit identifier (VCI)* that identifies the connection at this switch and will be inside the header of the packets that belong to this connection
- an incoming interface on which packets for this VC arrive at the switch
- an outgoing interface in which packets for this VC leave the switch
- a potentially different VCI that will be used for outgoing packets

An example of this in action: a packet arrives and contains the designated VCI value on its header - the packet should be sent out the specified outgoing interface with the specified outgoing VCI value without having been first placed in its header.

It should be noted that the VCI is not a globally significant identifier, but only has significance on a given link. 

When a new connection is created, we need to assign a new VCI for that connection on each link the connection will traverse, and that the chosen VCI is not currently in use. 

There are two broad approaches to establishing a connection state. One is for a network administrator to configure the state, in which the virtual circuit is permanent. Alternatively, a host can send messages to the network to cause the state to  be established - known as *signaling*. 

Lets assume a network administrator wants to manually create a new virtual connection from host A to host B. The administrator needs to identify a path through the network from A to B. The admin picks a VCI value currently unused for each link in the connection. 
![[Pasted image 20250203115126.png]]

Once the VC tables are set up, data transfer can proceed. 

Of course, setting up the VC tables is tedious and prone to mistakes. Thus, either a network management tool or some sort of signalling is almost always used even when setting up "permanent" VCs. We now describe how the same VC just described can be set up by signaling. 

To start, host A sends a setup message to switch 1. The message contains the complete destination address of host B. The message flows on to switches 2 and 3 before arriving at B.

When switch 1 receives the connection request, it crates a new entry in its VC table for this new connection. All following switches do the same. To complete the connection. everyone needs to know what their downstream neighbor is using as the VCI. Host B acknowledges the connection setup to switch 3 and tells switch 3 the VCI that they chose, and then so on and so forth. 

When host A no longer wants to send data to host B, it tears down the connection by sending a teardown message to switch 1. and the appropriate table entries would be deleted. 

There are several things to note about virtual circuit switching: 
- since host A has to wait for the connection request to go to host B and then back to itself, there is at least a one round-trip time (RTT)  of delay before data is sent
- the per-packet overhead caused by the header is reduced relative to the datagram model
- if a switch or link fails, the connection is broken and a new one will need to be established
- routing has been largely ignored in this scenario 

## Asynchronous Transfer Mode

Asynchronous Transfer Mode (ATM) became an important technology in the 1980s and early 1990s, and was seen as a competitor to Ethernet switching and IP. 

![[Pasted image 20250204110708.png]]
We skip the discussing the GFC because it never saw much use. The VPI and VCI combine to become a virtual circuit identifier, split to allow for a level of hierarchy. WE skip to the CRC header byte known as the *header error check* (HEC). This provides error detection capability on the cell header only. 


The most significant thing to notice about the ATM cell is rereason it is called a cell and not a packet. This is because it only has o ne size - 53 bytes. One reason for this is to facilitate the implementation of hardware switches. Ficed-length packets turn out to be very helpful if you want to build fast, highly scalable switches. There are too main reasons for this: 
1. It's easier to build hardware with simple jobs, and processing packets is easier if you know how long each one is
2. If all packets have the same length, you can have lots of switching element all doing the same thing in parallel. 

Another good argument for ATM has to do with end-to-end latency. ATM was meant for voice calls and data. Because voiec is low bandwith but has strict delay requirements, you dont want it to be stuck behind a large data packet. Hence, if you force all packets to be cells, then large data can still be served and voice packets can be interweaved in. This idea is still alive and well today in cellular access networks. 

How long should ficed length packets be? Too short and the amount of header information rlative the data gets larfer, so the percentage of bandwith used  goes down. More seriously, if you build a device that processes cells at some maximum number of cells per second, then as cells get shorter the total data rate drops. 

On the other hand, if cells are too big then you might waste bandwidth filling out the space needed for a complete cell. If you want to send 1 byte of data and the required payload is 48 bytes, you need 47 bytes of padding.

## Source Routing

 A third approach that uses neither virtual circuits nor conventional datagrams is known as *source routing*. The name derives from the fact that all information about network topology that is required to switch a packet across a network is provided by the source host. 

  There are multiple ways to do this. One way is to have an ordered list of switch ports and rotate the list to the next switch in the path is always at the front. 
![[Pasted image 20250204113153.png]]

This approach assumes that the host A knows enough bout the topology to form a header that has all the right directions, and is somewhat analgous to build the forwarding tables in a datagram network. 

We also cannot predict how big the header need to be, since it must hold one word for every swtich on the path, implying that headers have no upper bound unless we can predict with absolute certainty the maximum number of switched a packet will ever need to pass. 

You can also vary implementation on the rotation. For example, you may just strip numbers or just carry a pointer. 
![[Pasted image 20250204113516.png]]

Source routes can sometimes be categorized as *strict* or *loose*. Strict source routes demand that every node along the path be specified. Loose routes can be thought of as a set of waypoints rather than a completely specified route. 

# Switched Ethernet

We now focus more closely on a specific switching technology: *Switched Internet*. Suppose you have a pair of Ethernets you want to connect. One approach you might try is putting a repeater between any pair of hosts. This would not be a workable solution if doing so exceeded the physical limitations of the Ethernet. Alternatively, you can put a node in between two Ethernets, and have the node forwards frames from one Ethernet to the other. This node would fully implement Ethernet's collision detection and media access protocols.

This node is essentially a bridge. This strategy does have some pretty serious limitations, but a number of refinements have been added over the years to make bridges an effective mechanism for interconnecting a set of LANs. 
