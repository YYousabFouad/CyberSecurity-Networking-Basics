# How to Send an HTTP Request Using Telnet Manually

Web browsers do not load pages by magic—under the hood they open a TCP connection, send a plain-text **HTTP** request, receive raw **HTML** text back, and render it visually. You can run that exact same conversation yourself from a terminal using **Telnet**.

---

## 1. The Core Pattern Behind Network Protocols

Almost every network protocol follows the same four-step sequence:

```text
[ Client ] --- 1. Open Connection (TCP/UDP) ---> [ Server ]
           --- 2. Send Request / Command    --->
           <-- 3. Receive Response / Data   ---
           --- 4. Render or Use the Data
```

- **Why this demonstration matters:** Testing HTTP with Telnet shows both how web browsing actually works and the basic pattern shared by nearly all application protocols.

---

## 2. HTTP vs. HTML (Know the Difference)

| Concept | Full Name | Role | Example |
| :--- | :--- | :--- | :--- |
| **HTTP** | *HyperText Transfer Protocol* | Messaging rules to request and transfer web pages | `GET /simple.html HTTP/1.1` |
| **HTML** | *HyperText Markup Language* | Tag-based text describing how page content looks | `<H1>Title</H1>`, `<strong>bold</strong>` |

### How HTML Tags Work
In a browser's **View Page Source**, a simple web page (`http://pracnet.net/simple.html`) is just plain text wrapped in opening and closing **tags**:

```html
<HTML>
<HEAD>
  <TITLE>This is a Simple HTML Page</TITLE>
  <link rel="icon" href="favicon.png" />
</HEAD>
<BODY>
  <H1>This is a Title</H1>
  <hr>
  <H2>This is a Subtitle</H2>
  <p>This is some text.</p>
  <p>This is some <strong>bold</strong> text.</p>
  <p>This is some <em>italic</em> text.</p>
  <p>This is an image: <BR />
    <img src="//pracnet.net/favicon.ico">
  </p>
  <hr>
</BODY>
</HTML>
```

- `<HTML> ... </HTML>` — Marks the start and end of the HTML document.
- `<H1>`, `<H2>` — Heading (title / subtitle) tags; `<hr>` draws a horizontal dividing line.
- `<p> ... </p>` — Paragraph text blocks.
- `<strong> ... </strong>` — Renders text as **bold**.
- `<em> ... </em>` — Renders text (*emphasis*) as *italic*.
- `<img src="...">` — Tells the browser which image URL to display.

> Real websites (like Wikipedia) contain much larger HTML/CSS/JS source files, while this small page highlights the bare-bones mechanics cleanly.

---

## 3. Step-by-Step: Manual HTTP Request with Telnet

### Step 1: Open a TCP Connection on Port 80
Port **80** is the standard port for **unencrypted HTTP** (sites shown without HTTPS / padlock security).

Run this in a Linux terminal (or Windows with the Telnet client installed):

```bash
telnet pracnet.net 80
```

Telnet resolves the domain name to its IP address and opens a raw TCP session:

```text
Trying 104.225.208.27...
Connected to pracnet.net.
Escape character is '^]'.
```

### Step 2: Type the Raw HTTP Request
Once connected, type the request lines exactly as below, then press **Enter twice** (one Enter after `Host: pracnet.net`, and a second Enter to send a blank line):

```http
GET /simple.html HTTP/1.1
Host: pracnet.net

```

**What each line means:**
- `GET` — HTTP method asking the server to send back a resource.
- `/simple.html` — Path of the specific page requested.
- `HTTP/1.1` — Version of the HTTP protocol being spoken.
- `Host: pracnet.net` — Tells the web server which domain you want (required in `HTTP/1.1` since one server IP can host multiple websites).
- **Blank line (`Enter` twice)** — Signals to the server that your HTTP request headers are complete.

### Step 3: Read the Server's HTTP Response
The web server immediately sends back an **HTTP Response** containing response **headers**, a blank line, and the raw **HTML body**:

```http
HTTP/1.1 200 OK
Connection: Keep-Alive
Keep-Alive: timeout=5, max=100
content-type: text/html
last-modified: Tue, 12 Sep 2023 18:12:09 GMT
accept-ranges: bytes
content-length: 408
date: Wed, 13 Sep 2023 23:08:49 GMT
server: LiteSpeed

<HTML>
<HEAD>
  <TITLE>This is a Simple HTML Page</TITLE>
  <link rel="icon" href="favicon.png" />
</HEAD>
...
```

**Key parts of the response:**
- `HTTP/1.1 200 OK` — Status line confirming the request succeeded.
- `content-type: text/html` — Tells the client the body contains HTML text.
- `content-length: 408` — Size of the HTML payload in bytes (`408` bytes).
- **HTML Body** — The exact same `<HTML> ... </HTML>` text visible in the browser's **View Page Source**.

---

## 4. What a Web Browser Really Does

```text
User enters URL ---> Browser opens TCP connection (Port 80)
                ---> Sends: GET /simple.html HTTP/1.1 + Host header
                <--- Receives: HTTP 200 OK + raw <HTML> text
                ---> Renders HTML tags visually on the screen
```

Your browser is simply automating the exact Telnet + `GET` exchange you ran manually, then translating the returned HTML tags into headings, bold/italic text, lines, and images.

---

## 5. Why Telnet Works for HTTP (and Its Limits)

- **Telnet speaks plain ASCII text:** Any protocol built around human-readable **ASCII commands** (such as plain **HTTP**, **SMTP**, or unencrypted **FTP** control channels) can be tested manually with Telnet.
- **Protocols Telnet cannot easily replicate:**
  - **Binary protocols:** Protocols expecting raw binary byte chunks instead of ASCII text.
  - **Encrypted or encoded protocols:** Protocols like **HTTPS (HTTP over TLS on Port 443)** encrypt every byte after the TLS handshake, so raw ASCII typed directly over Telnet will fail.
- Even when protocols use binary framing or encryption, the core workflow stays identical: **connect → send request → receive response → process/render data**.
