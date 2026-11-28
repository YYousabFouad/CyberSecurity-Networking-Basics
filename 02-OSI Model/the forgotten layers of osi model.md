# The Forgotten Layers of the OSI Model (Layers 5, 6, and 7)

Most networking guides spend almost all their time on Layers 1–4 and treat Layers 5–7 as something to just memorize. In reality, **Session (Layer 5)**, **Presentation (Layer 6)**, and **Application (Layer 7)** each solve a clear, practical problem needed for two computers to share data.

---

## 1. Where Layers 5–7 Fit in the OSI Model

Layers 1–4 deliver raw `1`s and `0`s from one application process to another; Layers 5–7 make sure those `1`s and `0`s reach the right user, get decoded properly, and trigger the right action:

| Layer | Name | Core Responsibility |
| :--- | :--- | :--- |
| **7** | **Application** | Defines application commands (what to *do* with the interpreted data) |
| **6** | **Presentation** | Interprets raw `1`s and `0`s into characters, numbers, or data formats |
| **5** | **Session** | Distinguishes individual user sessions independently from L2/L3/L4 addresses |
| **4** | **Transport** | **Service-to-Service** delivery (Ports) |
| **3** | **Network** | **End-to-End** delivery (IP Addresses) |
| **2** | **Data Link** | **Hop-to-Hop** delivery (MAC Addresses) |
| **1** | **Physical** | Transports raw bits (`0`s and `1`s) over cables or Wi-Fi |

---

## 2. Layer 5 — Session Layer (Identifying User Sessions)

### Why Layer 5 Was Created (Mainframes & Dumb Terminals)
The OSI Model was designed before personal computers existed, when users accessed giant room-sized **mainframes** through **dumb terminals** (just a monitor and keyboard—all computing happened on the mainframe, which is also why Linux is a multi-user OS):

```text
  User A (Blue) --+                                 +-- User C (Blue)
                  |--> [Mainframe 1] === [Mainframe 2] <--|
  User B (Green) -+     Same MAC/IP       Same MAC/IP  +-- User D (Green)
```

- When multiple users on **Mainframe 1** talked to users on **Mainframe 2** at the same time, every packet shared the **same Layer 2 MAC**, **same Layer 3 IP**, and potentially even the **same Layer 4 Port**.
- **Problem:** How do you keep User A's (Blue) data separate from User B's (Green) data?
- **Layer 5 Solution:** Identify each **user session** independently from the Layer 2, 3, and 4 addresses below it.

### Modern Example: HTTP Cookies
Suppose you log into `site.com` on your phone using **home Wi-Fi**, then walk to a **coffee shop** and join the coffee shop's Wi-Fi:
1. Switching Wi-Fi networks gives your phone a **new IP address** (Layer 3 changes).
2. If the website tracked who you were only using Layer 1–4 information, you would get logged out every time you changed networks.
3. Instead, after you log in, the server creates an **HTTP Cookie** (an arbitrary text string identifying your session) and sends it to your browser.
4. When your IP address changes at the coffee shop, your browser still sends that same cookie—so `site.com` still recognizes your **session**.

---

## 3. Layer 6 — Presentation Layer (Interpreting the 1s and 0s)

Once Layers 1–5 have delivered the packet to the right **MAC**, **IP**, **Port**, and **User Session**, the payload is still just a stream of **`1`s and `0`s**.

- **Layer 6's job:** Tell the receiving device **how to group and interpret** those `1`s and `0`s.

The exact same `1`s and `0`s mean completely different things depending on the encoding chosen:
- **4-bit groups → Hexadecimal** (each 4 bits becomes one hex digit).
- **6-bit groups → Base64** (each 6 bits becomes one Base64 character).
- **32-bit or 64-bit groups → Numbers** (large decimal integers).
- **8-bit groups → Extended ASCII** (each 8 bits becomes a text character).

### Modern Example: HTTP Uses Extended ASCII
In **HTTP**, Layer 6 interprets every 8 bits using **Extended ASCII encoding**:
- Byte 1 → `G`
- Byte 2 → `E`
- Byte 3 → `T`
- Continuing across the stream reveals characters like `GET /simple.html HTTP/1.1` *(as seen when sending an [HTTP request manually using Telnet](../06-Network%20Protocols/How%20to%20send%20HTTP%20request%20using%20Telnet%20manually.md))*.

---

## 4. Layer 7 — Application Layer (Executing Commands)

Once Layer 6 has turned the `1`s and `0`s into readable characters, **Layer 7 defines what those characters actually mean to the application**:

```text
Raw bits (L1-L5) ---> L6 (Extended ASCII) ---> "GET /simple.html HTTP/1.1"
                                                       |
                                                       v
                      L7 (HTTP Command)   ---> Fetch simple.html from pracnet.net
```

- In HTTP, `GET` is an **application command** instructing the web server to retrieve and return the requested webpage.
- Every application protocol defines its own set of Layer 7 commands (for example, `GET`/`POST` in HTTP vs. `RETR`/`LIST` in FTP).

---

## 5. Why TCP/IP Models Combine Layers 5, 6, and 7

Each protocol is free to implement Layers 5, 6, and 7 however it wants—or skip a layer altogether. Comparing **HTTP** and **FTP** makes this clear:

| Protocol | Layer 5 (Session) | Layer 6 (Presentation) | Layer 7 (Application) |
| :--- | :--- | :--- | :--- |
| **HTTP** | **HTTP Cookies** (survives IP changes) | Extended ASCII encoding | HTTP commands (`GET`, `POST`, etc.) |
| **FTP** | **Not implemented** — relies on L3 IP + L4 Port (if your IP changes, you must log back in) | Extended ASCII encoding | FTP commands (`LIST`, `RETR`, `STOR`, etc.) |

Because every protocol implements session tracking, encoding, and commands in its own way inside the application itself, both the **5-layer** and **4-layer TCP/IP models** combine Layers 5, 6, and 7 into one single **Application Layer**.

---

## 6. The Big Lesson: Models Are Layers of Abstraction

Whether you reference the **7-layer OSI Model**, **5-layer TCP/IP Model**, or legacy **4-layer TCP/IP Model**:
- **They are mental models, not rigid laws:** Real protocols do not strictly snap into one box without overlapping—and trying to force every protocol into exact OSI buckets misses the point.
- **Separation of responsibilities (Abstraction):**
  - Each layer does its own job so the other layers do not have to worry about it.
  - **Layer 3** handles end-to-end delivery without caring how individual links move bits underneath.
  - **HTTP Cookies (Layer 5)** work identically whether the network underneath runs **IPv4** or **IPv6**.
- Understanding this independence makes advanced networking concepts like **tunneling**, **underlays**, and **overlays** much easier to grasp later on.
