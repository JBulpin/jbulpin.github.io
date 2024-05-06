## SRV6 and why you should care...

![SRv6 - SR-MPLS.png](..%2FSRv6%20-%20SR-MPLS.png)


If you are a tech enthusiast or work in the Service provider space its likely you have awareness of Segment-Routing or SRv6. I find that most customers or colleagues have knowledge of SR-MPLS but very few understand SRv6 and how it works. Many engineers are either IPv6 Evangelists or are still very intimidated by the protocol and subnetting, I’m hoping SRv6 is a strong reason to get familiar with IPv6 as its fundamental to building your transport.

I aim to demystify this concept at a basic level and provide understanding for a common use case, L3VPN for IPv4 traffic which most engineers are strongly familiar.

![SR-MPLS.png](Images%2FSRv6%2FSR-MPLS.png)

In MPLS we have the notion of labels which aid in transporting between prefixes
Provider Edge(PE) nodes advertising prefixes amongst each other, these are delivered using MP-BGP as the control plane mechanism, when forwarding to the desired prefix we rely on two labels as indicated in the 1st image, Transport label which guides you from one PE to the next and is changed at every hop because every node creates state and builds a table that is locally significant which we call the Forwarding Information Base(FIB)

Just before traffic reaches the Egress PE-2 the Transport label is removed exposing the VPN Label which the Device will then lookup and place the packet in the correct VRF removing the VPN label and forwarding onwards as IP.

![SRv6 - SR-MPLS.png](..%2FSRv6%20-%20SR-MPLS.png)

In SR-MPLS there is an additional learning curve of Segment Routing Traffic Engineering (SR-TE) but if you are going deploy L3VPNs today I would encourage you leverage SR-MPLS as you don’t need to enable any traffic-engineering capabilities which will simplify operations and remain familiar to any engineer who MPLS experience.

In Segment routing we have the concept of Segment-ID’s (SID’s) these are identifiers that are globally significant such as the Prefix-SID which is for advertised Prefixes, Node-SID which is for PE/P nodes to stay unique within the SR Domain and adjacency-SID which is locally significant for link identifiers, these are characteristics that SR uses to identify components within a Domain 

One of the major benefits of SR-MPLS is the removal of LDP protocol which we use to create state tables on every node, this function is now integrated within the IGP such as IS-IS and OSPF which has full visibility of the whole Domain as we leverage the Link-State protocol.

Why is this all important because we are replacing the transport label for the Prefix-SID as you can see from the diagram, PE-2 Egress router still advertises prefixes leveraging MP-BGP but to help reach this prefix the IGP assists, in this case we are steering towards the Prefix-SID identifier 100, which all routers along the path know is associated with 10.10.1.1, the Prefix SID unlike the MPLS label does not change as all nodes have associated this prefix with the SID which as mentioned is globally significant because of the IGP

![SRv6.png](Images%2FSRv6%2FSRv6.png)


I’m going to oversimplify things in the operation and exclude a few bits and maybe leave that for a deeper dive in the future but SRv6 this is where things get more fun… 

In our scenario we are transporting IPv4 traffic and leveraging an IPv6 underlay. In SRv6 we have another SID which is called the Service-SID which comes in the form of an IPv6 address, this is like a label in MPLS that is advertised along with the IPv4 prefix that is the destination prefix. 

In the SRv6 Scenario PE-2 has an MP-iBGP peering with PE-1, advertisement 10.10.1.1 is passed onto PE-1 indicating that this prefix is reachable via PE-2 utilizing the Service-SID (IPv6 Address such as fc00:0:ff86::) this Prefix will be installed in PE-1 within BGP and become a routable destination under the VRF indicating that if CE-1 wants to reach the prefix of 10.10.1.1 it will be encapsulated with an IPv6 packet with the destination of the Service-SID received from PE-2.

We have achieved the same result between the variations, so why change? The goal is to always find a balance between simplification and improving network steering capability which in the past has increased complexity and planning with the likes of RSVP-TE. MPLS leverages LDP + IGP, SR-MPLS collapses these two into one so no need for LDP, and with SRv6 we are going even further using IPv6 itself

![SRv6-intro image.png](Images%2FSRv6%2FSRv6-intro%20image.png)