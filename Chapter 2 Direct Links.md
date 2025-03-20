# Connecting a network 

One of the fundamental problems we face is connecting two nodes together. Whether we want to construct a trivial 2-node network with one link or connect the billionth node to the Internet.

# 2.6 Multi-Access Networks

The Ethernet can be thought of as a bus that has multiple stations plugged into it. 

## Physical Properties

A transceiver is a small device directly attached to the tap, detects when the line is idle, and drives the signal when the host is transmitting. It also received incoming signals. 
![[Pasted image 20250123111637.png]]

Multiple Ethernet segments can be joined by repeaters. A repeater is a device that forwards digital signals. No more than 4 repeaters can be positioned between any pair of hosts. 
![[Pasted image 20250123111844.png]]

Whether a given Ethernet spans a single segment, a linear sequence of segments connected by repeaters, or multiple segments connected in a star configuration, data transmitted by any one host on that Ethernet reaches all other hosts. This causes problems with collision, as each host is part of the same *collision domain*. 

## Access Protocol

The algorithm that controls access to a shared Ethernet link is called the Media Access Control (MAC). We first discuss the Ethernet's frame format and addresses. 

### Frame Format
The following defines the Ethernet frame. 
![[Pasted image 20250123112204.png]]

The preamble allows the receiver to synchronize with the signal. The packet type servers as the demultiplexing key, and identifies to which of possibly many higher level protocols this frame should be delivered. A frame has at most 1500 bytes, and at minimum 46 bytes (this minimum helps with collision detection). The CRC is a bit framing protocol. 

### Addresses

Every ethernet host has a unique Ethernet address (technically, the address belongs to the adaptor not the host). An example of an address would be 8:0:2b:e4:b1:2. To ensure every adaptor gets a unique address, each  manufacturer of ethernet devices is allocated a different prefix that must be prepended to the address on every adaptor they build.

An ethernet adaptor receives all frames and accepts: 
- frames addressed to its own address
- frames addressed to the boradcaset address
- framed addressed to a mulitcast address, if is has been instructed to listen to that address
- all frames, if  placed in promiscuous mode
### Transmitter Algorithm

When an adaptor has a frame to send and the line is idle, it transmits the frame immediately. When an adaptor has a frame to send and the line is busy, it will wait for the line to go idle and hen transmit immediately. 

The Ethernet is said to be a *1-persistent* protocol because an adaptor with a frame to send transmits with probability 1 whenever a busy line goes idle. In general, a *p-persistent* algorithm transmits with probability $0\le p \le 1$ . We may choose a $p < 1$ because there may be multiple adaptors waiting for the busy  line to become idle, and we don't want then transmitting all the time. 

For Ethernet, it is possible for two adaptors to transmit at the same time, and this is when frames collide. At the moment an adaptor detects that its frame has collided, it sends a jamming sequence to stop transmission.

If the hosts are close to each other, the adaptor will send only 96 bits - called a *runt frame*, but in the worst case scenario a transmitter may need to send as many as 512 bits. Why 512 bits? This has to do with the fact that the farther two nodes are apart, the longer it takes for frames to arrive, and at this time the network is vulnerable to collision. 

![[Pasted image 20250123114947.png]]
Once an adaptor has detected a collision and stops transmission, it waits and then tries again. Each time it fails, the adaptor doubles the amount time it waits. This is known as *exponential backoff*. Actually what happens, is the algorithm selects a *k* between 0 and $2^n - 1$ and waits $k \times 51.2\micro s$ , where *n* is the number of collisions experienced so far. Adaptors typically retry up to 16 times. 

## Longevity of Ethernet

There are some properties that explain why the Ethernet has persisted. 

First, the Ethernet is easy to administer and maintain. It's plug and play. Second, it is inexpensive - the only costs are the cable/fiber and the network adaptor. 


> [!NOTE] Summary
> Ethernet uses transceivers and physical lines to transmit packets. It has a collision detection algorithm that attempts to send data in variable time periods. 

# Wireless Networks

There are some concerns that come up in wireless. You typically have less access to power, its inherently multi-access - giving concerns to eavesdropping and privacy - , and how much you are allowed to transmit. 

![[Pasted image 20250123120607.png]]

We will be using the stricter definitions of:
**bandwith** - the width of a frequency band
**data rate** - the number of bits per second that can be sent over the link

## Basic Issues

Because wireless links all share the same medium, the challenge is to share that medium without interfering with each other. This sharing comes from dividing along dimension and space. Certain entities are allowed to use a particular frequency in a particular area, and areas can be limited by weakening signals to reduce the distance information can be sent from its origin. 

One idea that shows up a lot when a spectrum is shared by many devices and application is *spread spectrum*. The idea is to spread the signal over a wider frequency band to minimize impact of interference. 

For example, *frequency hopping* is a spread spectrum technique that transmits the signal over a random sequence of frequencies (not truly random though). 

Another spread spectrum technique is called *direct sequence*, and it adds redundancy for greater tolerance of interference. For each bit the sender wants to transmit, it send the XOR of that bit and n random bits. 

![[Pasted image 20250123121402.png]]In many wireless networks today, there are two different classes of endpoints. One endpoint (sometimes described as the *base station*) has no mobility but has a wired connection to the Internet or other networks. 
![[Pasted image 20250123121955.png]]
Note that although this diagram shows a seeming directly link to the base station, these transmissions can be sensed by many other devices. 

There are also wireless mesh networks in which message are forwarded via a chain of peer nodes. This requires, however, a level of sophistication in hardware and software for non-base nodes. 

Now, lets look at two common wireless technologies. 

### 802.11/Wi-Fi

The main challenge of Wi-fi is to mediate access to a shared communication medium - ie space. 

#### Physical Properties

There are a lot of physical layer standards, and they have evolved over time to have many variants. All these variants have a maximum bit rate that is defined, but you can typically use lower bit rates. Lower bit rates usually allow for signals to be more easily decoded, so if you have a place with a lot of noise, a lower bit rate will be more consistent. 

#### Collision Avoidance

Unlike Ethernet, all nodes are not connected to each other, which makes collision detection a little harder.

![[Pasted image 20250127111619.png]]

Looking at the figure above, A and C are *hidden nodes* in respect to each other because they are out of range of each other, but their frames collide at B. 
![[Pasted image 20250127111733.png]]
In this diagram, if B sends to A, although C can hear B it should not prevent C from transmitting to D. 

802.11 addresses these problems using CSMA/CA, where the CA stands for collision *avoidance*. 

In 802.11, waiting for an absence of signal does not imply no collision, as seen by our figures. Instead, the receiver has to send a ACK back to the sender. 

In addition, 802.11 has a optional mechanism called RTS-CTS(Ready to Send-Clear to Send). The sender sends an RTS - a short packet - to the receiver. The receiver then sends another short packet - the CTS - which can be heard from all nodes within the receiver's range. This CTS tell these neighboring nodes that it will be busy for a specified amount of time, and to try sending packets once that time period has ended. 

If two nodes send an RTS frame at the same time, they will not receive a CTS frame and then have to wait a random amount of time defined by the exponential backoff algorithm. 

#### Distribution System

So far, we have been discussing with a mesh topology in mind. At the current time, however, most 802.11 networks use a base-station-oriented topology. 

In this case, some nodes can roam (like your laptop) and some are connected to a wired network infrastructure. 802.11 calls these base stations access points(APs) and are connected by a *distribution system*. 
![[Pasted image 20250127112826.png]]The distribution network operated at the link layer. 

The idea behind this approach is that if node A wants to communicate to node E, it will forward its request to AP-1, which sends that frame to the distribution system, which then sends the frame to AP-3, and then gets sent to E. 

802.11 specifies how nodes select access points and how these choices work in light of nodes moving from one cell to another. 

The technique for selecting an AP is called scanning and it has four steps: 
1. The node sends a **Probe** frame. 
2. All APs within reach reply with a **Probe Response** frame. 
3. The node selects one of those APs and sends it an **Association Request**. 
4. The AP replies with an **Association Response** frame. 

![[Pasted image 20250127113325.png]]
Since a node is actively searching for an AP, this is called active scanning. Alternatively, an AP can periodically send a **Beacon Frame** to advertise its capabilities, and a node can change to this AP by sending an **Association Request** back to the access point. 

#### Frame Format

Most of the frame format is exactly what we would expect: 
![[Pasted image 20250127113548.png]]
A peculiar thing is that there are four addresses rather than two (source, destination). This is to account for the fact that the original sender may have changed. In the simplest case, both **DS** bits are 0, indicating that **Addr1** is the target and **Addr2** is the source. In the most complex, both **DS** bits are set to 1, indicating that a message went to the distribution system and then to another wireless nodes. In this case, **Addr1** is the ultimate destination, **Addr2** the immediate sender (from distribution to dest), **Addre3** the intermediate destination (from source to dist) , **Addr4** the original source.

### Security of Wireless Links
There are some mechanisms you can use to control access to both link itself and transmitted data. 

#### Bluetooth (802.15.1)

The basic Bluetooth network configuration, called a **piconet**, consists of a master device and 7 slave devices. Any communication is between master and slave; the slaves do not directly communicate with each other. 
![[Pasted image 20250127114644.png]]
# Access Networks

This section will describe *Passive Optical Networks* and *Cellular Networks* . In both cases, the networks are multi-access, but the mediating access is quite different. 

## Passive Optical Network

PON adopts a point-to-multipoint design, which means the network is structured as a tree, with a single point starting in the ISP's network and then fanning out to reach up to 1024 homes. 
![[Pasted image 20250127120910.png]]
The figure above shows an example PON, simplified to depict just one ONU (optical network unit) and one OLT (Optical Line Terminal).

Because splitters are passive, PON has to implement some form of multi-access protocol. The approach is as follows. Firstly, up and downstream traffic are transmitted at different frequencies so they are independent of each other. Downstream traffic starts at the OLT, and the signal is propagated down every link in the PON. Therefore, every frame reaches every ONU. The device then looks at a unique identifier in the individual frames sent over the wavelength and either keeps or drops the frame. 

Upstream traffic is time-division multiplexed, where each ONU periodically gets a turn to transmit. Because the ONUs are fairly far apart, the transmission is not based off of synchronized clocks, but rather the OLT transmits grants to individual ONUs, giving them a time interval during which they can transmit. 

## Cellular Network

Like Wi-Fi, cellular networks transmit data at certain bandwidths in the radio spectrum.

Traditional cellular technologies range from 700-MHz o 2400 MHz. 

Like 802.11, cellular technology relies on base stations connected to a wired network, often called *Broadcast Base Units (BBU)*, and the mobile devices connected to them are usually referred to as *User Equipment (UE)*, ad the set of BBUs are anchored at an *Evolved Packet Core (EPC)* hoseted in a central offfice. The wireless network served by the EPC is often called a *Radio Access Netword (RAN)*. 

BBUs officially go by another name - Evolved NodeB (eNodeB or eNB) 
![[Pasted image 20250128113722.png]]

The above figure shows one possible configuration of the end-to-end scenario. 

The geographic area served by a BBU's antenna is called a *cell*. A BBU could serve a single cell or use mulitple directional antennas to serve multiple cells. Cells can overlap, thus causing a user to communication with more than one cell. At the same time, the UE can only be under the control on one BBU. As the device leaves once cell and goes to another, BBUs seize and cease control according to that of the strongest signal. If the device is involved in a call or other network session at the time, the session must be transferred to the new base station in what is called a *handoff*. 

There have been multiple generation of protocols implementing the cellular network, known as 1G, 2G, 3G, and so on. 

As of 3F, the generational designation actually corresponds to a standard defined by the 3GPP (3rd Generation Partnership Project), and the 3GPP continue to define the standard for 4G and 5G. 

The sequence of releases and generations is called LTE (*Long-Term Evolution*), and the industry as a whole has been on a fairly well-defined evolution path known as LTE. 

The main innovation of LTE is how it allocation the available radio spectrum to UEs. The difference between LTE's strategy and WiFi's strategy in this regard is bases off an assumption about utilization. Wi-Fi assumes a lightly loaded network (and thus optimistically transmits when the link is idle) while cellular assumes high utilization (explitly assigning different users to different shares of the available radio spectrum).

The mechanism for LTE is called *Orthogonal Frequency Division Multiple Access (OFDMA)*. The idea is to multiplex data over a set of 12 orthogonal subcarrier frequencies, each modulated independently. The "Multiple" Access implies that data can be simultaneously sent on behalf of multiple users, each on a different frequency for a different duration of time. 

OFDMA leads to conceptualizing the radio spectrum as a two-dimensional resource. 
![[Pasted image 20250128120200.png]]

A scheduler makes allocation decisions at the granularity of blocks of 7x12=84 resource elements, called a *Physical Resource Block*. 

The 1ms *Transmission Time Interval (TTI)* corresponds to the time frame in which the BBU received feedback from UEs about the quality of the signal they are expecting. The *Channel Quality Indicator (CQI)* reports the observed signal-to-noise ratio. The base station uses this informatio to adapt how it allocates the available radio spectrum to the UEs its serving. 

This description of sceduling radio spectrum is specific to 4G. Transitioning from 4G to 5G introduces additional degrees-of-freedom in how the radio spectrum is scheduled. 

# Perspective: Race to the Edge

Softwarization is radically transforming the network. Meany of the fiber-to-the-home and cellular networks described are from complex hardware appliances that have a bundled set of functionality, making them slow to change. 

In response, network operators are transitioning to appliean with open software running on commodity services, switches, and access devices. This initiative is called CORD. 