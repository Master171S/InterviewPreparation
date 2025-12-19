# Junior Pen Tester Interview Cheat Sheet

Essential concepts for technical interviews, focusing on network scanning mechanics, shell connectivity, and subnetting fundamentals.

---

## 1. Networking: The TCP SYN Scan
*   **The Mechanism:** `SYN` -> `SYN-ACK` -> `RST` (Reset).

### Why it's "Stealthy" (The "Green Flag" Explanation)
*   **The Application Layer Gap:** Web servers (like Apache) typically only log a connection after the 3-way handshake is complete.
*   **The OS Hand-off:** Because we send a `RST` immediately, the OS drops the connection before the web server software ever sees it. 
*   **Result:** Your IP does not appear in the standard `access.log`.

### Term to Remember: "Traffic Signature"
*   **Clarification:** This refers to the pattern of packets seen by security devices (IDS/Firewall).
*   **To a Web Server:** Silence (Invisible).
*   **To an IDS:** A loud alarm. The "Signature" is thousands of requests (`SYN`) without completion (`ACK`), which triggers suspicion.

---

## 2. Shells: Reverse vs. Bind
*   **Concept:** Direction is everything.

### Bind Shell
*   **Direction:** Attacker connects to Victim.
*   **Obstacle:** Usually blocked by **Ingress** (incoming) firewall filters.

### Reverse Shell
*   **Direction:** Victim connects back to Attacker.
*   **Why it works:** Firewalls usually have strict Ingress rules but loose **Egress** (outgoing) rules. A reverse shell mimics normal outgoing traffic (like browsing the web).

---

## 3. Subnetting & Safety
*   **Scope Example:** `192.168.10.0/24`

### Danger Zone
*   **.0 (Network Address):** Do not scan this address.
*   **.255 (Broadcast Address):** Scanning this is "noisy" and can cause **broadcast storms**, potentially disrupting the network.

### Correction to Remember
*   **Common Mistake:** Thinking "/24" means the first 3 *bits* are full.
*   **The Reality:** "/24" means the first **24 bits** (or 3 full octets) are fixed/masked.