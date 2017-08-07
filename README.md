**Routing Basics**

To be capable of routing packets a router must know at least the following information:

* Destination address.

* Neighbor routers from which it can learn about remote networks. 

* Possible routes to all remote networks.

* The best route to each remote network.

* How to maintain and verify routing information. 

**Understanding the difference between a routing protocol and a routed protocol**

* A routing protocol is a tool used by routers to dynamically find all the networks in the internetwork as well as ensure that all routers have the same routing table.

* Once all routers know about all networks, a routed protocol can be used to send packets(user data) through the established internetwork. Routed protocols are assigned to an interface and determine the method of packet delivery. Examples of routed protocol are IPv4 and IPv6.

**The IP routing process**

* The IP routing process doesn’t change, regardless of the size of the network.

     Host A                                       Router                                 Host B

![image alt text](https://i.imgur.com/Yjqax2G.png)

**Host A pings Host B, this is the steps:**

A packet is created at the host.

1. ICMP creates an echo request payload.

2. ICMP hands the payload to IP, which then creates a packet. At a minimum this packet contains an IP source address, an IP destination address, and a protocol field. The packet is forwarded.

3. After the packet is created, IP determines whether the destination IP address is on the local network or a remote one.

4. Because IP has discovered that this is a remote request, the packet needs to be sent to the default gateway so the packet can be routed to the correct remote network. The registry in windows is parsed to find the configured default gateway.

5. The default gateway of host A (172.16.20.2) is configured to 172.16.10.1. For this packet to be sent to the local gateway, the hardware address of the router interface must be known, why? So the packet can be handed down the Data Link Layer, framed, and sent to the router’s interface that is connected to the 172.16.10.0 network.

6. The Address Resolution Protocol(ARP) cache of the host is checked to see whether the IP address of the default gateway has already been resolved to a hardware address. If it is, the packet is then free to be handed down to the Data Link Layer for framing. If the MAC isn’t already in the ARP cache of the host, an ARP broadcast is sent out onto the local network to search for the MAC address of 172.16.10.1

7. After the packet and destination MAC address have been handed to the Data Link Layer, the WLAN/LAN driver is used to provide media access. A frame is then generated, encapsulation the packet with control information. Below is a typical frame, note that it doesn’t include the remote host’s MAC address. ![image alt text](https://i.imgur.com/H7iJqYz.png)

8. When the frame is completed, it’s handed down to the physical layer to be placed onto the  the physical medium one bit at the time.

**The router receives the packet:**

9. Every device in the collision domain receives these bits and builds the frame. They each run a CRC and check the answer in the FCS field. If the the answer don’t match, the frame is discarded. But if the CRC matches, then the hardware-destination is checked to see if it matches to the router’s interface. If it’s a match then the ether-type field is checked to find the protocol used at the Network Layer.

10. The packer is pulled from the frame and what is left of the frame is discarded. The packet is handed down to the protocol listed in the Ether-Type field - it’s given to IP.

**The router routes the packet: **

11. IP received the packet and checks the IP destination address. Because the packet’s destination address doesn’t match any of the addresses configured in the router’s interface, the router will lookup the destination IP network address in it’s routing table.

12. The routing table must must have a entry for the network 172.16.20.0 or the packet will be discarded immediately and an ICMP message will be sent back to the originating device with a "Destination Unreachable message".

13. If the router does find an entry for the destination network in it’s table, the packet is switched to the exit interface - in this example : interface ethernet 1.

14. The router packet-switches the packet to the ethernet 1 buffer.

15. Now that the packet is in the Ethernet 1 buffer, IP needs to know the hardware address of the destination host and first checks the ARP cache. If the hardware address of Host B has already been resolved and is in the router’s  ARP cache, then the packet and the hardware address are handed down to the Data Link Layer to be framed. But if the hardware address hasn’t already been resolved, the router then sends an ARP request out E1 looking for the hardware address of 172.16.20.2. Host B responds with it’s hardware address, the packet and hardware-destination address are both sent to the Data Link layer for framing.

16. The Data Link layer creates a frame with the destination and source hardware address - ethertype field, and FCS field at the end. The frame is handed to the physical layer to be sent out one bit at the time

**Finally, the remote host receives the packet:**

17.  Host B receives the frame and immediately runs a CRC. If the result matches what is in the FCS field , the hardware-address is then checked. If the host finds a match, the ether-type field is checked to determine the protocol that the packet should be handed to at the Network Layer - IP in this example.

18. At the Network Layer, IP receives the packet and checks the IP destination address. Because there’s finally a match made, the protocol field  is checked to find out whom the payload should be given to.

19. The payload is handed to ICMP. ICMP responds to this immediately discarding the packet and generating a new payload as an echo request.

**The destination host becomes a source host:**

20. A packet is created, including the source and destination addresses, protocol field and payload. The destination is now host A.

21. IP checks to see whether the destination IP address is a device on the local LAN or on a remote network. Like we already know the destination is on a remote network so the packets needs to be sent to the default gateway.

22. The default gateway IP is found in the registry of the windows device, and the ARP cache is checked to see whether the MAC address has already been resolved from an IP address.

23. After the MAC address of the default gateway is found, the packet and the destination MAC address are handed down to the Data Link layer for framing.

24. The Data Link layer frames the packet of information and includes the following in the header.

* The destination and source MAC address.

* The Ether-type field with 0x0800(IP) in it.

* The FCS field with the CRS result in tow.

1. The frame is handed down to the Physical Layer to be sent out over the network medium one bit at the time.

**Time for the router to route another packet:**

1. The router’s Ethernet 1 interface receives the and build a frame. The CRC is run, and the FCS field is checked to make so the answer match.

2. If CRC is found ok, the MAC destination address is checked. Because the router’s interface is a match, the packet is pulled from the frame, and the Ether-Type field is checked to see which protocol at the network layer the packet should be sent to.

3. The protocol is determined to be IP, so it’s gets the packet. IP runs a CRC check on the IP header first and then checks the destination IP address.

4. The packet is switched to interface Ethernet 0.

5. The router checks the ARP cache to determine whether the hardware address has already been resolved.

6. The hardware address is already in the cached from the originating trip to Host B, so the packet are handed to the Data Link Layer.

7. The Data Link Layer builds a frame with the destination and source hardware addresses and then puts the IP in the Ether-Type field. A CRC is run on the frame, and the result is placed in the FCS field.

8. The frame is then handed to the Physical Layer to be sent out onto the local network one bit at the time. 

**The original source host, now the destination host, receives the packet:**

1. The destination host receives the frame, runs a CRC, checks the MAC destination address, and looks in the Ether-Type field to find out whom to hand the packet to.

2. IP is the designated receiver, and after the packet is handed to IP at the Network Layer, IP checks the protocol field for further direction. IP finds instructions to give the payload to ICMP, and ICMP determines the packet to be an ICMP echo reply.

3. ICMP acknowledges that it has received the replay by sending an exclamation point (!) to the user interface. ICMP then attempts to send four more echo requests to the destination host.

                                                                                                                 

         ![image alt text](https://i.imgur.com/eJKPNiU.png)          

**What is a byte:**

     ![image alt text](https://i.imgur.com/1PteQwu.png)

![image alt text](https://i.imgur.com/Dua3xVL.png)

![image alt text](https://i.imgur.com/QEXxvK7.png)
