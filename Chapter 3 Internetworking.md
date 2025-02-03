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
