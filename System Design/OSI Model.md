# OSI(Open Systems Interconnection) Model

![OSI Layers](<Media/OSI Layers.png>)

Instead of thinking about networking as one giant thing, the OSI model splits it into 7 layers stacked on top of each other on the basis of the job they perform.

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

**Application Layer Protocols**

- **HTTP/HTTPS:** Exchange requests and responses on the Web. HTTPS secures HTTP communication with TLS, the successor to the older SSL (Secure Sockets Layer).
- **DNS:** Looks up information about domain names, such as the IP address for a server.
- **FTP:** Transfers files between a client and a server.

REST, GraphQL, gRPC, webhooks, WebSockets, and SSE are different API approaches and communication mechanisms used at the application layer.

## Layer 6 - Presentation Layer

This layer is responsible for Data formatting, including tasks like encryption, compression and character encoding.

For exammple: -

- TLS(Transport layer security) encryption is used for encrypting HTTP to make it HTTPS. In the OSI model, TLS is usually placed in the presentation layer, since encryption is a presentation-layer function, but in real-world networking, TLS is effectively implemented as a protocol that sits between the application and transport layers.
- UTF-8 encoding
- JPEG compression.

## Layer 5 - Session Layer

This layer is responsible for setting up, managing, and ending sessions/communication.

It is involved in creating a session identifier, tracking session state, associating incoming data with the correct session and terminating the session when the job is finished. TLS is a common real example for setting up this connection.

For Example: The server would know that this session #123 belongs to a client that was doing a particular job, like editing file One.txt, and not some other client watching a video.
Client <-------> Server
      Session #123

> Unlike the OSI model In modern `TCP/IP` model networks there isn't a separate presentation and session layer implementation. The Application just handles it itself. For instance, you could send a request to a server, and the server responds with a session ID = #48C19. For all the subsequent requests, you would just include that session ID.

### Where Layers 5–7 Actually Live

```text
            ┌ Application  (7)  HTTP, login sessions (cookies)
 IN THE APP │ Presentation (6)  TLS encryption, compression, encoding
            └ Session      (5)  TLS sessions, HTTP/2 streams, keep-alive
───────── socket API: connect() / send() / recv() ─────────  ◀── per-app firewalls/filters hook in here
            ┌ Transport    (4)  TCP / UDP
 IN THE OS  │ Network      (3)  IP
            └ Data Link    (2)  Ethernet / Wi-Fi driver
 HARDWARE     Physical     (1)  NIC, cables, radio
```

Layers 5–7 all live inside the app. In real systems there's no separate session-layer protocol. The work the OSI model calls "session," like resuming a TLS session or multiplexing HTTP/2 streams, is done by the app's own libraries. The **TCP/IP** model -- named after it's two main protocols -- just merges 7-5 into a single "application layer." TLS doesn't fit neatly: it handles both encryption (6) and session setup (5).

### What Is a Session?

A **session** is state that both ends remember across many messages, plus an ID to look that state up. Because of it, a later message can be understood in the context of earlier ones without repeating the setup work (handshake, key exchange, login).

A **connection** is different: it is one live pipe between two endpoints. A session sits on top of connections and can outlive them. A TLS session can be resumed on a brand-new TCP connection, and your login session survives closing the browser tab.

A single HTTPS page load involves several kinds of state stacked on top of each other:

| What | Who keeps the state | Identified by | Lasts until |
| --- | --- | --- | --- |
| TCP connection (L4) | OS kernel | 4-tuple: source IP, source port, destination IP, destination port | The connection closes |
| TLS session (L5/6) | App's TLS library (OpenSSL, BoringSSL, etc.) | Session ID or session ticket | The ticket expires (TLS 1.3 caps it at 7 days) |
| HTTP/2 stream (L5) | App's HTTP library | Stream ID (1, 3, 5, ...) | One request/response finishes |
| Login session (L7) | Web server + a store like Redis | Session ID cookie | Logout or expiry |

For how these sessions are set up step by step, and what the OS does as a request travels from one app to another, see [Life of a Request](<../Software Engineering/Networking/Life of a Request.md>).

## Layer 4 - Transport Layer

This layer deals with ports(to determine which application gets the data) and other depending on which protocol is in use.

There are 2 major protocols that could be used here:

1. TCP(Transmission Control Protocol) provides:
    - Reliable and ordered delivery
    - Retransmission of lost packets
    - Flow and Congestion control(don't overwhelm receiver)
    - **3 Way handshake** for establishing the connection

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
