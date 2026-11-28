# Network Protocols & Internet Connectivity

## 1. What is a Network Protocol?
* **Protocol:** A standardized set of rules, formats, and messages that govern how devices communicate and exchange data across a network, forming an **Internet standard**.
* **Why Protocols Matter:** Without shared protocols, devices built by different manufacturers running different operating systems would not understand each other's signals or data structures.

---

## 2. Core Network & Application Protocols

### 2.1 ARP (Address Resolution Protocol)
* **Definition:** Resolves a known Layer 3 **IP Address** to an unknown Layer 2 **MAC Address** on a local network segment.
* **Key Messages:** **ARP Request** (Broadcast) and **ARP Reply / Response** (Unicast).
* **RFC Reference:** **RFC 826** (*An Ethernet Address Resolution Protocol*).

#### Detailed Explanation of ARP & RFC 826
1. **How ARP Works (Step-by-Step):**
   * **Cache Check:** Before sending data on a local network, a host checks its local **ARP Cache (ARP Table)** to see if it already knows the MAC address matching the destination IP (or Default Gateway IP).
   * **ARP Request (Broadcast):** If no entry exists, the host broadcasts an **ARP Request** frame to `FF:FF:FF:FF:FF:FF` asking every device on the local subnet: *"Who has IP `x.x.x.x`? Tell `y.y.y.y`."*
   * **ARP Reply (Unicast):** Only the device owning that target IP responds directly (unicast) with an **ARP Reply** containing its MAC address.
   * **Caching:** The sender stores the `IP -> MAC` mapping in its ARP table (with a timeout/TTL) so future packets can be sent without repeating the broadcast.
2. **What RFC 826 Defines:**
   * **RFC (Request for Comments):** The formal engineering implementation guidelines published by the IETF that define how Internet standards work.
   * **RFC 826** specifically standardizes the ARP packet structure (Hardware Type, Protocol Type, Sender/Target MAC, Sender/Target IP), opcode values (`1` for Request, `2` for Reply), and cache update rules so all network vendors implement ARP identically.
3. **Why It Is Important for the Internet:**
   * **Bridges Layer 3 and Layer 2:** Applications and routing operate using logical **IP addresses**, while physical network interfaces (Ethernet/Wi-Fi) deliver frames using hardware **MAC addresses**. Without ARP, packets could never be encapsulated into Layer 2 frames to leave a host or traverse between routers hop-by-hop.

---

### 2.2 FTP (File Transfer Protocol)
* **Definition:** A standard client-server protocol used to upload, download, and manage files between hosts over a TCP/IP network.

#### Detailed Explanation of FTP
* **How It Works:**
  * FTP uses **TCP** for reliable delivery and uniquely splits communication into **two separate connections**:
    1. **Control Connection (Port 21):** Stays open for the entire session to send authentication credentials (`USER`, `PASS`) and commands (`LIST`, `RETR`, `STOR`, `CWD`).
    2. **Data Connection (Port 20 or dynamic ephemeral ports):** Opens temporarily whenever actual file contents or directory listings are transferred.
  * **Active vs. Passive Mode:** In *Active* mode, the server initiates the data connection back to the client; in *Passive (PASV)* mode, the client initiates both connections (crucial for passing through modern firewalls and NAT).
* **Security Note:** Standard FTP transmits usernames, passwords, and files in **cleartext** (unencrypted), which is why modern deployments replace it with **SFTP** (over SSH) or **FTPS** (FTP over TLS).
* **Why It Is Important for the Internet:**
  * FTP established the foundational model for reliable remote file distribution, website publishing, and software repository hosting across heterogeneous operating systems on the early and modern Internet.

---

### 2.3 SMTP (Simple Mail Transfer Protocol)
* **Definition:** The standard push protocol used to send and relay email messages from a client to a mail server and between mail servers across the Internet.

#### Detailed Explanation of SMTP
* **How It Works:**
  * Operates over **TCP** (commonly **Port 25** for server-to-server relay, **Port 587** for authenticated client submission, and **Port 465** for implicit TLS).
  * Uses a structured text-based command/response conversation (`HELO`/`EHLO`, `MAIL FROM`, `RCPT TO`, `DATA`, `QUIT`) with 3-digit status codes (e.g., `250 OK`, `550 Mailbox unavailable`).
  * **Push vs. Pull:** SMTP only **pushes** outgoing mail toward the recipient's mail server. To retrieve/read emails from the inbox, clients use pull protocols like **IMAP** or **POP3**.
* **Why It Is Important for the Internet:**
  * SMTP is the universal backbone of global electronic mail, enabling decentralized mail servers operated by completely independent organizations (e.g., Gmail, Outlook, corporate servers) to reliably route messages worldwide.

---

### 2.4 HTTP (HyperText Transfer Protocol)
* **Definition:** The foundational application-layer protocol used for transmitting web pages, APIs, images, and hypermedia between web clients (browsers/apps) and web servers.

#### Detailed Explanation of HTTP
* **How It Works:**
  * Follows a **Request–Response** architecture typically over **TCP Port 80** (and QUIC/UDP in HTTP/3).
  * **Stateless Design:** Each HTTP request is independent; servers maintain user sessions across requests using **Cookies**, **Headers**, or **Tokens**.
  * **HTTP Methods (Verbs):** Defines the desired action—such as `GET` (retrieve data), `POST` (submit/create data), `PUT`/`PATCH` (update), and `DELETE` (remove).
  * **Status Codes:** Servers return standardized 3-digit codes indicating outcomes: `2xx` (Success, e.g., `200 OK`), `3xx` (Redirection), `4xx` (Client Error, e.g., `404 Not Found`), and `5xx` (Server Error).
* **Security Limitation:** Plain HTTP sends all headers, cookies, credentials, and payloads in **plaintext**, making traffic vulnerable to eavesdropping and Man-in-the-Middle (MITM) tampering.
* **Why It Is Important for the Internet:**
  * HTTP is the core protocol of the **World Wide Web (WWW)** and modern **REST/GraphQL APIs**, powering virtually all browser navigation, cloud services, and mobile-to-backend communication.

---

### 2.5 SSL (Secure Sockets Layer)
* **Definition:** The original cryptographic security protocol designed to provide privacy, authentication, and data integrity for communications over a computer network.

#### Detailed Explanation of SSL
* **How It Works:**
  * Sits between the Transport Layer (TCP) and Application Layer protocols (like HTTP or SMTP) to wrap plaintext traffic in an encrypted tunnel.
  * Combined **Asymmetric Cryptography** (public/private keys and digital certificates to verify identity and exchange secrets) with **Symmetric Cryptography** (fast bulk encryption of session data) and **MACs (Message Authentication Codes)** to detect tampering.
* **Lifecycle & Deprecation:**
  * Developed by Netscape in the mid-1990s (`SSL 1.0`, `SSL 2.0`, `SSL 3.0`).
  * All versions of SSL are now **officially deprecated** due to critical cryptographic vulnerabilities (such as POODLE and DROWN) and have been superseded by **TLS**.
* **Why It Is Important for the Internet:**
  * SSL pioneered encrypted web traffic and digital certificate trust models, making online banking, e-commerce, and private logins possible on the public Internet.

---

### 2.6 TLS (Transport Layer Security)
* **Definition:** The modern, cryptographically secure successor to SSL that encrypts and authenticates network communications between clients and servers.

#### Detailed Explanation of TLS
* **How It Works (The TLS Handshake):**
  1. **Negotiation (`ClientHello` / `ServerHello`):** Client and server agree on the TLS version (e.g., **TLS 1.2** or **TLS 1.3**) and a strong **Cipher Suite**.
  2. **Authentication:** The server presents its **X.509 Digital Certificate**, signed by a trusted **Certificate Authority (CA)**, proving its identity.
  3. **Key Exchange:** Both sides securely derive shared **Symmetric Session Keys** (using Diffie-Hellman / ECDHE, which also provides *Forward Secrecy*).
  4. **Encrypted Application Data:** All subsequent application traffic is encrypted and integrity-protected using the shared session keys.
* **Why It Is Important for the Internet:**
  * TLS is the primary cryptographic shield of the modern Internet—securing HTTPS, email transport, VPNs, instant messaging, and APIs against eavesdropping, impersonation, and data modification.

---

### 2.7 HTTPS (HyperText Transfer Protocol Secure)
* **Definition:** **HTTP** layered on top of **SSL/TLS** encryption, providing secure, authenticated web communication.

#### Detailed Explanation of HTTPS
* **How It Works:**
  * Operates by default over **TCP Port 443**.
  * Before any HTTP request (`GET`, `POST`, cookies, credentials) is transmitted, the client and server first establish an encrypted **TLS tunnel**. The entire HTTP message—including URL paths, query parameters, headers, and body—is encrypted inside that tunnel.
  * Guarantees three core security properties:
    1. **Confidentiality:** Attackers sniffing network packets cannot read the data.
    2. **Integrity:** Data cannot be altered or injected in transit without detection.
    3. **Authentication:** Users can verify they are communicating with the legitimate website rather than an impostor.
* **Why It Is Important for the Internet:**
  * HTTPS protects user privacy, passwords, payment details, and session tokens across untrusted networks (such as public Wi-Fi and ISP infrastructure), making the modern web safe for everyday use.

---

### 2.8 DNS (Domain Name System)
* **Definition:** The hierarchical, decentralized naming system that translates human-readable **Domain Names** (e.g., `google.com`) into machine-routable **IP Addresses** (e.g., `142.250.190.78`).

#### Detailed Explanation of DNS
* **How DNS Resolution Works (Step-by-Step):**
  * Operates primarily over **UDP Port 53** for fast standard queries (and **TCP Port 53** for large responses or zone transfers).
  * **Resolution Hierarchy:**
    1. **Local Cache Check:** Checks browser cache, OS cache, and the local `hosts` file.
    2. **Recursive DNS Resolver:** Queries the configured ISP or public resolver (e.g., `8.8.8.8` or `1.1.1.1`).
    3. **Root Name Servers (`.`):** Directs the resolver to the appropriate Top-Level Domain server.
    4. **TLD Name Servers (e.g., `.com`, `.org`, `.net`):** Directs the resolver to the domain's authoritative server.
    5. **Authoritative Name Server:** Returns the actual DNS record (e.g., **`A` record** for IPv4, **`AAAA` record** for IPv6, **`CNAME`** for alias, **`MX`** for mail) back to the client.
* **Why It Is Important for the Internet:**
  * Often called the **"Phonebook of the Internet,"** DNS allows humans to navigate millions of websites using memorable names instead of memorizing 32-bit or 128-bit numeric IP addresses, while allowing server IPs to change behind the scenes without breaking links.

---

## 3. Essential Host Configuration & DHCP

### 3.1 The 4 Items Every Host Needs for Internet Connectivity
To communicate locally and reach external networks on the Internet, every host requires four foundational network parameters:

| # | Configuration Item | Purpose |
| :--- | :--- | :--- |
| **1** | **IP Address** | Uniquely identifies the host on the network so packets can be addressed to and from it. |
| **2** | **Subnet Mask** | Distinguishes the **Network portion** from the **Host portion** of the IP, telling the host whether a destination is on the local subnet or a remote network. |
| **3** | **Default Gateway (Router IP)** | The local IP address of the router interface that forwards traffic destined for external networks (outside the local subnet). |
| **4** | **DNS Server IP(s)** | The IP address(es) of DNS resolvers used to translate domain names into IP addresses. |

---

### 3.2 DHCP (Dynamic Host Configuration Protocol)
* **Definition:** A network management protocol that automatically assigns IP configuration parameters to hosts joining a network.

#### Detailed Explanation of DHCP
* **How It Works (The DORA Process):**
  * Operates over **UDP** (**Port 67** for the DHCP Server, **Port 68** for the DHCP Client) using a 4-step exchange known as **DORA**:
    1. **D — Discover:** A newly connected host (with no IP yet) sends a broadcast `DHCP Discover` message (`0.0.0.0` $\rightarrow$ `255.255.255.255`) to locate available DHCP servers.
    2. **O — Offer:** The DHCP server replies with a `DHCP Offer`, proposing an available IP address from its pool along with a **Lease Time**.
    3. **R — Request:** The client broadcasts a `DHCP Request` formally accepting the offered IP (and informing any other DHCP servers that it chose that offer).
    4. **A — Acknowledge (ACK):** The server finalizes the assignment with a `DHCP ACK`, delivering all **4 required connectivity items** (IP Address, Subnet Mask, Default Gateway, and DNS Server IPs).
  * **Leases & Renewal:** IP assignments are temporary (*leases*) and automatically renew (typically at 50% of the lease duration), recycling unused IPs when devices disconnect.
* **Why It Is Important for the Internet:**
  * Without DHCP, administrators or users would have to manually configure all 4 network items on every laptop, phone, and server—and manually track addresses to prevent **IP conflicts**. DHCP makes plug-and-play Internet connectivity possible at massive scale across homes, enterprises, and mobile networks.

---

## 4. Quick-Reference Protocol Summary Table

| Protocol | Full Name | Default Port(s) / Transport | Core Role | Why It Matters for the Internet |
| :--- | :--- | :--- | :--- | :--- |
| **ARP** | Address Resolution Protocol | Layer 2 / Layer 3 Bridge | Maps IP addresses to MAC addresses locally | Enables Layer 3 IP packets to be delivered over Layer 2 physical links |
| **FTP** | File Transfer Protocol | TCP 20 (Data), 21 (Control) | Transfers and manages files between hosts | Standardized reliable remote file distribution across systems |
| **SMTP** | Simple Mail Transfer Protocol | TCP 25, 587, 465 | Sends and relays outgoing emails | Powers global, interoperable email delivery across providers |
| **HTTP** | HyperText Transfer Protocol | TCP 80 | Transfers web content and API requests/responses | Serves as the core communication protocol of the World Wide Web |
| **SSL** | Secure Sockets Layer | Cryptographic Layer *(Deprecated)* | Early encryption & authentication for network traffic | Pioneered encrypted web commerce before being replaced by TLS |
| **TLS** | Transport Layer Security | Cryptographic Layer | Modern encryption, integrity, and identity verification | Protects Internet traffic against eavesdropping and MITM tampering |
| **HTTPS** | HTTP Secure | TCP 443 (HTTP over TLS) | Secures web browsing and API traffic | Keeps passwords, payments, and private web data confidential |
| **DNS** | Domain Name System | UDP/TCP 53 | Translates domain names into IP addresses | Acts as the Internet's phonebook so humans use names instead of IPs |
| **DHCP** | Dynamic Host Configuration Protocol | UDP 67 (Server), 68 (Client) | Dynamically assigns IP, Mask, Gateway, and DNS | Automates plug-and-play network setup and prevents IP conflicts |