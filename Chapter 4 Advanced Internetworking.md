# Global Internet

At this point, we have seen how to connect a bunch of heterogeneous collection of networks to create an internetwork and to use the simple hierarchy of the IP address somewhat scalable. We now look at techniques that makes routing protocols more scalable. 
![[Pasted image 20250310110818.png]]

The above is a general picture of how the Internet may look like. In the modern world, though, the NSFNET backbone is more or less replaced with commercial providers who have built dozens of high-end routers located in major metropolitan areas. 

Notice that each provider and end-user are likely to be an administratively independent entity.  Because of this, there mya be differences in each provider about what best routing protocol to use or what metric is the best. We call each provider's network an *autonomous system* (AS) - a network administered independently of other ASs. 

We look at two scaling issues. The first is routing - we want to minimize the number of network numbers that get carried around in routing protocols and stored in routing tables. The second is address utilization - we don't want the IP address space to get consumed too quickly. 

## Routing Areas

An area is a set of routers configured to exchange link-state information with each other. 
![[Pasted image 20250310111916.png]]
There is one special area, known as backbone area or area 0. A router on the backbone area and a nonbackbone area is a area border router (ABR). 

Routing in a single area is exactly as described before: routers send link-state advertisements to construct a map of the area. However, link-state advertisements do not leave their specific area unless you are a border router. This means that a router does not know the entire topological order of the overall network. 

How does a router determine the next hop for a network in another area? Lets imagine the path of a packet that has to trabe from one nonbackbone are to another. First, it travels from its source network to the backbone area. Then, it crosses the backbone. Finally, it travels from the backbone to the destination network. 

To make this work, the area border routers summarize routing information from one area and make it available in their advertisements to other areas. For example, the information received by R1 can be seen as the costs of reaching the networks in area 1 as if all the routers were directly connected to R1.

In the case where there are two ABRs, routers in that area will have to make a choice of which one they use to various networks, which is easy since those routers will be advertising costs. 

When dividing a domain into areas, the network administrator trades between scalability and optimality of routing. Areas force packets to go to the backbone even if a more optimal path may have been available. It turns out, though, scalability is often more important than optimization. 

Network administrators can use a *virtual link* to decide which routers go in area 0. For example, R1 could share routing information with R8, so R8 is part of the backbone even though it is not directly connected. 

### Interdomain Routing (BGP)

We mentioned that the Internet is organized as a bunch of autonomous systems. 

![[Pasted image 20250310113547.png]]

Autonomous systems allow us to hierarchically aggregating routing information. We divide the routing problem into two parts - routing within an AS and routing between ASs. Since ASs can also be referred to as *domain*, we refer to the two parts of the routing problem as interdomain routing and intradomain routing. 

#### Challenges in Interdomain Routing

Perhaps the most important challenge of interdomain routing today is for each AS to determine its own routing *policies*. For example, a routing policy could look like this:

"Whenever possible, I prefer to send traffic via AS X than via AS Y, but I’ll use AS Y if it is the only path, and I never want to carry traffic from AS X to AS Y or vice versa."

A key design of interdomain routing is that policies should be supported by the interdomain routing system. In addition, I need to be able to implement such a policy without any help from other ASs, and in the face of other ASs messing up.  Also, there is often a desire to keep the policies *private* for competition reasons. 

There have been two major interdomain routing protocols. The first was the Exterior Gateway Protocol (EGP), which was bad for scalability. 

The replacement was the Border Gateway Protocol (BGP). BGP makes no assumptions about how ASs are interconnected. The model is general enough to accommodate tree-structured internetworks (look at the first image in this document). 

The internet today is fairly complex, so we define a few terms to address that complexity. *Local traffic* is traffic that originates at or terminates on nodes within an AS. *Transit* traffic is traffic that passes through an AS. We classify ASs into three broad types:
- Stub AS - AS with only one connection one one other AS; will only carry local traffic (small corporation)
- Multihomed AS - an AS with connections to more than one other AS, but refuses to carry transit traffic (large corporation)
- Transit AS - AS with connection to more than one other AS and carries both transit and local traffic (backbone provider)
![[Pasted image 20250310114823.png]]
The goals of interdomain routing is rather complex because our optimal path has a couple of requirements. First, it needs to find *some* path that is loop free. Second, paths must compliant with the various policies of the ASs along the path. Hence, we need to find the most optimal non-looping, *policy-compliant* path. 

Interdomain routing is also hard because of scale. The Internet backbone router has to forward any packet anywhere in the Internet. 

Furthermore, the autonomous nature of the domains means that it is impossible to calculate meaningful path costs for a path that crosses multiple AS since each domain has its own scheme for metrics. That means that a cost of 1000 may be acceptable to one AS, but not acceptable to another AS. Hence, interdomain routing advertises only reachability. 
#### Basics of BGP

Each AS has one or more *border routers* thrugh which packets enter and leave the AS. A border router is simply an IP router that is charged with the task of forwarding packets between ASs. 

Each AS that participates in BGP must also have at least one BGP speaker, a router that "speaks" BGP to other BGP speakers in other ASs. 

BGP does not belong to the two main classes of routing protocols: distance-vector nor link-state. Rather, BGP advertises *complete paths* as an eumerated list of ASs to reach a particular network. This is sometimes called a *path-vector* protocol. These complete paths allow for many policy decisions made in accordance with AS and also enables loops to be readily detected. 
![[Pasted image 20250317112050.png]]

We see this an action. Assume that providers are transit networks, while customer networks are stubs A BGP speaker for the AS of provider A would be able to advertise reachability information for each of the network numbers assigned to customers P and Q. 
![[Pasted image 20250317112421.png]]

An important job of the BGP is to prevent the establishment of loops. We look at the figure above. Suppose AS 1 learns that it can reach network 128.96 through AS 2. It advertises this to AS 3, who then advertises back to AS 2. Now, AS 2, for packets addressed to 128.96, might send the packet to AS3, which would then be sent to AS 1, which would then be sent to AS2, causing a loop. The solution to this would be if a router sees itself in a complete path, it knows it should not use that path. 

Clearly, AS numbers need to be unique, and are assigned by a central authority to assure uniqueness. 

An AS will only advertise routes that it considers good enough for itself, meaning it will only advertise the best route according to it s local policies. A BGP speaker, though, is not under any obligation to advertise a route, even if it has one. 

Given that links fail and policies change, BGP speakers need to be able to cancel previous advertised paths. They do this with a negative advertisement known as a *withdrawn route*. Both positive and negative reachability information are carried in a BGP update message. 

![[Pasted image 20250317113229.png]]

Unlike the routing protocols described before, BGP is defined to run on to p of TCP, the reliable transport protocol. Because BGP speaker can rely on TCP, then CGP speakers only need to send an occasional *keepalive* message.

#### Common AS Relationships and Policies

The three common relationships and policies between ASs are:
- *Provider-Customer* - Providers want to connect their customer to the rest of the Internet, so a common policy is to advertise all routes I know to the customer, and advertise all routes I learn from my customer to everyone. 
- *Customer-Provider* - The customer want to get traffic directed to them, and wants to be able to send traffic to the Internet. A common policy would be to advertise my own prefixes and routes learned from my customer to my provider, advertise routers learned from my provider to my customers, but don't advertise routes learned from one provider to another. This is to make sure the customer doesn't find themselves in the business of carrying traffic from one provider to another.
- *Peer* - Two providers who see themselves as equals usually peer to get access to each other's customers without having to pay another provider. The policy here is to advertise routes learned from customers to my peer, advertise routes learned from my peer to my customers, but don't advertise routes from my peer to any provider or vice versa. 
![[Pasted image 20250317115553.png]]
We can see that this figure gives us some structure to the Internet. At the bottom are stub networks, or customers of one or more providers. As we move up, we see providers who have other providers as their customers. At the very top are providers with only customers or peers. 


> [!NOTE] Key Takeaway
> How does all this help in building scalable networks? First, the number of ASs is much smaller than the number of networks. Second, a good interdomain route is finding a path to the right border route, of which there are only a few per AS. Hence, we are using hierarchy to subdivide our problem, and increase scalability. The complexity of interdomain routing is the number of ASs, and the intradomain is focused on a singular AS. 

#### Integrating Interdomain and Intradomain Routing

The question of how do all other routers in a domain get interdomain routing information remains. 

Let's start simple. In the case of a sub AS that only connects to other ASs at a single point, the border router is the only choice for all routes outside the AS. Such a router is a *default route*. 

The next step is to have border routers inject specific routes that they have learned from outside the AS. Consider, for example, the border router fo a provider AS that connects to a customer AS. The router could learn the network prefix 192.4.54/24 is located inside the customer AS, either through BGP or because the ifnormation is configured into the border router. It could inject a route to that prefix into the rougin protocol running inside the provider AS. This would be an advertisement of sort, and cause other routers in the provider AS that the border router is the place to send packets destined for that prefix. 

Finally, the backbone networks. Because backbone networks receive so much routing information, it uses *interior BGP* or *iBGP* to navigate through this information. iBGP enables any router in the AS to learn thebest border router to use when sending a packet to any address. At the same time, each router knows how to get to each border router using a conventional intradomain protocol with no injected information.
![[Pasted image 20250317121740.png]]


![[Pasted image 20250317121804.png]]
# IPV6

The motivation for IPV6 is to deal with the exhaustion of the IP address space. 
## Historical Perspective

Since the IP address is carried in every packet, changing the size of this address is no small nor simple matter. 

Since changing the IP address is so huge, network designers decided that they might as well change other things as well, including: 

- Support for real-time services
- Security support
- autconfiguration
- enhanced routing functionality, including support for mobile hosts

Although these were features meant to be introduced in IPV6, they have made their way into IPV4. In addition, they anticipated the this transition would be quite gradual, though few anticipated that the transition would be reaching it's third decade of transitioning. 

## Addresses and Routing

IPv6 includes a 128-bit address space. This means that there will be over 1500 adresses per square foot of Earth's surface. 

### Address Space Allocation

IPv6 does not use classes, but does have way to subdivide address space based on the first bits. 

![[Pasted image 20250318111806.png]]

The "everything else" category includes all the IPv4's three main address classes. 

The idea behind link-local use addresses is to enable a host to construct an address that will work on the network to which it is connected without being concerned about the global uniqueness of the address. 

#### Address Notation

The standard represnetation of IPv6 is x:x:x:x:x:x:x:x.

IPv6 can be shortened in a number of ways. For example, an address with a large number of contiguous zeros can be more compactly written by omitting the zeros. For example, 

47CD:0000:0000:0000:0000:0000:A456:0124

can be written as 

47CD::A456:0124

IPv6 addresses with an embedded IPv4 address have their own special notation. For example, 128.96.33.81 could be written as ::FFFF:128.96.33.81.

#### Global Unicast Addresses

The most important sort of addressing in IPv6 is plain old unicast addressing. It must do this in a way that supports the rapid rate of addition of new hosts to the Internet and that routing is done in a scalable way. 

To help us understand how this is done, we introduce some new terms. We can think of a nontransit AS (i.e. a stub or multihomed AS) as a *subscriber* and we may think of a transit AS as a *provider*. Furthermore, we may divide providers into *direct* and *indirect*. The former is connected to subscribers. The latter is often known as *backbone networks*.

With this set of definitions, we can see the Internet as intrinsically hierarchical. The difficulty is using this hierarchy without inventing mechanisms that fail when the hierarchy is not strictly observed, such as when the distinction between direct and indirect provider becomes blurred. 

Like CIDR, IPv6 allocation plan wants to provide aggregation of routing information to reduce the burden on intradomain routers. Again, the idea is to use an address prefix to aggregate reachability information to a large number of networks and ASs. The main way to achieve this is to assign an address prefix to a direct provider, and have that provider handles its subscribers. 

The drawback is that if a site decides to change providers, it will need to obtain a new address prefix and renumber all nodes in the site, which is super annoying. There is research about other addressing schemes - like geographic addressing, where a site's address is a function of its location. For now, though, we have provider-based addressing. 

One place where aggregation makes sense is at the national or continental level. If all addresses in Europe, for example, had a common prefix, then a great deal of aggregation could be done. 

A tricky solution occurs when a subscriber is connected to more than one provider. Which prefix should the subscriber use for his or her site? Suppose a subscriber is connected to two providers, X and Y. If the subscriber takes the prefix from  X, then Y has to advertise a prefix with no relationship to its other subscribers and therefore cannot be aggregate. If we split the prefix between X and Y, if one site goes down then half a user's site may go down if the connection to one provider goes down. One solution is that if X and Y have many customers, then they can agree on a prefix they will share. 

## Packet Format

Despite that IPv6 extends IPv4, its header format is actually simpler due to a concerted effort to remove unnecessary functionality from the protocol. 

![[Pasted image 20250320114424.png]]

As with many headers, this one starts with a *Version* field, which is set to 6. The *PayloadLen* gives the length of the packet in bytes. The *NextHeader* field replaces required options and special headers. 

Finally, the bulk of the header is taken up with the source and destination addresses, each of which are 16 bytes long. Hence, an IPv6 header is always 40 bytes lings. 

The way that IPv6 handles options is quite an improvement over IPv4. Previously, options were a mess at the end that routers had to completely parse through to find relevant information. Now, IPv6 treats options as extension header that must appear in a specific order, making it easy to determine relevant options. 

Each option has its own type of extension header. The type is identified by the *NextHeader* field in the header that preceded it, and each extension headers contains a *NextHeader* field to identify the header following it. The extension header serves as a demux key to identify the higher-layer protocol running over IPv6.

![[Pasted image 20250320115833.png]]

## Advanced Capabilities

The primary motivation behind IpV6 is to support the continued growth of the Internet. Since changing the header is itself a huge undertaking, IPv6 includes a large number of other improvements as well. It should be noted that IPv4, itself, has come to include many of these improvements. 

#### Autoconfiguration

While Internet growth has been impressive, the fact that getting connected to the Internet requires a fair amount of system administration expertise - in particular, that every host needs to be configured with a certain minimum amount of information, like a valid IP address, a subnet mask, and the address of a name server. IPv6 is meant to autoconfigure. 

With IPv4, autoconfiguration was possible but required a central server. With IPv6, a server is not required. 

Recall that IPv6 unicast addresses are hierarchical, and the least significant portion is the interface ID. Thus, we can subdivide the autoconfiguration problem into two parts:
1. obtain an interface ID unique on the link to which the host is attached. 
2. Obtain the correct address prefix for this subnet.

The first part turns out to be easy, since every host on a link must have a unique link-level address. 

#### Source-Directed Routing

IPv6 also has a routing header. This header contains a list of addresses that represent nodes or topological areas that the packet should visit en route to its destination - like a backbone provider. To specify topological entities rather than individual nodes, IPv6 defines an *anycast* address. An anyc ast addressed is assigned to a set of interfaces, and packets sent to that address will go to the 'nearest' of those interfaces. 

# Multicast

In situation when once host want to send the same data to multiple recipients, then typical IP communication where each packet is addressed to only one host, which we call *unicast* , is not suitable. The basic IP multicast modeal is a many-to-many model based on multicast *groups*, where each group has its own *mulitcast address*. Members of a group receive copies of packets sent to a multicast address. Using IP multicast, in comparison to unicast IP, is more scalable because it reduces redundancy in how many packet that would have been sent. 

IP's original many-to-many multicast has been supplemented with support for a form of one-to-many mulitcast, called the *Source-Specific Multicast* (SSM) - where a receiving host specifies both a multicast group and a specific sending host. The receiving hsot would then receive multicasts addressed to the specific group, but only if they are from the specified sender. IP's original many-to-many model is sometimes referred to as *Any Source Multicast* (ASM). 

A host signals its desire to leave a multicast group by communicating with the local router using a special protocol. In IPv4, that protocol is the *Internet Group Management Protocol* (IGMP); in IPv6, it is *Mulitcast Listener Discovery* (MLD). The router then has the responsibility to behave correctly. If a host fails to leave a multicast group when it should, the router periodically polls the network to determine which groups are still of interest to the attached hosts. 

## Multicast Addresses

Multicast addresses have a reserved space in IPv4 and IPv6. In IPv4, there are 28 bits of possible multicast addresses, which presents a problem when trying to take advantage of hardware multicasting on a local area network. 

Lets take the example of the Ethernet. Ethernet multicast addresses only have 23 bits when we ignore their shared prefix. Thus, in order to take advantage of Ethernet multicasting, IP has to map 28 bits IP multicast addressing into 23 bit multicast addressing. This is done by ignoring the high-order 5 bits. 

When a host on an Ethernet joins an IP multicast group, it configures its Ethernet interface to receive any packets with the corresponding Ethernet multicast address. Unfortunately, this causes the receiving host to receive not only the multicast traffic it desired but also traffic sent to any of the other 31 multicast groups that map to the same Ethernet address (since we have to map from big to small). Hence, the receiving host must examine the IP header of any multicast packet to determine whether the packet really belongs to the desired group. There are some switched networks where the switches can recognize unwanted packets. 

## Multicast Routing

A router must have multicast forwarding tables that indicate links - possibly more than one - to forward the multicast packet (possibly duplicating the packet over multiple links). Thus, instead of a table that specify a set of paths like unicast routing, multicast specify a set of trees: *multicast distribution trees*. To support Source-Specific Multicast, the multicast forwarding tables must indicate which link to use based on the multicast address and the unicast IP address of the source, against specifying a set of trees. 

To be specific, multicast routing is the process by which the multicast distribution trees are determined the process by which the multicast forwarding tables are built. 

### DVMRP

Distance-vector routing used in unicast can be expanded to multicast. The resulting protocol is called *Distance Vector Multicast Routing Protocol*, or DVMRP. DVMRP is a *flood and prune* protocol, where a packet is forwarded to all networks on the Internet, and then is pruned to have only hosts that belong to the multicast group. 

There are two major shortcomings. The first is that it truly floods the network. The second is that a given packet will be forwarded over a LAN by each of the routers connected to that LAN. 

The solution to the second limitation is to eliminate the duplicate broadcast packes generate when more than one router is connected to a given LAN. One way to do this is to designate one router as the *parent* router for each link - relative to the source - where only the parent router is allowed forward multicast packets from that source over the LAN. This mechanism is called the *Reverse Path Broadcast* (RPB) or *Reverse Path Forwarding* (RPF). It is reverse because we are considering the shortest path toward the *source* when making forwarding decisions, rather than to a destination. 

The RPB mechanism described implements a shortest path broadcast. We want to prune the set of networks that receives each packet addressed to group G to exclude those who have no hosts that are members of G. This is done in two stages. We have to recognize when a *leaf* network has no group members. Then, we have to propagate a "no members of G" here information up the shortest-path tree. 

Note that this is quite expensive to do, and in practice information is only exchanges when some source starts sending packets to that group. 

### PIM-SM

*Protocol Independent Multicast* - **PIM** - was developed in recognition of scaling problems. For example, broadcasting traffic to all routers until they explicitly ask to be removed is not a good choice if most routers don't want the traffic in the first place. Hence, PIM divides the problem space into *sparse mode* and *dense mode*. PIM dense mode (PIM-DM) uses a flood-and-prune algorithm that suffers in scalability. PIM sparse mode (PIM-SM) has become the dominant multicasting routing protocol, and will be our central topic. 

In PIM-SM, routers explicitly join the multicast distribution tree using PIM protocol messages known as *Join* messages. The question is where to send those *Join* messages, since any host could send to the multicast group. PIM-SM assigns each group a special router known as the *rendevous point* (RP). A number of routers are configured to be candidate RPs, and PIM-SM decides which router to use as the RP for a given group.  

When a router sends a *Join* message, it is sent using a normal IP unicast transmission. The intitial *Join* message is "*wildcarded*" - it applies to all senders. Each router along the path of this packet creates a forwading table entry for the shared tree, called a (\*, G) entry. Eventually, the shared tree will have a solid line from RP to R4. 

As more routers send *Join*s toward the RP, it causes new branches to be added.

![[Pasted image 20250331112731.png]]
Suppose a host wishes to send a message to the group. To do so, it sends a packet with the appropriate mulitcast froup address as its address and then sends it to a router on its local network known as the *designated router* (DR). This DR *tunnels* the packet to the RP, and on receiving the packet, will send it out onto the shared tree, of which the RP is the root. 
![[Pasted image 20250331113203.png]]

However, tunneling is not the most efficient. Instead, the RP could send a *Join* message toward the sending host, and as this *Join*message travels toward the host, it causes to router along the path to learn about the group, making it possible for the DR to send the packet to the group as *native* (not tunneled multicast) packets. Because this *Join* is specific to the sender, we indicate this route as a source-specific route.

PIM is protocol dependent because it does not depend on any particular routing protocol. However, PIM is very much bound up with IP - it is not protocol independent in terms of network-layer protocols. 

### Interdomain Multicast (MSDP)

PIM-SM does have some significant shortcomings. In particular, the existence of a single RP for a group goes again the principle that domains are autonomous. For a given multicast froup, all the participating domains would be dependent on the domain where the RP is located. Hence, PIM-SM is typically only used within a domain. 

The Multicast Source Discovery Protocol (MSDP) was devised to multicast across domains. It is used to connected different domain - each running PIM-SM internally - by connecting the RPs of the different domains. Each RP has one or more MSDP peer RPs in other domains. Each pair is connected by a TCP connection over which the MSDP protocols runs. Together, all the MSDP peers form a loose mesh called a broadcast network. MSDP messages are broadcast through the mesh of peer RPs using the Reverse Path Broadcast algorithm. 

Source information is sent through this mesh of RPs. 

![[Pasted image 20250331120020.png]]

If an MSDP peer RP receives one of these broadcasts has active receivers for that multicast group, it sends a source-specific *Join* to the source host. The *Join* adds a branch of the source-specific tree to this RP. The result is that every RP part of the MSDP network and has active receivers for a particular multicast group is added tot he source-specific tree of the new source. 

### Source-Specific Multicast (PIM-SSM)

The original service model of PIM was a many-to-many model. It was recognized in the late 1990s that it might be useful to add a one-to-many model. After all, lots of multicast applications only have one legitimate sender. In the original PIM design, only routers joined source-specific trees. However, once the need for a one-to-many service model was recognized, it was decided to make the source-specific routing capability of PIM-SM available to hosts. This newly exposed capability is now known as PIM-SSM (PIM Source-Specific Multicast). 

PIM-SSM introduces a new concept, the *channel*, which is a combination of a source address S and a group address G. G looks like a normal IP multicast address. To use PIM-SSM, a host specifies both the group and the source in an IGMP Membership Report message to its local router. The router then sends a PIM-SM source-specific *Join* message toward the source, thereby adding a branch to itself in the source-specific tree, bypassing the whole shared tree stage. Since the tree is source specific, only the designated source can send packets on that tree. 

The introduction of PIM-SSM has provided some significant benefits: 
- multicasts travel more directly to receivers
- the address of a channel is effectively a multicast group address plus a source address, so multiple domains can use the same multicast group address independently and without conflict, as long as they use it only with sources in their own domains. 
- because the specified source only can send to an SSM group, there is less risk of attacks based on malicious hosts overwhelming the routers or receivers with bogus multicast traffic.
- PIM-SSM can be used across domain exactly as it is used within a domain, without reliance on anything like MSDP.
#### Bidirectional Trees (BIDIR-PM)

BIDIR-PIM is a recent variant of PIM-SM that is well suited to many-to-many multicasting within a domain, especially when senders and receivers to a group may be the same, like in a multiparty video conference. Like in PIM-SM, would-be receivers join the group by send IGMP (which must not be source specific) and a shared tree rooted at an RP is used to forward multicast packets to receivers. Unlike PIM-SM, the shared tree also has branches to the *sources* since it is bidirectional.
![[Pasted image 20250402121621.png]]
For example, in 4.17(b) R4 forwards a multicast packet downstream to R2 at the same time that it forwards a copy of the same packet upstream to R5. 

A surprising  apsect of BIDIR-PIM is that a RP is not needed. All that is needed is a routable address, which is known as an RP address even though it need not be the address of an RP or anything at all. For example, 4.17(a) shows a *Join* from R2 terminating at R5, and a *Join* from R3 terminating at R6. The upstream forwarding of a multicast packet similarly flows toward the RP address until it reaches a router with an interface on the link where the RP address would reside, but then the router forwards the mulitcast packet onto that link as the final step of upstream forwarding, ensuring that all other routers on that link receive the packet. 

BIDIR-PIM cannot be used across domains, but has many advantages over PIM-SM for many-to-many mulitcast within a domain:
- there is no source registration process because router already know how to route a mulitcast packet toward the RP address. 
- The routes are more direct because they go only up the tree as far as necessary
- they use less state the source specific trees of PIM-SM because there is never any source-specific state.
- The RP cannot be a bottleneck
# Multiprotocol Label Switching

The *Multiprotocol Label Switching* (MPLS) combines some of the properties of virtual circuits with the flexibility and robustness of datagrams. On one hand, MPLS is very much associated with the IP's datagam, on the other, MPLS-enabled routers also forwards packets by examining relatively short, fixed-length labels, and these labels have a local scope, like in a virtual circuit network. 

There are three main things MPLS is used for: 
- To enable IP capabilities on devices that do not have the capability to forward IP datagrams in a normal manner
- to forward IP packets along explicit routes
- to support certain types of VPN services

## Destination-Based Forwarding
![[Pasted image 20250409110700.png]]

Consider the figure above. Each of the routers on the far right has one connected network. The remaining router (R1 and R2) have routing tables indicating which ougoinginterface each router would use when forwarding packets to one of those two networks. 

When MPLS is enabled on a router, the router allocates a label for each prefix in its routing table and advertise both label and prefix to its neighboring routers. This advertisement is carried in the Label Distribution Protocol. 
![[Pasted image 20250409111135.png]]

(4.19)

R2 has allocated the label value 15 for the prefix 18.1.1 and 16 for the prefix 18.3.3. After allocating the labels, R2 advertises the label bindings to its neighbors, so we see R2 advertising a binding between the label 15 and the prefix 18.1.1 to R1. In effect, R2 has said: "Please attach the label 15 to all packets sent to me that are destined to prefix 18.1.1."

Hence, when we send packets out, many routers only look at labels and send out packets in accordance to other labels. What's the point of this? 

We are replacing a normal IP destination address lookup with a label lookup. This is helpful because IP address lookup algorithm needs to find the *longest match*, whereas a label forwarding mechanism is an exact match algorithm. 

MPLS label is associated with a *forwarding equivalence class* (FEC) - a set of packets set to receive the same forwarding treatment in a particular router. In this example, each prefix in the routing table is an FEC - that is, all packets that match the prefix 18.1.1 get forwarded along the same path. 

Changing the forwarding algorithm from normal IP forwarding to label swapping has an important consequence: Devices unable to forward IP packets - like ATM switches - can be used to forward IP traffic in an MPLS network by being turned into *Label Switching Routers* (LSRs). 

Before we consider the benefits of turning an ATM switch into an LSR, we should tie up loose ends. Labels are attached to packets, but the type of link in which the packets are carried changes what we mean by this. 
![[Pasted image 20250409120710.png]]
When IP packets are carries as complete frames - like in Ethernet and PPP - the label is inserted as a "shim" between the layer 2 header and the IP header. If an ATM switch is to function as an MPLS LSR, then the label is in the ATM cell header, where the VCI and VPI usually is. 

Now we can built a network using a mixture of conventional IP routers, label edge routers, and ATM switches functioning as LSRs that all use the same routing protocols.
![[Pasted image 20250409121113.png]]
We can see that the benefits of have LSRs is that routers have fewer adjacencies (meaning routing neighbours)  and that edge routers have a full view of the topology og the network. 

Note that replacing ATM switches is done purely by switching protocol - no hardware change need be done. 

## Explicit Routing
