https://people.na.infn.it/~garufi/didattica/CorsoAcq/Trasp/Lezione9/tcpip_ill/

# TCP/IP Illustrated, Volume 1: The Protocols

W. Richard Stevens

## Chapter 1. Introduction

- 1.1 Introduction
- 1.2 Layering
- 1.3 TCP/IP Layering
- 1.4 Internet Addresses
- 1.5 The Domain Name System
- 1.6 Encapsulation
- 1.7 Demultiplexing
- 1.8 Client-Server Model
- 1.9 Port Numbers
- 1.10 Standardization Process
- 1.11 RFCs
- 1.12 Standard, Simple Services
- 1.13 The Internet
- 1.14 Implementations
- 1.15 Application Programming Interfaces
- 1.16 Test Network
- 1.17 Summary

## Chapter 2. Link Layer

- 2.1 Introduction
- 2.2 Ethernet and IEEE 802 Encapsulation
- 2.3 Trailer Encapsulation
- 2.4 SLIP: Serial Line IP
- 2.5 Compressed SLIP
- 2.6 PPP: Point-to-Point Protocol
- 2.7 Loopback Interface
- 2.8 MTU
- 2.9 Path MTU
- 2.10 Serial Line Throughput Calculations
- 2.11 Summary

## Chapter 3. IP: Internet Protocol

- 3.1 Introduction
- 3.2 IP Header
- 3.3 IP Routing
- 3.4 Subnet Addressing
- 3.5 Subnet Mask
- 3.6 Special Case IP Address
- 3.7 A Subnet Example
- 3.8 ifconfig Command
- 3.9 netstat Command
- 3.10 IP Futures
- 3.11 Summary

## Chapter 4. ARP: Address Resolution Protocol

- 4.1 Introduction
- 4.2 An Example
- 4.3 ARP Cache
- 4.4 ARP Packet Format
- 4.5 ARP Examples
- 4.6 Proxy ARP
- 4.7 Gratuitous ARP
- 4.8 arp Command
- 4.9 Summary

## Chapter 5. RARP: Reverse Address Resolution Protocol

- 5.1 Introduction
- 5.2 RARP Packet Format
- 5.3 RARP Examples
- 5.4 RARP Server Design
- 5.5 Summary

## Chapter 6. ICMP: Internet Control Message Protocol

- 6.1 Introduction
- 6.2 ICMP Message Types
- 6.3 ICMP Address Mask Request and Reply
- 6.4 ICMP Timestamp Request and Reply
- 6.5 ICMP Port Unreachable Error
- 6.6 4.4BSD Processing of ICMP Messages
- 6.7 Summary

## Chapter 7. Ping Program

- 7.1 Introduction
- 7.2 Ping Program
- 7.3 IP Record Route Option
- 7.4 IP Timestamp Option
- 7.5 Summary

## Chapter 8. Traceroute Program

- 8.1 Introduction
- 8.2 Traceroute Program Operation
- 8.3 LAN Output
- 8.4 WAN Output
- 8.5 IP Source Routing Option
- 8.6 Summary

## Chapter 9. IP Routing

- 9.1 Introduction
- 9.2 Routing Principles
- 9.3 ICMP Host and Network Unreachable Errors
- 9.4 To Forward or Not to Forward
- 9.5 ICMP Redirect Errors
- 9.6 ICMP Router Discovery Messages
- 9.7 Summary

## Chapter 10. Dynamic Routing Protocols

- 10.1 Introduction
- 10.2 Dynamic Routing
- 10.3 Unix Routing Daemons
- 10.4 RIP: Routing Information Protocol
- 10.5 RIP Version 2
- 10.6 OSPF: Open Shortest Path First
- 10.7 BGP: Border Gateway Protocol
- 10.8 CIDR: Classless Interdomain Routing
- 10.9 Summary

## Chapter 11. UDP: User Datagram Protocol

- 11.1 Introduction
- 11.2 UDP Header
- 11.3 UDP Checksum
- 11.4 A Simple Example
- 11.5 IP Fragmentation
- 11.6 ICMP Unreachable Error (Fragmentation Required)
- 11.7 Determining the Path MTU Using Traceroute
- 11.8 Path MTU Discovery with UDP
- 11.9 Interaction Between UDP and ARP
- 11.10 Maximum UDP Datagram Size
- 11.11 ICMP Source Quench Error
- 11.12 UDP Server Design
- 11.13 Summary

## Chapter 12. Broadcasting and Multicasting

- 12.1 Introduction
- 12.2 Broadcasting
- 12.3 Broadcasting Examples
- 12.4 Multicasting
- 12.5 Summary

## Chapter 13. IGMP: Internet Group Management Protocol

- 13.1 Introduction
- 13.2 IGMP Message
- 13.3 IGMP Protocol
- 13.4 An Example
- 13.5 Summary

## Chapter 14. DNS: The Domain Name System

- 14.1 Introduction
- 14.2 DNS Basics
- 14.3 DNS Message Format
- 14.4 A Simple Example
- 14.5 Pointer Queries
- 14.6 Resource Records
- 14.7 Caching
- 14.8 UDP or TCP
- 14.9 Another Example
- 14.10 Summary

## Chapter 15. TFTP: Trivial File Transfer Protocol

- 15.1 Introduction
- 15.2 Protocol
- 15.3 An Example
- 15.4 Security
- 15.5 Summary

## Chapter 16. BOOTP: Bootstrap Protocol

- 16.1 Introduction
- 16.2 BOOTP Packet Format
- 16.3 An Example
- 16.4 BOOTP Server Design
- 16.5 BOOTP Through a Router
- 16.6 Vendor-Specific Information
- 16.7 Summary

## Chapter 17. TCP: Transmission Control Protocol

- 17.1 Introduction
- 17.2 TCP Services
- 17.3 TCP Header
- 17.4 Summary

## Chapter 18. TCP Connection Establishment and Termination

- 18.1 Introduction
- 18.2 Connection Establishment and Termination
- 18.3 Timeout of Connection Establishment
- 18.4 Maximum Segment Size
- 18.5 TCP Half-Close
- 18.6 TCP State Transition Diagram
- 18.7 Reset Segments
- 18.8 Simultaneous Open
- 18.9 Simultaneous Close
- 18.10 TCP Options
- 18.11 TCP Server Design
- 18.12 Summary

## Chapter 19. TCP Interactive Data Flow

- 19.1 Introduction
- 19.2 Interactive Input
- 19.3 Delayed Acknowledgements
- 19.4 Nagle Algorithm
- 19.5 Window Size Advertisements
- 19.6 Summary

## Chapter 20. TCP Bulk Data Flow

- 20.1 Introduction
- 20.2 Normal Data Flow
- 20.3 Sliding Windows
- 20.4 Window Size
- 20.5 PUSH Flag
- 20.6 Slow Start
- 20.7 Bulk Data Throughput
- 20.8 Urgent Mode
- 20.9 Summary

## Chapter 21. TCP Timeout and Retransmission

- 21.1 Introduction
- 21.2 Simple Timeout and Retransmission Example
- 21.3 Round-Trip Time Measurement
- 21.4 An RTT Example
- 21.5 Congestion Example
- 21.6 Congestion Avoidance Algorithm
- 21.7 Fast Retransmit and Fast Recovery Algorithm
- 21.8 Congestion Example (Continued)
- 21.9 Per-Route Metrics
- 21.10 ICMP Errors
- 21.11 Repacketization
- 21.12 Summary

## Chapter 22. TCP Persist Timer

- 22.1 Introduction
- 22.2 An Example
- 22.3 Silly Window Syndrome
- 22.4 Summary

## Chapter 23. TCP Keepalive Timer

- 23.1 Introduction
- 23.2 Description
- 23.3 Keepalive Examples
- 23.4 Summary

## Chapter 24. TCP Futures and Performance

- 24.1 Introduction
- 24.2 Path MTU Discovery
- 24.3 Long Fat Pipes
- 24.4 Window Scale Option
- 24.5 Timestamp Option
- 24.6 PAWS: Protection Against Wrapped Sequence Numbers
- 24.7 T/TCP: A TCP Extension for Transactions
- 24.8 TCP Performance
- 24.9 Summary

## Chapter 25. SNMP: Simple Network Management Protocol

- 25.1 Introduction
- 25.2 Protocol
- 25.3 Structure of Management Information
- 25.4 Object Identifiers
- 25.5 Introduction to the Management Information Base
- 25.6 Instance Identification
- 25.7 Simple Examples
- 25.8 Management Information Base (Continued)
- 25.9 Additional Examples
- 25.10 Traps
- 25.11 ASN.1 and BER
- 25.12 SNMP Version 2
- 25.13 Summary

## Chapter 26. Telnet and Rlogin: Remote Login

- 26.1 Introduction
- 26.2 Rlogin Protocol
- 26.3 Rlogin Examples
- 26.4 Telnet Protocol
- 26.5 Telnet Examples
- 26.6 Summary

## Chapter 27. FTP: File Transfer Protocol

- 27.1 Introduction
- 27.2 FTP Protocol
- 27.3 FTP Examples
- 27.4 Summary

## Chapter 28. SMTP: Simple Mail Transfer Protocol

- 28.1 Introduction
- 28.2 SMTP Protocol
- 28.3 SMTP Examples
- 28.4 SMTP Futures
- 28.5 Summary

## Chapter 29. NFS: Network File System

- 29.1 Introduction
- 29.2 Sun Remote Procedure Call
- 29.3 XDR: External Data Representation
- 29.4 Port Mapper
- 29.5 NFS Protocol
- 29.6 NFS Examples
- 29.7 NFS Version 3
- 29.8 Summary

## Chapter 30. Other TCP/IP Applications

- 30.1 Introduction
- 30.2 Finger Protocol
- 30.3 Whois Protocol
- 30.4 Archie, WAIS, Gopher, Veronica and WWW
- 30.5 X Window System
- 30.6 Summary

## Appendix A. The tcpdump Program

- A.1 BSD Packet Filter
- A.2 SunOS Network Interface Tap
- A.3 SVR4 Data Link Provider Interface
- A.4 tcpdump Output
- A.5 Security Considerations
- A.6 Socket Debug Option

## Appendix B. Computer Clocks

## Appendix C. The sock Program

## Appendix D. Solutions to Selected Exercises

## Appendix E. Configurable Options

- E.1 BSD/386 Version 1.0
- E.2 SunOS 4.1.3
- E.3 System V Release 4
- E.4 Solaris 2.2
- E.5 AIX 3.2.2
- E.6 4.4BSD

## Appendix F. Source Code Availability
