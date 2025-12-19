# Linux Networking & Process Discovery

## 1. Checking Active Connections
When you have shell access but no GUI, you need to see what the server is doing.

### Commands
*   `netstat -antp`
*   `ss -lntp` (The modern replacement)

### The Flags (Memorize These)
*   **-a (All):** Show listening and non-listening sockets.
*   **-n (Numeric):** Don't resolve DNS (shows IPs instead of names, which is faster).
*   **-t (TCP):** Show only TCP connections.
*   **-p (Process):** Show which program (PID/Name) is using the port.

---

## 2. Port Binding (The "Invisible" Services)
*   **Listening on `0.0.0.0`:** The service accepts connections from anywhere. (Visible to Nmap).
*   **Listening on `127.0.0.1`:** The service accepts connections only from **localhost**. (Invisible to Nmap).
    *   **Significance:** You might find vulnerable services (like an unpatched MySQL database) running internally that you couldn't see from the outside.

---

## 3. Common Ports to Know
*   **22:** SSH
*   **53:** DNS
*   **80 / 443:** HTTP / HTTPS
*   **3306:** MySQL Database