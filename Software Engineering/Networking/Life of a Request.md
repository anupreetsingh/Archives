# Life of a Request

This note follows one request from the app on one machine to the app on another machine and back, through each operating system along the way. The running example is a browser loading `https://app.example.com/profile`.

[OSI Model](<../../System Design/OSI Model.md>) explains what each layer is responsible for. This note follows a request as it moves through those layers.

```text
        CLIENT MACHINE                                   SERVER MACHINE
 ┌──────────────────────────┐                     ┌──────────────────────────┐
 │ App: fetch("/profile")   │                     │ App: handler, middleware │
 │ HTTP + TLS libraries     │                     │ HTTP + TLS libraries     │
 ├──── socket API ──────────┤                     ├──── socket API ──────────┤
 │ Kernel: TCP → IP → frame │                     │ Kernel: frame → IP → TCP │
 │ NIC                      │ ──── network ─────► │ NIC                      │
 └──────────────────────────┘                     └──────────────────────────┘
```

## 1. The Client App

1. **App code asks for a URL.** It calls something high-level, such as `fetch("/profile")` in a browser or `requests.get(...)` in Python. The app never deals with sockets or packets directly.
2. **The HTTP library prepares the request.** It builds the method, path, and headers, including any `Cookie` or `Authorization` credential. See [How the Client Creates a Request](<HTTP.md#how-the-client-creates-a-request>).
3. **DNS resolves the hostname.** `app.example.com` is translated into an IP address, because the network routes by IP, not by name.
4. **The library checks for a reusable connection.** If one to the same server is already open (keep-alive or HTTP/2), it skips to [step 4](#4-sending-the-request). Otherwise it must ask the OS to open one.

## 2. Into the OS: Opening a Connection

Apps cannot use the network card or build TCP packets themselves. They ask the kernel through **system calls**, functions that cross from the app into the OS (see [Kernel](<../Computer Architecture.md#kernel>)). The set of system calls for networking is the **socket API**.

1. **`socket()`** asks the kernel for a new, unconnected endpoint. The kernel returns a **file descriptor**, a small integer such as `3` that the app uses to refer to this socket in later calls.
2. **`connect(fd, server_ip, 443)`** asks the kernel to connect that socket to the server. The kernel:
   - picks a free **source port**, such as `54012`
   - sends a **SYN**, receives a **SYN-ACK**, and replies with an **ACK**: the TCP [3 way handshake](<../../System Design/OSI Model.md#layer-4---transport-layer>)
3. **`connect()` returns** once the handshake completes. The app was blocked, waiting, until then. The connection is now identified by its **4-tuple**: source IP, source port, destination IP, destination port.

This is the only setup step the OS performs. Everything after it is the app writing and reading bytes on the file descriptor with `send()` and `recv()`.

On the server, the matching calls run in advance:

1. **`socket()` → `bind(443)` → `listen()`** reserve port 443 and tell the kernel to accept connections on it.
2. **The kernel completes handshakes on its own**, without involving the app, and queues finished connections.
3. **`accept()`** takes one queued connection and returns a **new file descriptor** for that specific client. The listening socket stays open for others.

Without libraries, the client side looks like this:

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # Get a TCP endpoint.
s.connect(("93.184.215.14", 80))                        # Kernel runs the handshake.
s.send(b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")  # Hand bytes to the kernel.
print(s.recv(4096))                                     # Read the response bytes.
s.close()                                               # Kernel sends FIN.
```

Port 80 is plain HTTP, so no TLS is involved. On port 443, a TLS library wraps the socket and runs its handshake through `send()` and `recv()` before any HTTP is sent.

## 3. Securing the Connection: TLS

1. **The app's TLS library runs the TLS handshake over that connection.** It just calls `send()`/`recv()` on the socket, so to the kernel these are ordinary bytes. Client and server agree on a cipher, the server proves its identity with a certificate, and both derive shared encryption keys. Those keys are the TLS session, and they live in the app's memory. An extension in this handshake, **ALPN**, also decides whether to speak HTTP/1.1 or HTTP/2.
2. **The server hands out a session ticket.** It's an encrypted blob of the session state that the client stores so it can resume later.

From here on, everything the app sends is encrypted by its TLS library before it reaches `send()`.

## 4. Sending the Request

1. **The HTTP library writes the request.** TLS encrypts it into records, and the app hands those bytes to the kernel with `send()`. The kernel copies them into the socket's send buffer, and `send()` returns. The app does not wait for the data to reach the server.
2. **The kernel wraps the bytes on the way down**, each layer adding its own header:

```text
[ encrypted HTTP bytes ]                                       app data
[ TCP header: ports, sequence number | data ]                  segment  (L4)
[ IP header: source and destination IP | segment ]             packet   (L3)
[ Ethernet header: next-hop MAC | packet | checksum ]          frame    (L2)
```

3. **The network card driver hands the frame to the NIC**, which transmits it as bits over the cable or radio (L1).

**Connection reuse**: the HTTP library reuses the connection for many requests. With HTTP/2, each request gets its own stream ID on the same connection, so many requests share one TCP + TLS setup in parallel. With HTTP/1.1, **keep-alive** reuses the connection for requests one after another instead of closing it after each one.

## 5. Across the Network

1. **The frame reaches the next hop**, usually a home router or switch, using the destination MAC address.
2. **Each router reads the IP header** and forwards the packet toward the destination IP. It wraps the packet in a **new frame** addressed to the next hop's MAC address.
3. **This repeats** until the packet reaches the server's network, often a load balancer first.

The IP addresses stay the same end to end, while the MAC addresses change at every hop. See [Layer 3](<../../System Design/OSI Model.md#layer-3---network-layer>) and [Layer 2](<../../System Design/OSI Model.md#layer-2---data-link-layer>).

TCP also works here, from both ends: the receiver acknowledges segments it gets, and the sender's kernel retransmits any that go unacknowledged and reorders arrivals by sequence number. The apps never see any of this.

## 6. Up the Server Stack

1. **The server's NIC receives the frame** and passes it to the kernel.
2. **The kernel unwraps each layer**: it checks the frame, reads the IP header to confirm the packet is for this machine, then reads the TCP header.
3. **The 4-tuple identifies the socket.** The kernel uses it to find the connection, and therefore the process that owns it, and places the bytes in that socket's receive buffer.
4. **The server app's `recv()` returns those bytes.** They are still TLS-encrypted.

## 7. The Server App

1. **The TLS library decrypts** the records using the session keys from [step 3](#3-securing-the-connection-tls).
2. **The HTTP library parses** the method, path, headers, and body into a request object.
3. **Middleware runs**, including authentication. The app-level login session rides inside the request: the `Cookie` or `Authorization` header is read and checked against Redis or by verifying a JWT. See [Authentication](<Authentication.md#session>).
4. **The handler runs** and builds a response, for example the user's profile as JSON.

## 8. The Response

The response follows the same path in reverse:

1. **The server app** serializes the response; TLS encrypts it; `send()` hands it to the server's kernel.
2. **The server kernel** wraps it into segments, packets, and frames, and sends it out.
3. **The network** routes it back to the client's IP.
4. **The client kernel** unwraps it and matches the 4-tuple to the browser's socket.
5. **The client app**'s `recv()` returns the bytes; TLS decrypts them, the HTTP library parses the response, and `fetch()` resolves with it.

## 9. Teardown and Reuse

**Teardown runs from the top down.** HTTP/2 sends `GOAWAY`, TLS sends `close_notify`, then the app calls `close()` and the kernel sends TCP `FIN`. The TLS session ticket and the login cookie are kept, so both outlive the connection.

**Resumption**: on the next visit, the OS does a fresh TCP handshake (new connection, new source port). The TLS library presents the saved ticket instead of doing a full handshake, which skips the certificate exchange and makes setup cheaper and faster. The session outliving the connection is exactly the idea the OSI session layer describes.

## What Role the OS Plays

- **It owns layers 2–4.** That covers the TCP state machine (handshake, sequence numbers, retransmissions, flow control), port assignment, IP routing, and the network card driver.
- **The socket API is the boundary.** Above it, the app hands the kernel a stream of bytes. Below it, the kernel turns those bytes into segments, packets and frames.
- **The kernel doesn't understand the bytes.** After the TLS handshake they're encrypted records, so the OS can't tell a TLS session, an HTTP/2 stream or a login cookie apart from random data. For each connection it only knows the 4-tuple, the protocol (TCP/UDP) and which process owns the socket.
- **OS-level filters see the same limited view.** Per-app firewalls and content filters (the kind that can block one app from port 443) hook in where the socket API hands off to TCP/UDP, at the top of layer 4. They see transport-level information (IP, port, protocol) plus the owning process, but nothing session-level, like which TLS session or HTTP request this is. Seeing that would mean decrypting the traffic, which takes a **TLS-intercepting proxy** that pretends to be the server to the app and acts as the client to the real server.

Two exceptions blur the line:

- **kTLS (Linux kernel TLS)**: the app still does the handshake, then hands the keys to the kernel so it can encrypt and decrypt records itself. The session is still set up by the app.
- **QUIC / HTTP/3**: runs over UDP and implements reliability, streams and TLS 1.3 inside the app. Even the "transport" work moves above the socket, and the OS only sees UDP datagrams.
