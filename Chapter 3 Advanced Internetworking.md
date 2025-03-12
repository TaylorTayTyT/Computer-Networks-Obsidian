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

