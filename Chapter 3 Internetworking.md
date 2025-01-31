**Problem: Not All Networks are Directly Connected**

To build a global network, we need a way to interconnect different types of links and multi-access networks - this is called internetworking. 

We can divide this problem up into subproblems. Firstly, we need a way to interconnect links. Devices that interconnect links of the same type are often called switches, or layer 2 (L2) switches. 

The point of a switch is to take packets that arrive on an input and forward them to the right output. 

*Routers* - or *gateways* - deal with interconnection of disparate network type - like the IP.

Once everything is connected through switches and routers, there could be multiple ways to get from one point to another. Such paths should be dynamic and loop free. 

# Switching Basics

A switch si a mechanism that allows us to interconnect links to form a larger network.  It is a multi-input, multi-output devices that transfers packets from an input to one or more outputs.