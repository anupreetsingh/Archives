## Casts

Every packet on a network has a destination address, and that address decides how many devices receive it. Networking calls this the packet's **cast**. You need the four kinds to see why mDNS behaves differently from ordinary DNS.

Every packet is addressed twice. The destination IP sits inside the packet, and the destination MAC sits on the frame wrapped around it. The cast decides both, and each device along the way reads only the address it understands.

### The four casts

- With **unicast**, one sender talks to exactly one receiver. Most internet traffic works this way, and so does normal DNS: when your Mac looks up `example.com`, it sends the query to one DNS server, such as 1.1.1.1 on port 53.
- With **broadcast**, one sender talks to every device on the local network. Every device must pick the packet up and check it, even when it has nothing to do with that device, so broadcast is used only when the sender has no other option. DHCP is the classic case: a device that has just joined has no IP address and doesn't know where the DHCP server is, so it sends to everyone.
- With **multicast**, the sender addresses a group, and only devices that have joined that group receive the packet. See below.
- With **anycast**, many servers around the world share one IP address, and internet routing delivers your packet to the nearest one. That's why 1.1.1.1 and 8.8.8.8 answer quickly wherever you are. On your own LAN, anycast looks exactly like unicast. The difference only shows up out on the internet.

### Groups and multicast

A **multicast group** is an IP address that belongs to no single device. It works like a radio channel: the sender transmits on it without knowing who's listening, and only devices that have tuned in hear it. IPv4 reserves `224.0.0.0` – `239.255.255.255` for groups. Some well-known ones:

- `224.0.0.251` (`ff02::fb` on IPv6) is mDNS, on port 5353.
- `239.255.255.250` is SSDP, which smart TVs and other UPnP devices use to announce themselves.
- `224.0.0.1` is "all hosts". Every device that supports multicast is in it automatically.

A device joins a group like this:

1. An app asks its kernel to join the group. On macOS, `mDNSResponder` joins `224.0.0.251` at boot.
2. The kernel tells the network card to start accepting frames sent to that group's MAC address.
3. The kernel announces the join on the LAN with an IGMP (Internet Group Management Protocol) message. A switch that does IGMP snooping notes which port wants the group. Most home switches ignore the message.

No server keeps a list of a group's members. Only the members themselves, and possibly the switch, know who is in it.

Multicast exists for two reasons:

- Discovery when you don't know who to ask. mDNS, AirPlay and printer discovery need to reach "whoever has this" without disturbing every device the way broadcast does.
- Sending the same data to many receivers at once without a separate copy for each, as IPTV does on some ISP networks.

Broadcast is effectively a group that every device is in and can't leave. Multicast is the version that devices choose to join.

### How a packet is delivered

Sender's app → sender's kernel → network card → access point / switch → receivers' network cards → receivers' kernels → receiving app

1. The app chooses the destination IP, and with it the cast.
2. The sender's kernel picks the destination MAC based on that IP:
   - For unicast, it uses ARP to find the MAC of the target, or of the default gateway if the target is outside the LAN.
   - For broadcast, it uses the fixed address `ff:ff:ff:ff:ff:ff` (all ones), with no lookup.
   - For multicast, it calculates the MAC from the IP, with no lookup: `01:00:5e` followed by the low 23 bits of the group address. So `224.0.0.251` becomes `01:00:5e:00:00:fb`.
3. The network card transmits the frame without reading it.
4. The access point and switch read only the destination MAC:
   - A known unicast MAC goes out the one port where that device is.
   - A broadcast MAC is flooded out every port. The access point sends it once over the air.
   - A multicast MAC is flooded like broadcast by a basic switch. A snooping switch sends it only to the ports that joined, though `224.0.0.x` groups are usually flooded anyway.
5. Each receiving network card checks the destination MAC against its accept list: its own MAC, the broadcast MAC, and the multicast MACs its OS subscribed to. It drops everything else in hardware, and the OS never sees it.
6. The receiving kernel checks the destination IP and port, then hands the data to the app listening on that port. If nothing is listening there, the kernel drops it.

So `ff:ff:ff:ff:ff:ff` isn't an instruction the access point chooses to follow. The sender's kernel writes it because the IP is a broadcast IP, and every device on the link treats all ones as meaning "for everyone".

### Side by side

| Cast | Dest IP (example) | Dest MAC | Switch / access point | Network cards that accept it | Leaves the LAN? |
|---|---|---|---|---|---|
| Unicast, same LAN | 192.168.1.50 | That device's MAC (ARP) | Sends to one port | One | No |
| Unicast, internet | 93.184.216.34 | Gateway's MAC (ARP) | Sends to the router's port | The router's | Yes, routed onward |
| Broadcast | 255.255.255.255 | `ff:ff:ff:ff:ff:ff` | Floods every port | All | No, routers never forward it |
| Multicast | 224.0.0.251 | `01:00:5e:00:00:fb` | Floods, or sends only to joined ports | Only subscribed ones | No for `224.0.0.x` |
| Anycast | 1.1.1.1 | Gateway's MAC (ARP) | Same as internet unicast | The router's | Yes, to the nearest server |

### Fine print

- There are two kinds of broadcast IP. `255.255.255.255` is the limited broadcast, meaning "everyone on the link I'm on". DHCP uses it because the device doesn't know its subnet yet. `192.168.1.255` is the directed broadcast, meaning "everyone in 192.168.1.0/24". On the LAN, both use the frame address `ff:ff:ff:ff:ff:ff`.
- The multicast MAC filter is rough. The mapping keeps only 23 bits of the IP, so 32 groups share each MAC. A card subscribed to one of them accepts frames for all 32, and the kernel drops the unwanted ones by checking the IP.
- The difference between broadcast and multicast is where the filtering happens. With broadcast, every device's kernel has to check the packet. With multicast, devices outside the group drop it in their network card.

### Why this matters for mDNS

The "m" in mDNS stands for multicast. Instead of asking one server, the Mac asks the `224.0.0.251` group "who is MacBook.local?". Every member hears the question, and only the device that owns that name answers. It usually sends the answer to the group too, so every listener can cache it. Routers don't forward `224.0.0.x` groups, so the whole exchange stays on your local network.

These are the general steps a Wi-Fi device (a laptop) takes to join a network and get online.

## Joining Wi-Fi

Joining Wi-Fi makes a device a member of a wireless LAN at Layers 1–2. When it's done, the device can send and receive frames by MAC address, but it won't have an IP address yet. Getting IP is DHCP's job.

Pick a MAC address → find networks → choose an access point → authenticate and associate → 4-way handshake → port opens → link up

### Access points, SSIDs and BSSIDs

- An **access point** is the Wi-Fi radio, usually inside the home router. It runs on one channel in one band (2.4, 5 or 6 GHz).
- The **SSID** (Service Set Identifier) is the network's name, such as "Home-WiFi".
- The **BSSID** (Basic Service Set Identifier) is the MAC address of one radio serving that name.

One SSID often has several BSSIDs: one per band, and more again with a mesh system. The device joins one BSSID, usually the one with the strongest signal, and can later **roam** to another BSSID on the same SSID as you move around.

### Which MAC address the device uses

Before it sends anything, macOS chooses the MAC address it will use on this network. This is the **Private Wi-Fi Address** setting:

- Off uses the hardware MAC, the "burned-in" one set at the factory and stored in the Wi-Fi chip.
- Fixed uses a random MAC that stays the same for this network. Each network sees a different address, so networks can't link the device across locations.
- Rotating uses a random MAC that changes from time to time.

The address the device sends is just a value the OS writes into each outgoing frame, so the OS is free to choose it. A random MAC has the "locally administered" bit set, so its second hex digit is 2, 6, A or E.

Everything later depends on this choice. The access point records the address, and the DHCP server hands out an IP based on it. If the address changes, the router sees a new device.

### Finding networks: beacons and probes

The device doesn't know which channel a network uses, so it hops through the channels and listens on each. That's why the Wi-Fi menu takes a moment to fill in.

- A **beacon** is an announcement the access point **broadcasts** to `ff:ff:ff:ff:ff:ff` about 10 times a second. It carries the SSID, the BSSID, the channel, the supported speeds and the security type (WPA2/WPA3). This is passive scanning: the device only listens.
- A **probe request** is the device **broadcasting** "which networks are here?". Each access point replies with a **probe response** addressed only to the device (unicast). This is active scanning.
- A **hidden network** leaves its name out of its beacons, so a device can only find it by probing for that name.

Beacons are broadcast for the same reason a DHCP Discover is. The sender doesn't know who's listening, so it tells everyone.

### Authentication and association

The beacon told the device the BSSID, so from here on it addresses that radio directly by unicast.

- **Authentication.** With WPA2 this step is a formality, and the password is checked later in the handshake. With WPA3, the password is proven here, through an exchange called **SAE** (Simultaneous Authentication of Equals). SAE stops anyone who records the exchange from guessing the password offline.
- **Association.** The device asks to join, and the access point accepts and assigns it an association ID.

### The 4-way handshake and the keys

Both sides already hold a master key derived from the password. They exchange random numbers (nonces) over four messages and each calculates the same keys, so the password itself is never sent. There are two keys, one for each kind of cast:

| Key | Who has it | Encrypts |
|---|---|---|
| **PTK** (pairwise transient key) | This device and the access point only | Unicast traffic between them |
| **GTK** (group temporal key) | Every device on the network | Broadcast and multicast traffic, which everyone must be able to read |

The access point sends the GTK to the device inside the handshake, encrypted. When a device leaves, many access points replace the GTK so that device can't keep reading the network's broadcasts.

WPA2/WPA3 (Wi-Fi Protected Access) is the security protocol. "Wi-Fi Protected Access" is also the Wi-Fi Alliance's certification name for it.

### The port and the station table

The access point treats each device as a **port**. The port starts closed: the access point accepts only handshake frames (EAPOL) from the device and drops everything else. When the handshake succeeds:

- The access point adds the device to its **station table**: its MAC address, association ID and keys.
- The port opens, and the access point starts working as a **bridge**. It decrypts frames arriving over the air and passes them to the wired side (the switch, then the router) as ordinary Ethernet frames. Frames heading the other way, it encrypts and sends over the air.

On Wi-Fi, every frame goes through the access point. Even two laptops on the same network don't talk directly: each frame goes laptop → access point → laptop.

### What macOS does

- `airportd`, the Wi-Fi daemon, runs the scanning, joining and handshake.
- When the handshake finishes, the interface `en0` goes **link up**. `configd` notices, updates its dynamic store (the one `scutil` reads), and starts DHCP.
- macOS saves the network to its known networks and stores the password in the Keychain, so it rejoins automatically next time.

### Frames side by side

| Frame | Sender → receiver | Destination MAC | Cast | Acknowledged? | Encrypted? |
|---|---|---|---|---|---|
| Beacon | Access point → everyone | `ff:ff:ff:ff:ff:ff` | Broadcast | No | No |
| Probe request | Device → everyone | `ff:ff:ff:ff:ff:ff` | Broadcast | No | No |
| Probe response | Access point → device | Device's MAC | Unicast | Yes | No |
| Authentication / association | Device ↔ access point | Each other's MAC | Unicast | Yes | No |
| 4-way handshake | Device ↔ access point | Each other's MAC | Unicast | Yes | Keys are created here |
| Normal traffic | Device ↔ access point | Each other's MAC | Unicast | Yes | Yes, with the PTK |
| Broadcast / multicast | Access point → everyone | `ff:ff…` or `01:00:5e…` | Broadcast / multicast | No | Yes, with the GTK |

### Fine print

- Unicast Wi-Fi frames are acknowledged: the receiver sends a short ACK, and a frame with no ACK is resent. Broadcast and multicast frames get no ACK and are never resent. That's why they go out at a slow, reliable speed, and why a device simply retries if it misses one.
- Once joined, Layers 1–2 are done. The device can reach its MAC-address neighbours, but it can't use the internet until DHCP gives it an IP address.

## DHCP

**DHCP** (Dynamic Host Configuration Protocol) is how a device gets its network settings automatically when it joins a network, so nobody has to type them in. "Dynamic" is the key word. Before DHCP, an administrator had to visit each machine and enter its settings by hand.

DHCP starts the moment Wi-Fi reports link up. The device can send frames by MAC address, but it has no IP address, no subnet, no router and no DNS server.

Discover → Offer → Request → Acknowledge (**DORA**) → check the address is free → save the settings → renew before the lease runs out

### Who's involved

- The **DHCP client** runs on the device. On macOS it lives inside `configd` (its IPConfiguration component), the same daemon that keeps the dynamic store `scutil` reads.
- The **DHCP server** usually runs on the home router. It keeps:
  - a **pool** of addresses it can hand out, such as `192.168.1.2` – `192.168.1.254`
  - a **lease table** recording which MAC address has which IP, and until when
  - optional **reservations**, which always give a particular MAC address the same IP

DHCP runs over UDP. The server listens on **port 67** and the client on **port 68**. Fixed ports on both sides let each find the other before either has an address.

### The exchange (DORA)

1. **Discover.** The device has no IP and doesn't know where the server is, so it broadcasts: from `0.0.0.0:68` ("I have no address yet") to `255.255.255.255:67`, in a frame addressed to `ff:ff:ff:ff:ff:ff`. Inside the packet it puts:
   - its MAC address, so the server knows who's asking
   - a random **transaction ID**, so it can match replies to this request
   - a list of the settings it wants (subnet mask, router, DNS servers…)
   - the IP it had last time, if it remembers one
2. **Offer.** The server picks a free address from its pool and offers it, for example `192.168.1.23`, with the settings attached. The Offer usually comes back as **unicast** to the device's MAC address, even though the device doesn't have that IP yet. Its network card accepts the frame because the MAC is its own, and the client recognises the reply by the transaction ID. If the client asked for a broadcast reply (a flag in the Discover), the Offer is broadcast instead.
3. **Request.** The device broadcasts "I'm taking `192.168.1.23` from server `192.168.1.1`". It broadcasts rather than replying only to that server so that any other DHCP server that made an offer knows it wasn't picked and can put its address back in its pool.
4. **Acknowledge (ACK).** The server confirms, records the lease in its table, and sends the final settings. Only now is the address the device's. If the address is no longer available, the server sends a **NAK** instead, and the device starts again from Discover.

### What the device gets

The ACK carries the device's configuration. Each item is a numbered DHCP "option":

| Setting | Example | What it's for |
|---|---|---|
| IP address | `192.168.1.23` | The device's own address on this LAN |
| Subnet mask (option 1) | `255.255.255.0` (`/24`) | Deciding "is this destination local or not?" |
| Router (option 3) | `192.168.1.1` | The default gateway for anything outside the subnet |
| DNS servers (option 6) | `192.168.1.1`, or the ISP's servers | Where to send name lookups |
| Lease time (option 51) | 86400 seconds (1 day) | How long the device may keep the address |

The DNS server is often the router itself, which forwards the lookups to the ISP's servers. Other routers hand out the ISP's DNS addresses directly, which is where addresses like Comcast's come from.

### Checking the address and saving the settings

1. Before using the address, the device sends a few ARP probes asking "does anyone have `192.168.1.23`?". If another device answers, the address is already in use, so the device sends a **Decline** and starts over.
2. If nobody answers, the device configures `en0` with the address, and `configd` writes the IP, mask, router and DNS servers into its dynamic store. Every app and system service now sees the new network state.
3. macOS also saves the lease for this network, so next time it can ask for the same address straight away.

### The lease

An address is only on loan. The device keeps it for the lease period and renews it before the period ends:

| When | What the device does | Cast |
|---|---|---|
| Halfway through the lease (T1) | **Renew**: sends a Request straight to the server that gave it the address | Unicast |
| 87.5% through (T2), if renewing failed | **Rebind**: broadcasts the Request so any DHCP server can answer | Broadcast |
| Lease expires | Stops using the address and starts again from Discover | Broadcast |

Renewing skips Discover and Offer because the device already has an address and knows the server.

Rejoining a network works the same way. The device remembers the address it had there, so it skips Discover and broadcasts a Request for that address. The server either ACKs it, which is the fast path when you get home, or NAKs it, and the device goes back to Discover.

### Messages side by side

| Message | Source IP:port | Destination IP:port | Destination MAC | Cast |
|---|---|---|---|---|
| Discover | `0.0.0.0:68` | `255.255.255.255:67` | `ff:ff:ff:ff:ff:ff` | Broadcast |
| Offer | `192.168.1.1:67` | `192.168.1.23:68` | Device's MAC | Usually unicast |
| Request | `0.0.0.0:68` | `255.255.255.255:67` | `ff:ff:ff:ff:ff:ff` | Broadcast |
| ACK | `192.168.1.1:67` | `192.168.1.23:68` | Device's MAC | Usually unicast |
| Renew (T1) | `192.168.1.23:68` | `192.168.1.1:67` | Router's MAC | Unicast |
| Rebind (T2) | `192.168.1.23:68` | `255.255.255.255:67` | `ff:ff:ff:ff:ff:ff` | Broadcast |

### On macOS

- `ipconfig getifaddr en0` shows the IP address the device got.
- `ipconfig getpacket en0` shows the full ACK: every option and the lease time.
- `scutil --dns` shows the DNS servers DHCP supplied, as they appear in the dynamic store.
- System Settings › Wi-Fi › Details › TCP/IP has a **Renew DHCP Lease** button, which starts the exchange again.

### Fine print

- **Private Wi-Fi Address.** The DHCP server tracks devices by MAC address. A device whose MAC changes looks like a new device: it may get a different IP, and reservations or MAC-based parental controls on the router stop matching it.
- **When no server answers,** macOS gives itself a **self-assigned** address in `169.254.x.x`, the link-local range. That address only reaches other devices on the same LAN, which is why macOS shows the warning "Self-assigned IP".
- **Broadcasts don't cross routers,** so in an office where the DHCP server is on another subnet, the router runs a **DHCP relay**. The relay picks up the broadcast Discover and forwards it to the server by unicast.
- **The router gets its own public address** from the ISP's DHCP server through the same DORA exchange, one level up.
- **IPv6** usually skips DHCP for addresses. Devices build their own address from the router's announcements (SLAAC), though DHCPv6 exists too.

### scutil

`scutil`(System Configuration Utility) is MacOS command line tool for reading and changing the systems network and naming configuration.

`scutil` sends a request to daemon `configd` which keeps a dynamic store, an in-memory database of the current network state: active interfaces, IP addresses, DNS servers, proxies and more. It updates this store whenever something changes, for example when you join a new Wi-Fi network or a VPN connects.
Apps and system services read the same store through Apple's SystemConfiguration framework. `scutil` is simply the command-line way to look into it.

## Ports, UDP and TCP

An IP address gets a packet to the right device. A **port** gets it to the right app on that device. Layer 4 (transport) adds the ports, and its protocol, UDP or TCP, decides how much care goes into the delivery.

App opens a socket → kernel assigns a port → kernel adds a UDP or TCP header → IP packet → frame → … → receiving kernel matches the port to a socket → receiving app

### Ports

A port is a 16-bit number, from 0 to 65535(2^16 - 1), carried in the UDP or TCP header. Every packet has two: a source port and a destination port. The ranges are used differently:

| Range | Name | Used by |
|---|---|---|
| 0 – 1023 | Well-known | Standard services: 53 DNS, 67/68 DHCP, 80 HTTP, 443 HTTPS, 22 SSH |
| 1024 – 49151 | Registered | Other agreed services: 5353 mDNS, 5432 PostgreSQL, 3000/8000 for dev servers |
| 49152 – 65535 | Ephemeral | Temporary ports the kernel gives clients for outgoing connections |

The server side uses a fixed, agreed port so clients know where to find it. The client side gets a random ephemeral port, because nobody needs to find the client: the server just replies to whatever source port the request came from. Open five tabs to the same site and you get five connections, each with a different source port.

### Sockets

A **socket** is the kernel's object for one end of a network conversation. The app never touches it directly. It holds a **handle** to it.

#### What a handle is

A process can't reach into kernel memory, so the kernel keeps the real object (an open file, a pipe, a socket) on its own side and gives the process a small number that stands for it. That number is the **handle**. On Unix systems like macOS it's called a **file descriptor** (fd).

It works like a coat-check ticket. You hand over your coat and get ticket 7. You can't walk into the cloakroom yourself, but you can say "do something with ticket 7", and the attendant does it.

- Each process has its own **file descriptor table**: fd number → kernel object.
- `0`, `1` and `2` are taken from the start (stdin, stdout, stderr), so the first socket an app opens is usually fd `3`.
- Files, pipes and sockets all get file descriptors and share the same calls: `read(fd)`, `write(fd)`, `close(fd)`. That's the Unix idea that "everything is a file".
- The number means something only inside its own process. fd `3` in your browser and fd `3` in Slack are different objects.
- Mach port rights, in the IPC notes, are the same idea in Mach: a name the process holds for an object the kernel owns.

The socket object on the kernel side holds:

- the protocol (TCP or UDP)
- the local IP and port, and for TCP the remote IP and port
- the connection state (LISTEN, ESTABLISHED…)
- a **send buffer** and a **receive buffer**

#### A socket in use

Server, for example a web server on port 443:

1. `socket()` creates an empty socket object and returns its handle, say fd `3`.
2. `bind(fd 3, port 443)` attaches the socket to port 443.
3. `listen(fd 3)` marks it as waiting for connections.
4. `accept(fd 3)` waits until a client connects, then returns a **new** handle, say fd `4`, for that one connection. fd `3` goes on listening for the next client.

Client, for example a browser:

1. `socket()` returns a handle, say fd `3`.
2. `connect(fd 3, server IP:443)` makes the kernel pick an ephemeral source port, carry out the TCP handshake, and return once the connection is set up.

Sending and receiving:

- `write(fd, data)` copies the data into the socket's send buffer and returns. From there, the kernel splits it into segments, adds the TCP and IP headers, and sends them out.
- When packets arrive, the kernel puts their data into the socket's receive buffer. `read(fd)` copies it out to the app, and if the buffer is empty, `read` waits.

So the app only ever deals with a number and a buffer. Ports, headers, handshakes and retransmission all happen below the socket API, in the kernel.

#### Which socket gets an incoming packet?

The kernel keeps a table of all sockets and matches each arriving packet to one:

- For UDP, it matches the destination IP and port.
- For TCP, it first looks for a connection that matches all four values: source IP, source port, destination IP and destination port (the **4-tuple**). If there isn't one, it looks for a socket listening on the destination port.

That's how one server on port 443 handles thousands of clients. Each connection is a separate socket with a different 4-tuple, and each has its own handle in the server process.

If nothing is listening on the port:

- For TCP, the kernel replies with a **RST** (reset), and the client sees "connection refused".
- For UDP, it replies with an ICMP "port unreachable" message.

You can see handles directly: `lsof -nP -i :443` lists each socket using port 443, with the process that holds it and its fd number (for example `4u`, where u means open for reading and writing).

### UDP: send and forget

UDP (User Datagram Protocol) adds an 8-byte header: source port, destination port, length and checksum. That's all.

- There's no connection and no setup. The first packet already carries data.
- Each send is one self-contained message, called a **datagram**.
- Nothing checks whether a datagram arrived, so it can be lost, duplicated or arrive out of order. The app deals with that, or doesn't need to.

UDP fits when a single question gets a single answer, or when late data is worthless:

- DNS: one question, one answer. If no answer comes, the client just asks again.
- DHCP and mDNS: they have to work before a connection is even possible, and they rely on broadcast or multicast, which TCP can't do.
- Voice, video and games: a resent packet arrives too late to be useful.
- QUIC (HTTP/3): it builds its own reliability on top of UDP.

### TCP: a reliable connection

TCP (Transmission Control Protocol) turns unreliable packets into a reliable, ordered **stream of bytes**. Its header is at least 20 bytes, and the key fields are:

- source and destination ports
- a **sequence number**, the position of this data within the stream
- an **acknowledgement number**, the next byte the sender expects to receive
- **flags**: SYN (start), ACK (acknowledge), FIN (finish), RST (abort)
- a **window**, how many more bytes the receiver can accept right now

#### The 3-way handshake

Before any data flows, the two sides set up the connection:

1. **SYN.** The client says "I want to connect, and my bytes will be numbered starting from x." x is a random starting number.
2. **SYN-ACK.** The server says "Agreed. I've got your x, and my bytes start from y."
3. **ACK.** The client says "I've got your y."

Each side now knows the other's starting number, and both have confirmed that packets get through in both directions. This costs one round trip before any data is sent. That delay is one reason HTTP/3 moved to QUIC over UDP.

#### Sending data

- Every byte is numbered. The receiver acknowledges what it has received so far ("I have everything up to byte 5000").
- If an acknowledgement doesn't come back, the sender resends the missing data. That's **retransmission**.
- Segments that arrive out of order wait in the receiver's buffer until the gap is filled, so the app always reads bytes in the right order.
- The window stops a fast sender from overwhelming a slow receiver. That's **flow control**.
- When packets start getting lost, the sender assumes the network is overloaded and slows down. That's **congestion control**.

#### Closing

- Each side sends a **FIN** when it has finished sending, and the other side ACKs it. That makes four messages in total, because each direction closes separately.
- The side that closes first waits a short while in **TIME_WAIT**, so any stray packets from this connection die out before the same 4-tuple can be reused.
- **RST** ends a connection immediately, without the polite FIN exchange.

### TCP vs UDP side by side

| | UDP | TCP |
|---|---|---|
| Setup | None | 3-way handshake (one round trip) |
| Delivery | Best effort: can be lost, duplicated or reordered | Every byte arrives, in order, exactly once |
| Unit | Separate messages (datagrams) | One continuous byte stream |
| Header | 8 bytes | 20+ bytes |
| Broadcast / multicast | Yes | No, it's one-to-one only |
| Used by | DNS, DHCP, mDNS, QUIC/HTTP/3, voice and video | HTTP/1.1 and 2, TLS, SSH, databases, email |

### Ports and NAT

NAT, from the private and public IP section, depends on ports. Behind a router, every device shares one public IP, so on the way back in, the IP can't tell the router which device a reply is for. The router solves this by giving each outgoing connection its own public port and writing down which private IP and port it stands for:

1. The laptop sends from `192.168.1.23:52344` to a server on port 443.
2. The router rewrites the source to `<public IP>:61001` and records "61001 ↔ 192.168.1.23:52344" in its NAT table. Each connection gets its own public port, so five connections from the laptop use five entries.
3. The reply comes back to `<public IP>:61001`. The router finds the entry for 61001 and forwards the reply to `192.168.1.23:52344`. The stored IP gets it to the device, and the laptop's kernel then uses port 52344 to hand it to the right app.

The port identifies the connection, not the device. The router finds the device only because it stored the private IP next to that port.

For TCP, the router can see when the connection closes (FIN or RST) and removes the entry. UDP has no close, so the router removes UDP entries after a short idle timeout.

### Reading the symptoms

- "Connection refused" means the device answered with a RST. The device is reachable, but nothing is listening on that port.
- A timeout means nothing answered at all. The device is down, or a firewall is silently dropping the packets.

### On macOS

- `lsof -nP -iTCP -sTCP:LISTEN` lists the apps listening for TCP connections, and their ports.
- `lsof -nP -i :443` lists the connections using port 443.
- `netstat -an -p tcp` lists every TCP connection with its 4-tuple and state (LISTEN, ESTABLISHED, TIME_WAIT…).
- `nc -vz example.com 443` checks whether a TCP connection to that port succeeds.

## DNS

**DNS** (Domain Name System) turns a name people can remember, like `www.example.com`, into the IP address a connection needs, like `93.184.216.34`. Every connection to a named site starts with a DNS lookup, before any TCP handshake can happen.

App asks for a name → macOS stub resolver (`mDNSResponder`) → recursive resolver → root server → TLD server → authoritative server → answer travels back, cached at each step

### Names are a tree

A domain name is read right to left, from the most general part to the most specific:

```text
www.example.com.
                └── "." the root (the trailing dot is usually hidden)
            └────── com     Top-level domain (TLD)
    └────────────── example a domain someone registered
└────────────────── www     a name inside that domain
```

Each level is a **zone** run by a different party. The root zone knows who runs `com`, the `com` zone knows who runs `example.com`, and `example.com`'s own servers know the actual addresses. No single server knows every answer. Each one knows only who to ask next.

### The four roles

| Role | What it does | Example |
|---|---|---|
| **Stub resolver** | Lives on the device. Asks one question and waits for a complete answer | `mDNSResponder` on macOS |
| **Recursive resolver** | Does the legwork: walks the tree from the root, follows referrals, caches everything | Your ISP's resolver (the default), or a public one you choose: 1.1.1.1 (Cloudflare), 8.8.8.8 (Google) |
| **Root / TLD servers** | Don't know the answer, but say which servers to ask next | `a.root-servers.net`, Verisign's `.com` servers |
| **Authoritative server** | Holds the real records for a domain and gives the final answer | The name servers the domain's owner set up |

A home router often sits between the stub and the recursive resolver as a **DNS forwarder**. It doesn't walk the tree itself. It passes questions on to the ISP's resolver and caches the answers. That's why DHCP often lists the router (`192.168.1.1`) as your DNS server.

### A full lookup

The Mac wants `www.example.com`, and nothing is cached anywhere:

1. The app calls `getaddrinfo("www.example.com")`, which hands the request to `mDNSResponder`.
2. `mDNSResponder` checks its cache, misses, and sends the question to the DNS server it got from DHCP. The query has the "recursion desired" flag set, which means "give me the final answer, not a referral".
3. The recursive resolver checks its own cache, misses, and starts at the top. It has the root servers' addresses built in, in a file called **root hints**.
4. It asks a root server, which replies with a **referral**: "I don't know, but the `.com` servers are these, at these addresses."
5. It asks a `.com` server, which also refers it on: "`example.com`'s name servers are `ns1.example-dns.com`, at this address."
6. It asks `example.com`'s authoritative server, which gives the answer: "`www.example.com` A `93.184.216.34`, TTL 3600." The authoritative server flags the answer as authoritative (the **AA** flag).
7. The recursive resolver caches every step, including the root's and `.com`'s referrals, and returns the answer to the Mac.
8. `mDNSResponder` caches the answer for its TTL and hands the IP to the app, which can now open a TCP connection.

The stub asks a single **recursive** question ("get me the answer"). The recursive resolver then asks a series of **iterative** questions ("who should I ask next?").

In practice, steps 4 and 5 almost never happen: a busy recursive resolver (your ISP's, 1.1.1.1) serves so many users that it already has the root and `.com` referrals cached. Even when the name itself isn't cached, the lookup is usually one trip, straight to the authoritative server.

### Records

The answer is a **record**. Each record has a name, a type, a value and a TTL:

| Type | Maps a name to | Example |
|---|---|---|
| **A** | An IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | An IPv6 address | `example.com → 2606:2800:220:1::248` |
| **CNAME** | Another name, an alias that the resolver then looks up | `www.example.com → example.com` |
| **NS** | The name servers for a zone, which is what referrals are made of | `example.com → ns1.example-dns.com` |
| **MX** | The mail servers for a domain | `example.com → mail.example.com` |
| **TXT** | Free text, used for domain ownership checks and email security | `"v=spf1 …"` |
| **PTR** | An IP address back to a name (reverse lookup) | `34.216.184.93.in-addr.arpa → example.com` |
| **HTTPS** | Connection hints for browsers, such as HTTP/3 support | `example.com → alpn="h3,h2"` |

Browsers usually ask for A and AAAA (and often HTTPS) in parallel, then connect to whichever address works.

### Caching and TTL

Every answer has a **TTL** (time to live): how many seconds anyone may cache it. Answers are cached at every step: in the app, in `mDNSResponder`, in the router and in the recursive resolver.

- A long TTL (hours) means faster lookups and less load on the servers, but changes take a long time to reach everyone.
- A short TTL (seconds to minutes) means changes spread quickly, which suits sites that move traffic between servers.
- "Doesn't exist" answers are cached too. That's called **negative caching**, and it's why a newly created name can keep failing for a while.

### The message on the wire

A DNS query is a small UDP packet sent **to port 53 on the DNS server being asked**: the router by default, or a resolver like 1.1.1.1 if configured. The asker sends **from a random high port**, and the reply comes back from port 53 to that port. Each hop works this way (device → router → resolver → root/TLD/authoritative). The packet contains:

- a 16-bit **ID**, so the asker can match the reply to its question (the random port and the ID together make forged replies hard to slip in)
- **flags**: query or response, recursion desired/available, authoritative answer, truncated
- the question: name and type
- in replies, the answer section, plus referral and extra-address sections

Each query uses a random ID and a random source port. An attacker trying to slip in a fake reply has to guess both.

When an answer is too big for UDP, the server sets the **truncated** flag and the stub asks again over TCP port 53.

Every response carries a result code:

- **NOERROR**: here's the answer, or the name exists but has no record of that type
- **NXDOMAIN**: the name doesn't exist
- **SERVFAIL**: the resolver couldn't get an answer, for example because the authoritative server didn't respond
- **REFUSED**: the server won't answer you

### How the query travels

Using the casts and subnet rules from earlier:

- If the DNS server is the router (`192.168.1.1`), it's inside the subnet. The Mac ARPs for the router's MAC and sends the query from `192.168.1.23:<ephemeral>` to `192.168.1.1:53` by unicast. The router then makes its own query to the ISP's resolver from its public IP.
- If the DNS server is `1.1.1.1`, it's outside the subnet. The frame goes to the gateway's MAC, the router NATs the source to its public IP and port, and anycast delivers the query to the nearest Cloudflare location.

### Encrypted DNS

Classic DNS is plain text. Anyone on the path, such as the Wi-Fi network or the ISP, can read every name you look up, and can change the answers. Two encrypted alternatives exist:

- **DoT** (DNS over TLS) uses TCP port 853. It's easy to spot and easy to block.
- **DoH** (DNS over HTTPS) uses port 443. It looks like ordinary web traffic.

On macOS, an encrypted-DNS profile makes `mDNSResponder` use DoH or DoT for every app. A browser can also turn on its own DoH and bypass the system resolver entirely.

### DNS filtering

Because every connection starts with a lookup, controlling the resolver controls which names work. A filtering resolver checks each name against a blocklist and, for a blocked name, returns NXDOMAIN or a dead address like `0.0.0.0` instead of the real one. The app never learns the real IP, so it never connects.

Filtering only sees lookups that go through the filtering resolver. These bypass it:

- an app that uses its own DoH resolver
- a hardcoded IP address, which skips DNS altogether
- changing the device's DNS settings
- a cached answer from before the block

### On macOS

- `dscacheutil -q host -a name example.com` resolves through `mDNSResponder`, the same path apps take.
- `dig example.com` asks the server directly and skips `mDNSResponder` and its cache (see the mDNSResponder notes).
- `dig +trace www.example.com` walks the tree from the root itself, printing each referral.
- `dig @1.1.1.1 example.com AAAA` asks a specific resolver for a specific record type.
- `dig -x 93.184.216.34` does a reverse (PTR) lookup.

## Routing across the internet

Once a packet leaves the home router, it crosses a series of independent networks to reach the server. No single router knows the whole path. Each one only knows which neighbour to hand the packet to next.

Laptop → home router (default gateway) → ISP's access network → ISP's core → other networks (transit, peering) → destination's network → server

### Routing tables

Every device that sends IP packets has a **routing table**: a list of "for addresses in this range, send to this next hop, out of this interface". For each packet, it picks the **most specific** matching entry. That's called **longest prefix match**.

The laptop's table is tiny:

| Destination | Next hop | Meaning |
|---|---|---|
| `192.168.1.0/24` | Direct, on `en0` | My own subnet: ARP for the device and deliver it myself |
| `0.0.0.0/0` (default) | `192.168.1.1` | Everything else: hand it to the router |

`0.0.0.0/0` matches every address, so it's the fallback whenever nothing more specific matches. This is the "local or not?" rule from the OSI notes, written as a table.

- The home router's table is nearly as small: its LAN subnet, and a default route to the ISP.
- Routers deep inside the internet have **no default route**. They carry a full table of every public address range in the world, about a million IPv4 ranges, so they always know a specific next hop.

### What happens at each hop

Every router along the path does the same thing:

1. It receives a frame addressed to its own MAC address.
2. It strips off the frame. The frame only mattered on the link it just crossed.
3. It reads the destination IP and looks it up in its routing table to find the next hop.
4. It lowers the packet's **TTL** by one. If the TTL reaches zero, it drops the packet and sends an ICMP "time exceeded" message back to the sender. This stops a packet looping forever if routes are misconfigured.
5. It wraps the packet in a new frame for the next link and sends it. That link might be Ethernet, fibre or a cable modem's DOCSIS, each with its own kind of frame.

What changes along the way:

| Field | Changes |
|---|---|
| Source and destination MAC | At every hop, because each frame only spans one link |
| TTL | Drops by one at every router |
| Source IP and port | Once, at the home router's NAT |
| Destination IP and port | Never |

### Autonomous systems

The internet is a network of networks. It's made up of tens of thousands of independently run networks called **autonomous systems** (ASes). Each has a number:

| Network | AS number |
|---|---|
| Comcast | AS7922 |
| Cloudflare | AS13335 |
| Google | AS15169 |

Inside its own AS, each operator routes however it likes, usually with an internal protocol such as OSPF or IS-IS. **Between** ASes, everyone uses one protocol: BGP.

### BGP: how networks learn routes

**BGP** (Border Gateway Protocol) is how ASes tell each other which addresses they can reach:

1. Cloudflare announces to its neighbours: "`1.1.1.0/24` is reachable through me (AS13335)."
2. Each neighbour passes the announcement on, adding its own AS number to the path: "reachable through AS3356, AS13335."
3. The announcement spreads until every network knows at least one path to `1.1.1.0/24`.
4. Each router chooses among the paths it has learned. It weighs business rules first (cost, contracts), then the shortest AS path.

BGP is built on trust. If a network wrongly announces someone else's addresses, by mistake or on purpose, traffic can be pulled to the wrong place. That's called a **route leak** or **BGP hijack**, and it has caused major outages.

### How networks connect

- With **transit**, a smaller network pays a larger one to carry its traffic to the rest of the internet. Your ISP buys transit from big carriers.
- With **peering**, two networks exchange traffic between their own customers directly, usually free, because both benefit.
- An **IXP** (Internet Exchange Point) is a building where hundreds of networks plug into a shared switch so they can peer cheaply.

A large site like Google or Netflix peers directly with your ISP, so your traffic often reaches it in only a few hops.

### Anycast, revisited

Anycast is a BGP trick. Cloudflare announces `1.1.1.0/24` from hundreds of cities at once. Every router picks its best path, which is usually the shortest, so each user's packets end up at the nearest site, even though they all use the same address.

### Paths and distance

- The way back can be different. The reply is routed separately, by each network's own choices, so the return path may not match the outbound one. This is called **asymmetric routing**.
- Distance sets a lower limit on speed. Light in fibre covers about 200 km per millisecond, so a New York–London round trip can't take less than about 55 ms, however good the routers are. That's why CDNs (content delivery networks) put copies of sites in cities close to users.
- Size has a limit too. Most links carry at most 1500 bytes per packet (the **MTU**, maximum transmission unit). TCP keeps its segments small enough to fit, and a packet that's too big is either split up (fragmented) or rejected with an ICMP message.

### ICMP

**ICMP** (Internet Control Message Protocol) carries IP's error and status messages. It sits alongside IP rather than on top of UDP or TCP, and it has no ports:

- **echo request / echo reply** are what `ping` uses
- **time exceeded** is sent when a packet's TTL ran out
- **destination unreachable** covers no route to the network, no host, or (for UDP) no listener on the port

### Seeing the path

`traceroute` uses the TTL to reveal each hop:

1. It sends a packet with TTL 1. The first router lowers it to 0, drops the packet, and replies "time exceeded". That reveals hop 1.
2. It sends another with TTL 2, which reveals hop 2, and so on until the destination itself replies.

A `*` in the output means that router didn't send a reply. Many routers are set up to stay silent, but they still forward traffic normally.

### On macOS

- `netstat -rn` shows the routing table. Look for the `default` line and its gateway.
- `route -n get example.com` shows which route and interface a particular destination would use.
- `ping 1.1.1.1` checks whether a destination is reachable and shows the round-trip time.
- `traceroute example.com` lists each hop on the way.

## The full connection

Everything above, put together as one example: a MacBook at home opens `https://www.example.com`. Each step has its own flowchart, and "Send a packet" at the end is a sub-flowchart that steps 3 and 4 both use.

Rounded boxes are start and end points. Diamonds are decisions. `(A)`, `(B)`, `(C)` are connectors: "go to (B)" jumps to the box marked `(B)─▶`. `[B]` means broadcast and `[U]` unicast.

| Who | MAC | IP |
|---|---|---|
| Laptop | `L-MAC` (its Private Wi-Fi Address) | `192.168.1.23` (from DHCP) |
| Access point | `BSSID` | — |
| Router, LAN side | `R-MAC` | `192.168.1.1` |
| Router, internet side | `W-MAC` | `203.0.113.7` (public, from the ISP) |
| ISP's DNS resolver | — | `75.75.75.75` |
| Web server | — | `93.184.216.34` |

### 1. Join the Wi-Fi

```text
        ╭────────────────────────────────╮
        │ Laptop in range of a network   │
        ╰───────────────┬────────────────╯
                        ▼
        ┌────────────────────────────────┐
        │ Pick the MAC for this network  │
        │ (Private Wi-Fi Address)        │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Scan the channels:             │
        │ beacons [B], probes [B]        │
        └───────────────┬────────────────┘
                        ▼
                ╱     Known     ╲              ┌──────────────────────────────┐
               ╱     network?    ╲─ yes ──────▶│ Password from the Keychain   │
               ╲                 ╱             └───────────────┬──────────────┘
                ╲───────┬───────╱                              │
                        │ no                                   │
                        ▼                                      │
        ┌────────────────────────────────┐                     │
        │ You pick the SSID and          │                     │
        │ type the password              │                     │
        └───────────────┬────────────────┘                     │
                        ▼                                      │
        ┌────────────────────────────────┐                     │
        │ Authenticate + associate       │◀────────────────────┘
        │ with the BSSID [U]             │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ 4-way handshake [U]            │
        │ both sides derive PTK + GTK    │
        └───────────────┬────────────────┘
                        ▼
                ╱    Password   ╲              ╭──────────────────────────────╮
               ╱     correct?    ╲─ no ───────▶│ Join fails                   │
               ╲                 ╱             ╰──────────────────────────────╯
                ╲───────┬───────╱
                        │ yes
                        ▼
        ┌────────────────────────────────┐
        │ AP opens the port,             │
        │ en0 goes link up               │
        └───────────────┬────────────────┘
                        ▼
        ╭────────────────────────────────╮
        │ Go to DHCP                     │
        ╰────────────────────────────────╯
```

### 2. Get an address (DHCP)

```text
        ╭────────────────────────────────╮
        │ Link up, no IP yet             │
        ╰───────────────┬────────────────╯
                        ▼
        ┌────────────────────────────────┐
   (A)─▶│ Discover [B]                   │
        │ 0.0.0.0 → 255.255.255.255:67   │
        └───────────────┬────────────────┘
                        ▼
                ╱     Offer     ╲              ╭──────────────────────────────╮
               ╱    received?    ╲─ no ───────▶│ Self-assigned 169.254.x.x    │
               ╲                 ╱             ╰──────────────────────────────╯
                ╲───────┬───────╱
                        │ yes
                        ▼
        ┌────────────────────────────────┐
        │ Request [B]                    │
        │ "I'm taking .23 from .1"       │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ ACK [U]: IP, mask, router,     │
        │ DNS servers, lease             │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ ARP probe [B]:                 │
        │ "is anyone using .23?"         │
        └───────────────┬────────────────┘
                        ▼
                ╱    Someone    ╲              ┌──────────────────────────────┐
               ╱    answered?    ╲─ yes ──────▶│ Decline, go to (A)           │
               ╲                 ╱             └──────────────────────────────┘
                ╲───────┬───────╱
                        │ no
                        ▼
        ┌────────────────────────────────┐
        │ configd saves the settings     │
        │ (dynamic store, scutil)        │
        └───────────────┬────────────────┘
                        ▼
        ╭────────────────────────────────╮
        │ Go to DNS                      │
        ╰────────────────────────────────╯
```

### 3. Find the server's IP (DNS)

```text
        ╭────────────────────────────────╮
        │ Browser opens                  │
        │ https://www.example.com        │
        ╰───────────────┬────────────────╯
                        ▼
        ┌────────────────────────────────┐
        │ getaddrinfo() hands the name   │
        │ to mDNSResponder               │
        └───────────────┬────────────────┘
                        ▼
                ╱   Cached in   ╲              ┌──────────────────────────────┐
               ╱  mDNSResponder? ╲─ yes ──────▶│ Got the IP, go to (C)        │
               ╲                 ╱             └──────────────────────────────┘
                ╲───────┬───────╱
                        │ no
                        ▼
        ┌────────────────────────────────┐
        │ UDP query to 192.168.1.1:53    │
        │ (Send a packet, below)         │
        └───────────────┬────────────────┘
                        ▼
                ╱   Cached at   ╲              ┌──────────────────────────────┐
               ╱   the router?   ╲─ yes ──────▶│ Answer, go to (B)            │
               ╲                 ╱             └──────────────────────────────┘
                ╲───────┬───────╱
                        │ no
                        ▼
        ┌────────────────────────────────┐
        │ Router asks the ISP resolver   │
        │ 75.75.75.75:53                 │
        └───────────────┬────────────────┘
                        ▼
                ╱ Cached at the ╲              ┌──────────────────────────────┐
               ╱  ISP resolver?  ╲─ yes ──────▶│ Answer, go to (B)            │
               ╲                 ╱             └──────────────────────────────┘
                ╲───────┬───────╱
                        │ no
                        ▼
        ┌────────────────────────────────┐
        │ Resolver asks root → .com      │
        │ → authoritative server         │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
   (B)─▶│ Answer cached at each step     │
        │ on its way back: 93.184.216.34 │
        └───────────────┬────────────────┘
                        ▼
        ╭────────────────────────────────╮
        │ Go to TCP                      │
        ╰────────────────────────────────╯
```

### 4. Open the connection (TCP)

```text
        ┌────────────────────────────────┐
   (C)─▶│ socket() → fd 3, then          │
        │ connect(93.184.216.34:443)     │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Kernel picks port 52344        │
        │ and builds a SYN               │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Send the SYN to the gateway    │
        │ (Send a packet, below)         │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Router NATs the source to      │
        │ 203.0.113.7:61001              │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Internet hops: new frame and   │
        │ TTL −1 at each, BGP routes     │
        └───────────────┬────────────────┘
                        ▼
                ╱    Reached    ╲              ╭──────────────────────────────╮
               ╱   the server?   ╲─ no ───────▶│ Timeout                      │
               ╲                 ╱             ╰──────────────────────────────╯
                ╲───────┬───────╱
                        │ yes
                        ▼
                ╱   Listening   ╲              ╭──────────────────────────────╮
               ╱   on port 443?  ╲─ no ───────▶│ RST: connection refused      │
               ╲                 ╱             ╰──────────────────────────────╯
                ╲───────┬───────╱
                        │ yes
                        ▼
        ┌────────────────────────────────┐
        │ SYN-ACK back, NAT rewrites     │
        │ it to 192.168.1.23:52344       │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Laptop sends ACK:              │
        │ connection ESTABLISHED         │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ TLS + HTTP over                │
        │ the connection                 │
        └───────────────┬────────────────┘
                        ▼
        ╭────────────────────────────────╮
        │ Page loads                     │
        ╰────────────────────────────────╯
```

### Send a packet (used in steps 3 and 4)

```text
        ╭────────────────────────────────╮
        │ Packet ready                   │
        │ (destination IP known)         │
        ╰───────────────┬────────────────╯
                        ▼
                ╱  Destination  ╲              ┌──────────────────────────────┐
               ╱  in my subnet?  ╲─ yes ──────▶│ Next hop = destination       │
               ╲                 ╱             └───────────────┬──────────────┘
                ╲───────┬───────╱                              │
                        │ no                                   │
                        ▼                                      │
        ┌────────────────────────────────┐                     │
        │ Next hop = default gateway     │                     │
        │ 192.168.1.1                    │                     │
        └───────────────┬────────────────┘                     │
                        ▼                                      │
        ┌────────────────────────────────┐                     │
        │ Look up the next hop's MAC     │◀────────────────────┘
        └───────────────┬────────────────┘
                        ▼
                ╱     In the    ╲              ┌──────────────────────────────┐
               ╱    ARP cache?   ╲─ no ───────▶│ ARP request [B], reply [U]   │
               ╲                 ╱             └───────────────┬──────────────┘
                ╲───────┬───────╱                              │
                        │ yes                                  │
                        ▼                                      │
        ┌────────────────────────────────┐                     │
        │ Wrap it in a frame             │◀────────────────────┘
        │ to that MAC                    │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ Encrypt with the PTK,          │
        │ send it to the AP              │
        └───────────────┬────────────────┘
                        ▼
        ┌────────────────────────────────┐
        │ AP bridges it to the switch,   │
        │ switch sends it to that port   │
        └───────────────┬────────────────┘
                        ▼
        ╭────────────────────────────────╮
        │ Delivered to the next hop      │
        ╰────────────────────────────────╯
```

### The SYN's addresses on each part of the trip

| Part of the trip | Frame (src → dst MAC) | Source IP:port | Destination IP:port | TTL |
|---|---|---|---|---|
| Laptop → router (via AP and switch) | `L-MAC` → `R-MAC` | `192.168.1.23:52344` | `93.184.216.34:443` | 64 |
| Router → ISP | `W-MAC` → ISP gateway | `203.0.113.7:61001` | `93.184.216.34:443` | 63 |
| Each internet hop | That router → next router | `203.0.113.7:61001` | `93.184.216.34:443` | −1 per hop |
| Arriving at the server | Last router → server | `203.0.113.7:61001` | `93.184.216.34:443` | What's left |

The frame changes on every link. The source IP and port change once, at NAT. The destination never changes.
