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


> [!NOTE] Summary
> These switching basics mostly comprise of how do we send packets over a network. One option is datagrams - which include having a relatively static routing table to handle packet management. Another option is the virtual circuit method. This method involves two steps: connection and data-sending. This method sets up connections with surrounding nodes and uses an algorithm to undestand where to send packets. 

# Switched Ethernet

We now focus more closely on a specific switching technology: *Switched Internet*. Suppose you have a pair of Ethernets you want to connect. One approach you might try is putting a repeater between any pair of hosts. This would not be a workable solution if doing so exceeded the physical limitations of the Ethernet. Alternatively, you can put a node in between two Ethernets, and have the node forwards frames from one Ethernet to the other. This node would fully implement Ethernet's collision detection and media access protocols.

This node is essentially a bridge. This strategy does have some pretty serious limitations, but a number of refinements have been added over the years to make bridges an effective mechanism for interconnecting a set of LANs. 

## Learning Bridges

The first optimization we can make is to observe that it need not forward all frames that it receives. 
![[Pasted image 20250206114722.png]]
Consider the bridge above. If host A sends a frame to host B, the bridge does not need to send the frame over to port 2. The question is: how did the bridge know which port the various hosts reside?

One option is for the bridge to have a table like the one above. Hence, it would know exactly which host in related to which port. Because having a human maintain this table would be too cumbersome, the bridge will inspect the source address in all frames it receives, and records the fact that a frame from host A was just received on port 1. 

Note that each packet caries a global address, and the bridge decides which output to send a packet. 

When a bridge first botts, the table is empty, and then adds entries over time. Also, a timeout is associated with each entry such that after a period of time the bridge discards the entry. This is in guard of the situation when one host is moved from one network to another. If a bridge receives a frame addressed to a host not currently in the table, it forwards the frame on all other ports. 

## Spanning Tree Algorithm

The strategy we just talked about works fine until the network has a loop in it, in which case it fails horribly. 

![[Pasted image 20250206120600.png]]

Suppose a packet enters S4 from Host C, and that the destination address is one not yet in any switches' forwarding table. S4 sends a copy of its packet to S1 and S6.  S6 forwards to S1, and S1 forwards to S6, both of which are forwards to S4. S4 still does not have the packet's destination address in its table, so it keeps forward to S1 and S6. Hence, a cycle forms. 

Why would there be a loop? One possibility is that the network is managed by more than one administrator, and that no single person know the entire configuration of the network. Another reason is that loops may be built in to provide redundancy just in case something fails in the middle of the network.

This problem is addressed by running a distributed *spanning tree* algorithm. 
![[Pasted image 20250206121148.png]]
The spanning tree algorithm is a protocol used by a set of switches to agree upon a spanning tree for a particular network. This means that each switch will limit which ports it will send data through, and is possible that an entire switch will not participate in sending frames. This algorithm is also dynamic, to account for possible switch failures. 

The main idea behind the spanning is for switches to select the ports over which they will forward frames. The root node will be the switch with the smallest ID, and will always forward frames over all ports. Next, each switch computes the shortest path to the root and notes which ports are on this path. Then, to account for the fact that there could be a switch connected to its ports, the switch elects a single *designated* switch to forward frames towards the root. The designated switch is the one closest to the root, and if there is a tie, the one with the smallest ID wins. Of course, each switch may be connected to more than one other switch, so it participates in the election of a designated switch for each such port. This means that each switch decides if it is the designate switch relative to each of its ports. The switch forwards frames over those ports for which it is the designated switch. 
![[Pasted image 20250206122903.png]]While it is possible for a human to look at the given network a compute the spanning tree, switches do not have the luxury of seeing the topology of the entire network. Instead, they have to send configuration messages. 

The configuration messages contain three pieces of information: 
1. The ID of the switch sending the message.
2. The ID of the sending switch believed to be the root switch. 
3. The distance - measured in hops - from the sending switch to the root switch

Each switch records the current *best* configuration message it has seen on each of its ports, including messages from itself and by other switches. 

Initially, each switch thinks its the root and sends a config message saying that it is the root. Upon receiving the message, a switch checks to see if the new message is better than the current config. The new message is considered better if any of the following is true:
1. It identifies a root with a smaller ID
2. It identifies a root with an equal ID but with a shorter distance.
3. The root ID and distance are equal, but the sending switch has a smaller ID.

If the new message is better, the switch discards the old information. However, it first adds 1 to the distance-to-root field since the switch is one hop farther away from the root than the switch that sent the message. 

Once a switch receives a message indicating that it does not have the best message, it stops generating configuration messages and only forwards messages from other switches. Thus, the system stabilizes only when the root switch is still generating configuration messages and the other switches are forwarding messages only over ports for which they are the designated switch. At this point, a spanning tree has been built. 

## Broadcast and Multicast

Since he goal of a switch is to extend a LAN across multiple networks and most LANs support broadcast and multicast, switches must also support these two features. Broadcast is simple - each switch forwards a frame with a destination broadcast address out on each active (selected) port other than the one on which the frame was received.

Multicast can be implemented in exactly the same way, with each host deciding for itself to accept the message. However, we can also use the spanning tree algorithm to prune networks over which multicast frames need not be forwarded. This method is not widely adopted. 

## Virtual LANs (VLANs)

Switches do not scale. The spanning tree algorithm is linear, and switched forward all broadcast frames even when other people most likely do not need to see everyone's messages. 

The VLAN is a way to increase scalability. VLANs allow a single extended LAN to be partitioned into seemingly separate LANs, and each VLAN is assigned an identifier where packets can only travel from one segment to another if both have the same identifier. 
![[Pasted image 20250210110612.png]]



We now provide an example. Suppose I broadcast from Host X. The packets goes to S2. S2  observes that it came from a port configured as VLAN 100 and inserts the VLAN header  between the Ethernet header and its payload. The switch now forwards the packet, but sets an extra restriction that the packet may not be sent out an interface not part of VLAN 100. The packets is then forwarded onto S1, and then to host W. 

VLANs are attractive because you can change the logical topology w/o moving any wires or changing any addresses. 

![[Pasted image 20250210111707.png]]

We note that switches are limited in the kinds of networks they can interconnect. In particular, switches support only networks that have the exact same format for addresses - in other words, they do not generalize well. 

# Internet (IP)

In this section, we explore some ways to go beyond the limitations of bridges networks, and build large, highly heterogeneous networks with reasonably efficient routing.  We refer to such networks as *internetworks* 

## What is an Internetwork?

We use the term *internetwork* - or sometimes just *internet* - to refer to an arbitrary collection of networks interconnected to provide some host-to-host packet delivery service. When we talk about the widely used global internet in which a large percentage of networks are now connected, we call it the *Internet* with a capital *I*. We mainly want to learn about the internet, buyt will use real-world examples from the Internet. 

We also specify that a *network* is a directly connected or a switched network, using one technology like 802.11 or Ethernet. An *internetwork* is an interconnected collection of such networks. An internet is a *logical* network built out of a collection of physical networks. In this context, a collection of Ethernet segments connected by bridges or switches would still be viewed as a single network. 
![[Pasted image 20250210112952.png]]The *Internet Protocol* is the key tool used today to build scalable, heterogeneous networks. One way to think about IP is that it runs on all the nodes (both hosts and routers) in a collection of networks and defines the infrastructure that allows these nodes and networks to function as a single logical internetwork. 
![[Pasted image 20250210114409.png]]

## Service Model 

A good place to start when you build an internetwork is to define its *service model* - the host-to-host services you want to provide. The main concern is that we can provide a host-to-host service only if this service can somehow be provided over each of the underlying physical networks. For example, if our service were to guarantee delivery every packet in 1 ms or less, we should not have underlying network technologies that could arbitrarily delay packets.

The IP service model can be thought of ass having two parts: an addressing scheme - to identify all hosts - and a datagram (connectionless) model of data delivery. 

### Datagram Delivery 

Recall that a datagram is a type of packet that happens to be sent in a connectionless manner over a network. In essence, you send a packet, and the network makes a "best-effort" to send the packet to its destination, and does not attempt to recover from failure. This is sometimes called an *unreliable* service. 

Best effort, connectionless service is about the simplest service you could ask for from an internetwork. 

Because IP can "run over anything", it has made it extremely adaptable not only to new technology, but legacy protocols as well. 

### Packet Format

A key part of the IP service model is the type of packets that can be carried. Notice that this is a different way from the way we've been using to represent packets. This is because packet formats at the internetworking layer and above, and are are almost invariably designed to align on 32-bit boundaries to simplify the task of processing them in software. 
![[Pasted image 20250213114744.png]]

### Fragmentation and Reassembly

One of the problems of providing a uniform host-to-host service model over a heterogeneous collection of networks is that each network technology tends to have its own idea of how large a packet can be. Ethernet accepts packets up to 1500 bytes, but modern-day variants can deliver up to 9000 bytes. Hence, IP datagrams provide a means by which packets can be fragmented and reassembled when they are too big to go over a given network technology. This is a good choice given that new technologies are always showing up and the host does not need to send needlessly small packets. 

The central idea here is that every network type has a *maximum transmission unit* (MTU) - the largest IP datagram that it can carry in a frame. This value is smaller than the largest packet size on that network because the IP datagram needs to fit in the *payload* of the link-layer frame. 

When a host sends an IP datagram, it can choose any size it wants - a reasonable choice is the MTU of the network to which the host is directly attached. Fragmentation will be necessary if the path to the destination includes a network with a smaller MTU. 

To enable fragments to be reassembled, they all carry the same identifier in the *Ident* field. This identifier is chosen by the sending host. The receiver can then reassemble those fragments and, should not all fragments arrive, the receiver will give up reassembly and discard the fragments that did arrive. 
![[Pasted image 20250213121041.png]]

## Global Addresses

We need a global addressing scheme for IP.

Although Ethernet addresses are globally unique, Ethernet addresses are also *flat* - meaning they have no structure and provide few clues to routing protocols. In contrast, IP addresses are *hierarchical* , meaning they make up several parts that correspond to some sort of hierarchy in the internetwork. Specifically, IP addresses consist of two parts - the *network* and the *host*. The network part of an IP address identifies the network to which the host is attached; all hosts attached to the same network have the same network part in their IP address. The host part then identifies each host uniquely on that particular network. 

![[Pasted image 20250217112747.png]]

Looking at the routers above, we can see they are attached to two networks. They need to have an address on each network. For example, router R1 - which sits between the wireless network and an Ethernet - has an IP address on the interface to the Ethernet that has the same network part as the hosts on that Ethernet. Thus, it is more precise to think of IP addresses as belonging to interfaces than to hosts. 

What do these hierarchical addresses look like? The size of these two parts are not the same for all addresses. Originally, IP addresses were divided into three different classes - each of which defines different-sized network and host parts. 

![[Pasted image 20250217113646.png]]

The class of an IP address is identified in the most significant few bits. If the first bit is 0, it is class A. If the first bit is 1 and the second is 0, it is class B. If the first two bits are 1 and the third is 0, it is class C. Each class allocates a certain number of bits for the network part of the address and the rest for the host part (see above for the ratios).

This addressing scheme has a lot of flexibility, allowing networks of vastly different size to be accommodated fairly efficiently. The original idea was that the Internet would consist of a small number of WANs, a modest number of site-sized networks, and a large number of LANs. However, this scheme needed to be more flexible, and so IP addresses today are normally "classless."

By convention, IP addresses are written as four *decimal* integers separated by dots. Each integer represents the decimal value contained in 1 byte of the address, starting at the most significant. 

### Datagram Forwarding in IP

Recall that *forwarding*is the process of taking a packet from an input and sending it out on the appropriate output, while *routing* is the process of building up tables that allow the correct output for a packet to be determined. We focus here on forwarding.

The main points to bear in mind as we discuss forwarding of IP datagrams are the following:
- Every IP datagram contains the IP address of the destination host
- The network part of an IP address uniquely identifies a single physical network that is part of the larger Internet
- All hosts and routers that share the same network part of their address are connected to the same physical network
- Every physical network that is part of the Internet has at least one router that, by definition, is also connected to at least one other physical network

Forwarding IP datagrams works as follows. A datagram is sent from a source host to a destination hosts, possible passing through several routers. Any node tries to figure out if its connected to the same physical network  as the destination. If a match occurs, then the packet can be directly delivered over that network. Else, it needs to send the datagram to a router. In general, each node will have a choice of several routers, so it needs to choose the best router. This best router is called the *next hop* router. The router finds the correct next hop by consulting the forwarding table. 

In terms of pseudocode, this is the forwarding algorithm: 
![[Pasted image 20250217120032.png]]


### Subnetting and Classless Addressing

The original intent of IP addresses was that the network part would uniquely identify exactly one physical network. This approach has a couple of drawbacks. Imagine a large campus with lots of internal networks and decides to connect to the Ethernet. For every network, no matter how small, the site needs at least a class C network address. Even worse, for wany network with more than 255 hosts, they need a class B address. This causes a lot of waste in terms of network numbers. 

Using one network number per physical network, hence, uses up the IP address space much faster than we'd like. We also do not want too much network numbers because the router then has to add more values to its routing table, making routers more expensive and less efficient. 

*Subnetting* provides a single step to reducing total number of network numbers that are assigned. The ideas is to take a single IP network number and allocate the IP addresses within to several physical networks, now referred to as *subnets*. The subnets should be close to one another, because from the POV of the Internet, they will all look like a single network, with only one network number between them. This means that a router will only be able to select one router to reach any of the subnets. A perfect situation for this is a large campus or corporation. From outside the campus, all you need to know is the single point at which the campus is connected to rest of the Internet, and then then campus's internal subnets can handle the rest. 

The mechanism in which a single network number can be shared among mulitple networks involves configuring all nodes on each subnet with a *subnet mask*. All hosts on the same network must have the same network number. This subnet mask enables us to introduce a *subnet number* - all hosts on the same physical netwrok will have the same network number, meaning that hosts may be on different physical networks but share a single network number. 

![[Pasted image 20250217122539.png]]

What subnetting means to a host is that is now has both an IP address and a subnet mask for the subnet to which it is attached. 

![[Pasted image 20250217122641.png]]
For example, H1 is configured with an address of 128.96.34.15 and a subnet mask of 255.255.255.128. The bitwise AND of these two numbers defines the subnet number of the host and of all other hosts on the same subnet. In this case, the result becomes 128.96.34.0, so this is the subnet number for the topmost subnet. 

When the host wants to send a packet to a certain IP address, the first thing it does is perform a bitwise AND between its own subnet mask and the destination IP address. If the result equals the subnet number of the sending host, then we know the destination is on the same subnet and the packet can be delivered over the subnet. If not, the packet needs to be sent to another subnet, therefore sending the packet to a router. 

The forwarding of a router also changes slightly while introducing subnetting. Recall, previously the forwarding tables had entries of the form (NetworkNum, NextHop). Now, the table must hold (SubnetNumber, SubnetMask, NextHop). To find the right entry in the table, the router ANDs the packet's destination address with the SubnetMask for each entry in turn. 
![[Pasted image 20250220110910 1.png]]
An important consequence of subnetting is that different parts of the internet see the world differently. From outside our hypothetical campus, routers see a single network. However, internally, packets need to sent to the right subnet. Thus, not all parts of the internet see exactly the same routing information. This is an example of *aggregation* of routing information. 

## Classless Addressing

Subnetting has a counterpart, sometimes called *supernetting* but more often *Classless Interdomain Routing* (CIDR). CIDR takes the subnetting idea to its logical conclusion by doing away with address classes together. Why isn't subnetting only sufficient? In essence, subnetting allows us to split a classful address among multiple subnets, while CIDR allows us to coalesce several classful addresses into a single "supernet." The further tackles address space inefficiency and in a way that keeps the routing system from being overloaded. 

For example, consider a company whose network has 256 hosts on it. That's slightly too much for a Class C address, so you would be tempted to assign a class B. However, this is clearly inefficient, since you would only be using a tiny fraction of the hosts available for a class B. Even if subnetting helps us assign addresses carefully, any organization with more than 255 hosts will want a class B address. 

One way to deal with this is to refuse to give a class B address inless they have a need for something close to 64K addresses, and just given them a number of class C addresses. 

This solution raises a problem though: excessive storage requirements at the routers. If a single site has 16 class C network numbers, then each Internet backbone router needs 16 entries in its routing tables to direct packets to that site. If we assigned a class B, then we would only need one entry. 

CIDR tries to balance the number of routes that a router needs to know against the need to hand out addresses efficiently. To do this, CIDR helps us to *aggregate* routes - meaning it lets us use a single entry in a forwarding table to tell us how to reach a lot of different networks. 

To do this, it breaks the rigid boundaries between address classes. Lets look at our hypothetical organization with 16 class C network numbers. Instead of handing out some random 16 class C addresses, we can hand out a block of contiguous class C addresses. For example, we can give out class C network number from 192.4.16 through 192.4.31. See that the top 20 bits of the addresses are the same. Hence, what we have effectively created is a 20-bit network number - somewhere between a class B and a class C. Observe that, for this scheme to work, we need to hand out blocks of class C addresses that share a common prefix, which means that each block must contain a number of class C networks that is a power of 2. 

CIDR requires a new type of notation to represent network number - *prefixes* as they are known, since prefixes can be of any length. The convention is to place a /X after the prefix, where X is the prefix length in bits. So, for our example, the 20 bit prefix would be represented as 192.4.16/20. Nowadays, it is more common to hear people talk about "slash 24" prefixes than class C networks. Note that representing a network address in this way is similar to the (mask, value) approach used in subnetting. 

The ability to aggregate is only the first step. If we are an ISP, if we assign prefixes to customers in a way that many different networks share a common, shorter address prefix, we can get even aggregation of routes. 

![[Pasted image 20250220142707.png]]

One way to accomplish this is to assign a portion of the address space to the provider in advance, and then let the network provider assign addresses from that space to its customers as needed. 

### IP Forwarding Revisited

Because CIDR has prefices of any length, these prefixes in a forwarding table may "overlap", in the sense that some addresses may math more than one prefix. For example, if we find both 171.69 and 171.69.10 in the routing table, then a packet like 171.68.10.5 matches both. The rule in this case is the principle of "longest match" - the packet matches the longest prefix. 

## Address Translation (ARP)

We glosses over how to get a datagram to a particular host or router on that network. The main issue is that IP datagrams has IP addresses, while the physical hardware only understands the addressing scheme of that particular network. Thus, we need to translate the IP address to a link-level address that makes sense. We then encapsulate the IP datagram inside a frame that contains the link-level address and send it either to the ultimate destination or to a router that promises to forward the datagram toward the ultimate destination. 

A simple way to map an IP address into a physical network address is to encode a host's physical address in the host part of its IP address. For example, a host physical address 33.81 might be given the IP address 128.96.33.81. However, we are limited that the network's physical address size limit. 

A more general solution would be have each host maintain a table of address pairs where IP addresses would be mapped to physical addresses. We use the Address Resolution Protocol (ARP) for each host to dynamically learn the contents of the table using the network. Since these mapping may change over time dure to changes in the network, entries are timed out periodically and removed every 15 minutes. This set of mapping is known as the ARP cache or the ARP table. 

ARP takes advantage of the fact that many link-level network tech, such as Ethernet, support Broadcast. If a host wants to send an IP datagram to a host it knows to be on the same network, it will first check for a mapping in the cache. If not mapping is found, it invokes ARP by broadcasting an ARP query onto the network. The query contains the target IP address, and each host received the query and checks to se if it matches its IP address. If it matches, the host sends a response message containing the link-layer address back to the originator of the query. The originator adds that information to its ARP table. 

The query message also included the IP address and link-layer address of the sending host. THus, when a host broadcasts a query message, each host on the netwrok can learn the sender's link-level and IP address and add that to the table unless it already has that information. In that case, the existing entry gets refreshed, and resets the time to discard the entry. If a host is the target for that query, it adds the information about its sender to its table, even if it did not already have an entry for the hose because there's a good chance the source host is about to send an application level message and may eventually have to send an ACK back to the source - it will need the source's physical address to do this. If a host is not the target, then it does not need to add this link-level address. 

![[Pasted image 20250220153704.png]]

The ARP packet contains:

- a **HardwareType** field, which specifies the type of physical network (e.g. Ethernet)
- a **ProtocolType** field, which specifies the higher-layer protocol (e.g., IP)
- **HLen** (hardware) and **PLen** (protocol), which specifiy the length of the link-layer address and higher-layer protocol address
- An **Operation** field, which specifies whether this is a request or a response. 
- The source and target hardware (Ethernet) and protocol (IP) addresses.

We have now seen how IP deals with both heterogeneity and scale. IP makes a best-effort service model that makes minimal assumptions about the underlying network. IP then has a common packet format and a global address space for identifying all hosts. On scale, IP uses hierarchical aggregation to reduce the amount of information needed to forward packets. 

## Host Configuration (DHCP)

Ethernet addresses are configured into the network adaptor by the manufacturer, and is managed to make sure all addresses are globally unique. However, IP addresses have a network and a host, so a manufacturer cannot fix the IP address because they do not know which host a machine will connect to. Hence, IP addresses need to be reconfigurable.

In addition, a host must have the address of a default router to send packets whose destination is not on their same network. 

Because manually configuring IP address can be error prone, a protocol called the *Dynamic Host Configuration Protocol* (DHCP) is used to automate this process. 

DHCP relies on a DHCP server responsible for providing configuration information to hosts. At the simplest level, the DHCP server can function just as a centralized repository for host configuration information. This way, a network administrator, if they do need to edit addresses, doesn't have to go to the host directly but can just update the server. More sophisticatedly, the DHCP server maintains a pool of available addresses that it hands out to hosts on demand. 

Because the point of DHCP is to minimize manual configuration, and we do not want to go to each host to configure the address of the DHCP server, we come across the problem of discovery. 

To contact a DHCP server, a newly booted or attached host sends a DHCPDISCOVER message to the IP broadcast address (255.255.255.255). This means it will be received by all hosts and routers. In the simplest case, the DHCP server then responds. However, it is not desirable to have one DHCP server on every network, because this requires quite a large number of networks. Thus, DHCP uses a *relay* *agent* for each  network, and just contains the address of the DHCP server. 

![[Pasted image 20250224114214.png]]

When DHCP dynamically assigns IP addresses to its hosts, it is clear that hosts cannot keep addresses indefinitely. Hence, the DHCP allows addresses to be leased for some period of time. Once the lease expires, the server is free to return that address to its pool. A host with a leased address needs to periodically renew the lease. 
![[Pasted image 20250224114507.png]]

# Error Reporting (ICMP)

IP is always configured with a companion protocol known as the *Internet Control Message Protocol* (ICMP) that defines a collection of error messages sent back to the source host when a router or host fails. 

ICMP also has a handful of control messages a router can send back to a source host. ONe of the most useful - called *ICMP-Redirect* tells the source host that there is a better route to the destination. 

## Virtual Networks and Tunnels

We have focused on being able to unrestrictedly be able to send information to each other. However, there are times where we want to control connectivity. Here, the *virtual private network* (VPN) comes into play. 

We can first think about a private networks such that communication is restricted to certain sites. To make private network virtual, transmission lines would have to be replaced by some sort of shared network. A virtual circuit is a very reasonable replacement for a leased line because it still provides logical point-to-point connection between corporation sites. 
![[Pasted image 20250224115245.png]]
In the figure above, a virtual circuit network is used to provide controlled connectivity among sites. We can use a n IP network to provide this connectivity, but we can't just connect the corporations' sites to a single internetwork because we want X and Y to stay separate. Hence, we have an *IP tunnel*.

We can think of an IP tunnel as a virtual point-to-point link between a pair of nodes that are actually separated by an arbitrary number of networks. The router at the entrance of the tunnel will be provided with the IP address of the router at the far end. Whenever the router at the entrance of the tunnel wants to send a packet over this virtual link, it encapsulates the packet inside an IP datagram, and places the destination address as the address of the router at the far end of the tunnel. 

![[Pasted image 20250224115749.png]]
Why would anyone go through this trouble? One reason is security. A tunnel can be a private sort of link across a public network. Another may be that by connecting routers with a tunnel, we can build a virtual network in which all routers with a certain capability appear to be directly connected. Another is to carry packets from protocols other than IP across an IP network. We also see that tunneling forces a packet to be delivered to a particular place even if its original header suggest it go elsewhere. 

Tunneling has downsides. It increases the length of packets - wasting bandwidth and increasing fragmentation. Routers at the end of the tunnel also have to deal with more work because they have to add and remove the tunnel header. Finally, there is a management cost for whoever is administrating these tunnels. 

# Routing

We have assumed that switches and router know enough to choose which port to forward packets. As we've seen, this information is grabbed from the routing table. We now talk about how these switches and routers acquire information into their forwarding tables. 


> [!NOTE] Distinction between forwarding and routing
> Forwarding is the process of receiving a packet, looking up its destination address in a table, and sending the packet in a direction determined by that table. 
> Routing is the process by which forwarding tables are built. 

We will also make a distinction between the *forwarding table* and the *routing table*. The forwarding table is used when a packet is being forwarded and must contain enough information to accomplish the forwarding function. The routing table is built by routing algorithms as a precursor to building the forwarding table, general containing mappings from network prefixes to next hops. 

![[Pasted image 20250225113803.png]]

![[Pasted image 20250225113814.png]]

The key question we need to ask anytime we try to build a mechanism for the Internet is: "Does this scale?" The solutions that will be described do not scale much, but provide a good building block for the hierarchical routing infrastructure used in the Internet today. Hence, the problem of routing that we will be discussing is in the context of a small to midsized network. 

## Network as a Graph

Routing is a problem of graph theory. 
![[Pasted image 20250225114136.png]]


In our initial discussion, nodes are routers and edges are network links that have some associated cost. The basic problem of routing is to find the lowest-cost path between any two nodes. For a simple network, you might just want to calculate all shortest paths and then load them into some storage on each node. This has several shortcomings: 
- does not deal with node or link failures
- does not consider the addition of new nodes or links
- implies that edge costs cannot change

Hence, we instead run routing protocols among the nodes to have a distributed, dynamic way of finding the lowest cost path in the presence of link and node failures and changing edge costs. 

To begin, we assume that edge costs are known. We examine the two main classes of routing protocols: *distance vector* and *link state*. 

### Distance-Vector (RIP)

The idea behind this algorithm is suggested by its name (also referred to as the Bellman-Ford after its inventors).

The starting assumption is that each node knows the cost of the link to each of its directly connected neighbors. A link that is down is assigned an infinite cost.

![[Pasted image 20250227114710.png]]
![[Pasted image 20250227114740.png]]
  We consider the above a demonstration of this algorithm. Because each link is set to 1, the least-cost path is just the one with the fewest hops. Each node only knows one row of the table shown above. 

We consider each row the node's current belief about its distances from all other nodes. 

The next step is for every node to send a message to its directly connected neighbors containing its personal list of distances. This helps each node learn how far away it is from the neighbors of its neighbors. 

In the absence of any topology changes, it will take only a few exchanges of information before each node has a complete routing table. The process of getting consistent routing information to all nodes is called *convergence*. 

There are a few details to note. Firstly, there are two different circumstances in which nodes will send a routing update. One of these circumstances is the *periodic* update - each node sends an update message every so often. The second is a *triggered* update, where if a node notices a link failure or an update from its neighbor it will send an update to its neighbors. 

If a link goes down, a problem may occur when the network is trying to stabilize - specifically a *count to infinity* problem, which happens due to timing issues. Essentially, a node may initially know that a link is down, but then gets updates from its neighbors that will change its routing table to show that the link is down. Eventually, the nodes will know that the distance is infinite, but only when the distance is so big that it must be infinite.

We can use a technique called *split horizon* to solve this problem. However, this technique only works for routing loops that involve two nodes. 

### Routing Information Protocol (RIP)

One of the more widely used routing protocols in IP networks is the Routing Information Protocol (RIP). 

Routing protocols in internetworks differe very slightly from the idealized graph model described above. In an internetwork, the goal of the routers is how to forward packets to various *networks*. Hence, these link distances are the cost of reaching different networks. 

![[Pasted image 20250227120714.png]]
![[Pasted image 20250227120741.png]]
RIP has a fairly straightforward implementation of distance-vector routing. Routers running RIP send advertisements every 30 seconds. RIP also can have multiple addresses, not just IP - which is why it has *Family* has part of their advertisements. RIP, however, is only used in fairly small networks - those with no paths longer than 15 hops. 

## Link State

Link state routing is the second major class of intradomain routing protocol. 

The starting assumptions are rather similar to that of distance-vector routing. Each node is assumed to be capable of finding out the state of the link to its neighbors and the cost of each link. The basic idea behind link-state protocols is simple: each node knows its directly connected neighbors and, if we want the totality of this knowledge disseminated to every node, then every node has enough knowledge of the network to build a complete map of the network. Thus, link-state routing protocols rely on two mechanisms: reliable dissemination of link-state information, and the calculation of routes from the sum of all the accumulated link-state knowledge. 

### Reliable Flooding

*Reliable flooding* is the process of making sure all nodes participating in the routing protocol get a copy of the link-state information from all the other nodes. The basic idea is for a node to send its link-state information out on all its directly connected links, which then gets sent out on more links, so on and so forth until the information has reached every node. 

More precisely, a node sends out an update packet - aka a *link-state packet* (LSP) that contains:
- the ID of the node that created the LSP
- a list of directly connected neighbors of that node, with the cost of the link to each one
- a sequence number
- a time to live for this packet

The first two qualities are for route calculation, and the last two is for reliability. 

Flooding works in the following way. Firstly, the transmission of LSPs between adjacent routers is reliable using ACKs and retransmissions just link in the link-layer protocol. To ensure reliability, however, we need a couple more steps.

![[Pasted image 20250227122609.png]]

Consider the node X that has received a copy of an LSP originating at some other node Y. X checks to see if it has a stored copy of an LSP from Y. If not, it saves that LSP. If X already has a copy, it compares the sequence numbers, and keeps the largest one (assumed to be more recent). If the new LSP is more recent, then we discard our old LSP and save this new one. We then broadcast the LSP to our neighbors, except for the node that we received the LSP from. 

One the important design goals on a link-state protocol's flooding mechanism is that the newest information must be flooded to all nodes as quickly as possible, and old information removed from the network and not allowed to circulate. Another goal is just to remove the amount of routing traffic, to remove overhead of the application. 

One way to reduce overhead is to avoid generating LSPs unless absolutely necessary by reducing periodic updates. 

To make sure old information gets replaced by newer information, LSPs carry sequence numbers that do not wrap. If a node goes down and then comes back up, it will start with a sequence number of 0. If the node was down a long time, the old LSPs have all timed out.

### Route Calculation

The solution for the route calculation is based on Dijkstra's Shortest Path Algorithm. In practice, each switch computes its routing table directly from the LSPs it has collected using a realization of Dijkstra's algorithm called the *forward search* algorithm. Specifically, each switch has two lists - *Tentative* and *Confirmed*. Each of these lists contains a set of entries of the form *(Destination, Cost, NextHop)*. The algorithm works as follows: 
1. Initialize the *Confirmed* list with an entry for myself; this entry has a cost of 0. 
2. For the node just added to *Confirmed* in the previous step, call it node *Next* and select its LSP
3. For each neighbor(*Neighbor*) of *Next*, calculate the cost (*Cost*) to reach this *Neighbor* as the sum of the cost from myself to *Next* and *Next* to *Neighbor*. 
	1. If *Neighbor* is currently onneither the *Confirmed* nor the *Tentative* list, then add *(Neighbor, Cost, NextHop)* to the *Tentative* list, where *NextHop* is the direction I go to reach *Next*
	2. If *Neighbor* is currently on the *Tentative* list, and the *Cost* is less than the currently listed cost for *Neighbor*, then replace the current entry with *(Neighbor, Cost, NextHop)* where *NextHop* is the direction I go to reach *Next*.
4. If the *Tentative* list is empty, stop. Otherwise, pick the entry from the *Tentative* list with the lowest cost, move it to the *Confirmed* list, and return to step 2. 

![[Pasted image 20250306111525.png]]
![[Pasted image 20250306111553.png]]

This algorithm has many nice properties: It stabilizes quickly, does not generate much traffic, and responds rapidly to topology changes or node failures. On the downside, the information stored at each node can  be quite large. This causes a problem for scalability. 


> [!NOTE] Key Takeaway
> Distance-vector and link-state are both distributed routing algorithms. Distance-vector only talks to directly connected neighbors but tells them everythin they know; meanwhile, in link-state each node talks to all other nodes but only tells it what it knows for sure (ie. the state of its directly connected links). 

## The Open Shortest Path First Protocol (OSPF)

One of the most widely used link-state routing protocols is OSPF. The first word "Open" refers to the fact that it is an open, nonproprietary standard, created under IETF. The "SPF" part comes from an alternative name for link-state routing. OSPF adds quite a number of features to the basic link-state algorithm.

- *Authentication of routing messages* - because information is being passed from the impact of many other nodes, it is important to authenticate routing messages. Now, there is strong cryptographic authentication.
- *Additional hierarchy* - OSPF introduces another layer of hierarchy into routing by allowing a domain to be partitioned into *areas*. Hence, a router may not need to know how to reach every network, but only how to get to the right *area*. 
- *Load Balancing* - OSPF allows multiple routes to the same place to be assigned the same cost and will cause traffic to be distributed evenly over those routes, making better use of network capacity. 

![[Pasted image 20250306112933.png]]

There are several different types of OSPF messages, but they all begin with the same header. 

Of the 5 OSPF message types, type 1 is the "hello" message, where a router sends to its peers to notify that it is still alive and connected. The remaining types are used to request, send, and acknowledge the receipt of link-state messages.

Like any internetworking routing protocol, OSPF must provide information about how to reach networks. Thus, OSPF must provide a good chunk of information. A router running OSPF may generate link-state packets that advertise one or more of the networks that are directly connected to that router. In addition, a router connected to another router by some link must advertise the cost of reaching that router. These two types of advertisements are necessary to enable all the routers in a domain to determine the cost of reaching all networks in that domain and the appropriate next hop for each network.

![[Pasted image 20250306113625.png]]

The above is a Type I LSA that advertises the cost of links between routers. Type 2 LSAs are used to advertise networks to which the advertising router is connected, while the other types are used to support additional hierarchy as will be described. The *LS Age* is the equivalent to a time-to-live.

In a type 1 LSA, the *Link state ID* and the *Advertising router* field are identical. Each carries a 32-bit identifier for the router that created this LSA. It is essential that his ID is unique and that a router consistently uses the same router ID. 

The *LS sequence number* is used exactly as described above to detect old or duplicate LSAs. The *LS* checksum is similar to other we have seen and is used to verify that data has not been corrupted. 

Now to the actual link-state information. This is made a little complicated by the presence of TOS (type of service) information. Ignoring that for a moment, each link in the LSA is represented by a *Link ID*, some *Link Data*, and a *metric*. The first two of these fields identify the link; a common way to do this would be the router ID of the router at the far end of the link as the *Link ID*and then use the *Link Data*  to disambiguate among parallel links if  necessary. The *metric* is the cost of the link. *Type* tells us something about the link - like if it is a point-to-point link. 

The TOS information allows the OSPF to choose different routes for IP packed based on the value in their TOS field. Instead of assigning a single metric to a link, it is possible to assign different metrics based on the TOS (type of service) value. 

### Metrics 

The preceding discussion assumes that link costs, or metrics, are known when we execute the routing algorithms. We now look at some ways to calculate link costs that have proven effective in practice. On example that we have seen is to assign a cost of 1 to all links. In this case, the least-cost solution is the solution with the least amount of hops. Such as approach has some drawbacks. It does not distinguish between links on a latency basis, so a satellite link with 250-ms latency looks just as attractive to the routing protocol as a terrestrial link with 1-ms latency. It also does not distinguish between links based on a capacity basis, making a 1-Mbps link look just as good as a 10-Gbps link. Finally, it does not distinguish between link based on their current load, making it impossible to router around overloaded links. It turns out this last problem is the hardest. 

The ARPANET was the testing ground for a number of different approaches to link-cost calculation. The following discusses the evolution of the ARPANET routing metric and, in doing so, explores the subtle aspects of the problem. 

The original ARPANET routing metric measured the number of packets queued waiting to be transmitted on each link, meaning a link with 10 packets queued was assigned a larger cost than a link with 5 packets queued. However, this method just moves the packet towards the shortest queue rather than toward the destination. Stated more precisely, the original ARPANET routing mechanism did not take bandwidth nor latency of the link into consideration.

A second version takes both into consideration and used delay - rather than queue length - as a measure of load. This was done as follows:
1. each incoming packet is timestamped with its time of arrival (*ArrivalTime*). It's deparature is also recorded *(DepartTime)*
2. The link-level ACK is received from the other side. 
3. The following is calculated, where TransmissionTime and Latency are statically defined: $$(DepartTime - ArrivalTime) + TransmissionTime + Latency$$
Although an improvement, this still had a lot of problems. Under a light load, it worked reasonably well. Under a heavy load, a congested link would start to advetise a very high cost, causing all traffic to leave that link idle. It would then advertise a a low cost, and all the traffic would go back to that link. This instability actually led to a lot of links being idle. 

Another problem was the range of link values was much too large. For example, a heavily loaded 9.6 Kbps link would look 127 times more costly than a lightly loaded 56-kbs link. That means the routing algorithm would choose a path with 126 hops of lightly loaded 56-kbps links in preference to a 1-hop 9.6 kbps path. 

A third approach addressed these problems. The major changes were to compress the dynamic range of the metric considerably and to smooth the variation of the metric with time. 

The smoothing was achieved by several mechanisms. First, the delay measurement was transformed to link utilization, and that number was average with the last reported utilization so suppress sudden changes. Second, there was a hard limit on how much the metric could change from one measurement cycle to the next. 

For example, we could have the following rules:
- a highly loaded link never shows a cost of more than three times its cost when idle
- the most expensive link is only 7 times the cost of the least expensive
- a high-speed satellite link is more attractive than a low-speed terrestrial link
- cost is a function of link utilization only at moderate to high loads

However, in the majority of real-world network deployments, metrics rarely change at all because conventional wisdom holds that dynamically changing metrics is too unstable. More significantly, many networks today lack the great disparity of link speeds and latencies that prevailed in the ARPANET. 
![[Pasted image 20250306122008.png]]



> [!NOTE] Key Takeaway
> Why do talk about a decades old algorithm? Because it illustrates two lessons. The first is that computer systems are often *designed iteratively based on experience* and that it is better to deploy a simple solution sooner and expect to improve upon it later. The second is the KISS principle *(Keep it Simple, Stupid)*. Though having sophisticated optimizations are cool, the simplest solution is often the most robust. 

# Implementation


So far, we have describes what switches and routers must do without describing how they do it. 

This section gives an overview of both software-centric and hardware-centric designs. 

## Software Switch 

![[Pasted image 20250307110909.png]]

The above shows a software switch built using a general-purpose processor with four network interface cards (NICs). For a path where information is received on NIC 1 and then transmitted on NIC 2, the following could happen:
1. packets are received on NIC 1
2. NIC 1 copites its bytes directly into the main memory of the I/O bus (PCIe in this example) using a technique called *direct memory access* (DMA). 
3. The CPU examines the header to determine which interface the packet should be sent out on. 
4. The CPU instructs NIC 2 to transmit the packet out of main memory.

There are two bottlenecks with this approach. 

The first is that all information must pass into and out of memory. The second is that if packets are short, the cost of processing each packet is likely to dominate and potentially become a bottleneck. 


> [!NOTE] Key Takeaway
> The distinction between these two kinds of processing is important enough to give it a name - the *control plane* is the background processing required to "control" the network (eg. running OSPF, RIP, etc) and the *data plane* corresponds to the per-packet processing required to move packets from input port to output port. 

## Hardware Switch

The hardware and software switches have shifted to become more open source. 

![[Pasted image 20250307112237.png]]

The above is a white-box switch. The difference between this and a general-purpose processor is the addition of a Network Processor Unit (NPU) - a domain-specific processor that has been optimized for packet headers. 

The beauty of this new switch design is a white-box can now be programmed to be an L2 switch, an L3 router, or a combination of both. Exactly how one programs the NPU depends on the chip vendor.

NPU takes advantage of three technologies. First, a fast SRAM (Static Random Access Memory)-based memory buffers packets. Then, a TCAM-based memory stores bit patterns to be matches in the packets being processed. The "CAM" means that the key you want to look up in the table can be used as the address into the memory, and the T stands for ternary. Finally, the processing involved to forward each packet is implemented by a forwarding pipeline.

The benefit to having packet processing being implemented by a multi-stage pipeline is that though there is a latency for each stage, multiple packets can be processed at once. 

## Software Defined Networks

With switches becoming increasingly commoditized, attention is shifting to the software that controls them. This puts us in a tend of building *Software Defined Networks* (SDN).

The idea behind SDN is to decouple the network control plane (where routing algorithms like RIP, OSSPF, etc run) from the network data plane (where packet forwarding decisions get made), with the former going into software running on commodity servers and the latter by white-box switches. The key idea enabling SDN is to define a standard interface between the control plane and the data plane. This breaks the dependency on any one vendor's bundled solution. 

A key enabler for SDN's success is the Network Operating System (NOS), because it abstracts the details for the network switches and provides a *Network Map* to the application developer. This means that the app is free to simply implement the shortest path algorithm and load forwarding rules into underlying switches. 
![[Pasted image 20250307113907.png]]

> [!NOTE] Key Takeaway
> SDN is an implementation strategy. Instead of switches having to exchange messages to each other, the logically centralized SDN controller is charged with collecting link and port status information from individual switches.

# Perspective: Virtual Networks All the Way Down

What does it mean to virtualize a network?

Let's look at virtual memory as an example. Virtual memory creates an abstraction of a large and private pool of memory, though the physical memory may be partitioned by different applications. This allows programmers to operate under the illusion that there is plenty of memory. 

Similarly, server virtualization presents the abstraction of a virtual machine that has all the features of a physical machine. 

Virtualization lets users not interact with each other and preserves the abstractions and interfaces that existed before virtualization. 

VLANs are typically how we virtualize and L2 network, and helps to isolate different internal groups while giving them the appearance of having their own private LAN. However, there was problem. The 4096 possible VLANs were not sufficient to account for all the tenants that a cloud might host, and in a cloud the network needs to connect *virtual machines* rather than the physical machines that those VMs run on. VXLAN (Virtual Extensible LAN) was made to fix this problem (too complicated to explain now). Essentially, though, we have a VLAN encapsulated in a VXLAN overlay encapsulate in a VLAN. 

The hard part about virtual networks is the that networks are being nested inside networks (recursion). Another challenge is automating creation, management, etc of virtual networks. 
