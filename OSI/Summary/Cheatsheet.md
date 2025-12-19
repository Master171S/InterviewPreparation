### 📜 Personalized OSI Model Master Cheat Sheet

This summarizes everything we discussed, including the specific analogies and "Aha!" moments that helped you lock in the concepts.

***

#### 🗺️ The "Big Picture" Map

| Layer | Name | The Persona | Data Unit | Core Job | Key Protocols & Your "Mental Hooks" 🪝 |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **7** | **Application** | The Interface | Data | **User Intent:** Translating "I want" into network commands. | • **HTTP/S, DNS, FTP, DHCP**<br>• **Hook:** DNS is the phonebook that runs *before* the call.<br>• **Correction:** HTTPS requests security here, but L6 does the work. |
| **6** | **Presentation** | The Translator | Data | **Formatting:** Encryption, Compression, Translation. | • **TLS/SSL, JPEG, MPEG**<br>• **Hook:** SNI is the "sticky note" on the outside of the TLS handshake.<br>• **Hook:** .mp4 is a compression algorithm (codec), not just a file name. |
| **5** | **Session** | The Manager | Data | **Conversation:** Start, Stop, and Sync. | • **Session IDs**<br>• **Hook:** Phone Analogy. L4 is the "Connection" (Line open); L5 is the **"Session"** (Humans talking).<br>• **Feature:** Checkpoints allow resuming downloads. |
| **4** | **Transport** | The Postman | Segment | **Logistics:** Reliability & Multiplexing. | • **TCP (Reliable), UDP (Fast), Ports**<br>• **Hook:** **Multiplexing.** Your PC handles Spotify & Chrome at once using Port Numbers (Apartment #s).<br>• **Hook:** Ephemeral Ports are the temporary return addresses. |
| **3** | **Network** | The Navigator | Packet | **Path Selection:** Global Addressing & Routing. | • **IP, ICMP, Routers**<br>• **Hook:** **TTL** is a "hop counter" to kill zombie packets.<br>• **Hook:** **Routers** connect different networks; **Switches** connect the same network. |
| **2** | **Data Link** | Local Delivery | Frame | **Physical Addressing:** Hop-to-hop transfer. | • **Ethernet, MAC, Switches**<br>• **Hook:** **ARP** is the glue (IP $\rightarrow$ MAC).<br>• **Case:** The Two-Faced Router. Rogue DHCP servers cause IP conflicts and ARP flapping. |
| **1** | **Physical** | The Physics | Bits | **Transmission:** Raw signals on the wire. | • **Cables, Fiber, Wi-Fi**<br>• **Hook:** **No Header.** It is the stream.<br>• **Hook:** **Troubleshooting.** Always check the "blinking lights" first. |

***

#### 🕵️‍♂️ The "Detective" Toolkit (Troubleshooting)

| The Problem | The Tool / Cause / Fix |
| :--- | :--- |
| "I have an IP, but I can't build a Frame." | **The Tool:** ARP ("Who has this IP? Tell me your MAC!"). |
| "I'm visiting bank.com securely, but the ISP knows where I am." | **The Cause:** SNI (Server Name Indication) is sent in plain text at Layer 6. |
| "The internet works, then stops, then works." | **The Cause:** IP Conflict / Rogue DHCP. Two devices fighting over the Gateway IP.<br>**The Fix:** DHCP Snooping on the Layer 2 Switch. |