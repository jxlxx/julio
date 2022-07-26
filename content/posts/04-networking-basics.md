---
title: Networking Basics
date: "2022-06-01"
description: Some basic hardware involved in networking, and some other stuff too.
draft: false	
tags: ["networking"]
---

# Hardware Basics

A **router** is a network device (that may be composed of many virtual network devices) that connects multiple devices, phones, computers, etc, to form a local area network (LAN). Most routers allow device to connect wirelessly, or through ethernet.

An **access point** (AP) allows devices to connect wirelessly to a **LAN** (Local Area Network) and can also be used to extend the range of coverage to connect to a LAN (you can be physically farther away). The AP then also has a high-speed ethernet cable runs to a router.

A router acts as a hub for a LAN, and an AP is a sub-device in the LAN. Wireless routers can act as APs, but not all APs can act as routers.

A **modem** is a device that connects your LAN to a WAN (Wide Area Network). It is the thing that takes the digital signals that devices send to routers (binary data) and transforms it via modulation into analog data (electricity) that gets transmitted to the WAN. It also receives analog data and demodulates it, so the router can send it to the devices that asked for it. 

It is not uncommon to see router/modem combos.

A **switch** is a networking device that connects devices together. Usually physically, via ethernet cables. The word switch usually means a device that forwards data between devices at the layer 2 level using MAC addresses. There is such a thing as layer 3 switches, which incorporate some routing functionality. Routers usually have a kind of switch built-in to them. At this point discussing the difference between L2 switches, L3 switches, and routers etc, is beyond the basics, so I will stop.

# Layers (OSI Model)

Layered Network Model (ISO/OSI)


| # | Name | things |
| --- | --- | --- |
| 7 | Application | What the user interacts with, http, ftp |
| 6 | Presentation | encrypt, decrypt |
| 5 | Session | channels, called sessions, between devices. Can be used to sync audio and video |
| 4 | Transport | TCP, UDP |
| 3 | Network | Deciding what physical path the packets will take |
| 2 | Data Link | define the format of the data/packets |
| 1 | Physical | the physical bit, electricity on a wire kind of shit |

# IPv4 and IPv6

IPv4 addresses are made of 32 bits which usually organized into 4 bytes (also sometimes, thought rarely, called octets).

IPv6 addresses are made of 128 bits and usually written in hex.

If you have lots of zeros, you may omit them.

Pairs of equivalent IPv6 addresses:

```
2001:0db8:c9d2:0012:0000:0000:0000:0051
2001:db8:c9d2:12::51
```

```
2001:0db8:ab00:0000:0000:0000:0000:0000
2001:db8:ab00::
```

```
0000:0000:0000:0000:0000:0000:0000:0001
::1
```

The address `::1` is the *loopback address*. It always means “this machine I’m running on now”. 

In IPv4, the loopback address is `127.0.0.1`.

There’s an IPv4-compatibility mode for IPv6 addresses. To represent the IPv4 address `192.0.2.33` as an IPv6 address you write it like this: `::ffff:192.0.2.33`

## CIDR Notation

TODO

# Packets, Etc.

Data is bytes.

To share data through a network, there needs to be a common protocol that everyone agrees on, and data bytes must be in a specific format.

The most used data units in networks are the **packet**, **frame**, and **datagram**.

The word packet is often used as a catchall term for all the previously mention data units as well as others, but when you say packet and you want to refer to a specify type of packet called a packet, you probably mean an internet packet (IP).

Internet Packets are used for connection-oriented protocols.

Data is divided into packets, sent over the internet, then recombined to recreate the original data at its destination.

Packets are data units with the network layer (L3) of OSI.

There is such a thing as an MTU, a Maximum Transmitted Unit.

If packets are bigger than the MTU of a network, then they are broken up into smaller pieces of data called fragments.

The network layer (L3) is responsible for fragmentation.

Packets have `DF` flag, “Don’t Fragment.” If the `DF` flag is set to 1, and the size of the packet is bigger than the MTU of the packet, the packet is discarded. 

Next there are frames.

The main difference between a packet and a frame is its association with the OSI layers. While a packet is used in the network layer, a frame is a unit of data in the data link layer.

A frame contains more information about the transmitted message than a packet. (Like some flags and maybe some source and dest MAC addresses? need to look up)

Datagrams are used for connectionless protocols like UDP.

# Sockets

Sockets are the endpoints of communication.

Sockets allow programs to communicate to each other by using Unix file descriptors, sometimes over a network. 

A **file descriptor** is just an integer associated with an open file. 

That file can be anything: a network connection, a pipe a terminal, a legit file, whatever. This comes from the Unix philosophy that everything in Unix is a file. When Unix systems do any kind of I/O the do so by reading or writing to a file descriptor.

Socket is a big word. Like packets, its a bit of a catch-all. However in networking we are usually we are talking about Internet Sockets. 

Internet sockets are also called DARPA Internet addresses.

There are also many types of internet sockets, but we are going to talk about two kinds: **Stream Sockets** and **Datagram Sockets**. (Another interesting, and very powerful type of internet socket is a **Raw Socket**).

Stream sockets are reliable two-way connected internet streams. Things get output the same order they were input. Stream sockets use a protocol called Transmission Control Protocol, TCP. TCP makes ensures data arrive sequentially and error-free.

TCP is often heard when saying TCP/IP. IP stands for Internet Protocol and primarily deals with internet routing and is not concerned with data integrity.

Datagram sockets don’t care about order. They don’t care if they arrive where they are supposed to or not. If it does arrive though, the data in the packet will be error-free. Datagrams also use IP for routing, but they don’t use TCP, they use UDP. 

A socket address is usually a combination of a file descriptor, an IP address and a port number.

# How to get to `https://julio.lol`

TODO

# WiFi networks

TODO

# Good Software/Sites for Experimentation

- AWS free tier
- Namecheap
- Linode
- wireshark
- traceroute
- dig

# Other Stuff

[Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)

[jvns Networking Zine](https://jvns.ca/networking-zine-coloured.pdf)

[High Performance Browser Networking (O'Reilly)](https://hpbn.co/)

