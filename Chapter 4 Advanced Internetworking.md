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
