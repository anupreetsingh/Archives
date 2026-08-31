# OSI(Open Systems Interconnection) Model

![OSI Layers](<Media/OSI Layers.png>)

Instead of thinking about networking as one giant thing, the OSI model splits it into 7 layers stacked on top of each other.

Information in the layers travel in both directions, depending on whether a device is sending(7 to 1) or receiving data(1-7)

This helps engineers think about where a networking problem is occurring and what components are responsible.

Mnemonics to remember the name:

- All People Seem To Need Data Processing. (7-1)
- Please Do Not Throw Away Sausage Pizza. (1-7)

As data moves between different network layer, each layer adds its own header and creates a different **Protocol Data Unit(PDU)**.

| Layer | PDU Name |
| --- | --- |
| Application | Data |
| Transport (TCP/UDP) | Segment (TCP) or Datagram (UDP) |
| Network (IP) | Packet |
| Data Link (Ethernet/Wi-Fi) | Frame |
| Physical | Bits |

## Layer 7 - Application Layer

This is where applications interact with the network. It tells what network service does our application need?
For instance: your browser making an HTTP request.
Other examples of application layer services could be HTTPS, DNS, FTP, etc.

- Different ways applications expose APIs to request data: REST vs GraphQL vs gRPC

- **WebSockets and SSE(Server-Sent Events)** are different communication mechanisms that an application can choose depending on the communication pattern it needs.

  | WebSockets | SSE (Server-Sent Events) |
  | --- | --- |
  | Client ↔  Server | Server → Client |
  | Both sides can send messages anytime | Only server pushes messages |
  | Used for interactive communication | Used for live updates |
  | Examples: Chat, games, trading | Examples: Notifications, dashboards, news feeds |

## Layer 6 - Presentation Layer

This layer is responsible for Data formatting, including tasks like encryption, compression and character encoding.

For exammple: - TLS encryption, UTF-8 encoding, JPEG compression.

## Layer 5 - Session Layer

This layer is responsible for setting up, managing, and ending sessions/communication.

It is involved in creating a session identifier, tracking session state, associating incoming data with the correct session and terminating the session when the job is finished.

For Example: The server would know that this session #123 belongs to a client that was doing a particular job, like editing file One.txt, and not some other client watching a video.
Client <-------> Server
      Session #123

> Unlike the OSI model In modern `TCP/IP` model networks there isn't a separate session and presentation layer implementation. The Application just handles it itself. For instance, you could send a request to a server, and the server responds with a session ID = #48C19. For all the subsequent requests, you would just include that session ID.

## Layer 4 - Transport Layer

This layer deals with ports(to determine which application gets the data) and other depending on which protocol is in use.

There are 2 major protocols that could be used here:

1. TCP(Transmission Control Protocol) provides:
    - Reliable and ordered delivery
    - Retransmission of lost packets
    - Flow and Congestion control(don't overwhelm receiver)
    - **3 Way handeshake** for establishing the connection

2. UDP(User Datagram Protocol) provides:
    - Best effort delivery
    - Minimal overhead cause it doesn't check whether the packets arrived, they're in order or receiver is overwhelmed.

## Layer 3 - Network Layer

This layer is responsible for moving packets end-to-end between networks using protocols like IP(IPv4 or IPv6). Covers Responsibilities like:

- Logical Addressing: using IP addresses to identify source and destination.
- Routing: determining the path packets should take across networks
- Packet Forwarding: routers move packets towards their destination
- Fragmentation: Splits packets if they're too large for a network link.

**ACLs(Access Control Lists)** are a set of rules that says allow or deny specific traffic that are configured at the router. Examples: Allow TCP port 443, Deny traffic from 10.0.0.5 and allow subnet

**Firewalls** is a security device/software that inspects network traffic and decides whether to allow or block it. It uses ACL-like rules internally.

**Load Balancer** distributes traffic across multiple servers.

- Without load balancing: Server A crashes, the application is down.

```text
Users
  ↓
Server A
```

- With load balancing: Requests get spread across servers.

```text
Users
  ↓
Load Balancer
 ├── Server A
 ├── Server B
 └── Server C
```

## Layer 2 - Data Link Layer

Responsible for hop-to-hop communication between devices on the local network(LAN).

IP addresses from the network layer tells where the packets need to reach eventually but MAC(Media Access Control) address on computer network card tells which device the frames need to go to on the next hop  

## Layer 1 - Physical Layer

This layer covers the transmission of bits across physical infra of the network like cabling, connectors,etc.
So when someone says "You have problem in the physical Layer", it means fix your cabling, punch-downs, etc.  
Run Loopback tests, test/replace cables, swap adapter cards.  
