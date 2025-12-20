# Command Injection & Remote Code Execution (RCE)

A high-impact vulnerability where an attacker executes arbitrary system commands on a host server, potentially leading to full system compromise.

---

## 1. The Concept
*   **Definition:** Command Injection occurs when an application passes unsafe user-supplied data (from forms, cookies, or HTTP headers) to a system shell.
*   **The "Holy Grail":** Unlike SQLi (which targets the database) or XSS (which targets the browser), RCE gives the attacker direct control over the **Operating System**.
*   **Goal:** Execute system commands to steal sensitive data, compromise the server, or pivot into the internal network.

---

## 2. The Mechanics: Breaking Out of the Shell
Attackers use **Shell Operators** to chain commands together or manipulate the execution flow.

*   **`;` (Separator):** Runs the second command regardless of whether the first one succeeded. (Preferred by attackers for reliability).
*   **`&&` (Logical AND):** Runs the second command *only* if the first one succeeds.
*   **`|` (Pipe):** Passes the output of the first command as input to the second.
*   **`||` (Logical OR):** Runs the second command *only* if the first one fails.

---

## 3. The Scenario: The Vulnerable "Ping" Tool
Imagine a router admin panel that lets users ping an IP address to check connectivity.

### The Flawed Code (Python Example)
```python
import os
# VULNERABLE: Concatenating strings sends the input directly to the shell
command = "ping -c 4 " + user_input
os.system(command)
```

### The Attack
*   **Input:** `127.0.0.1; whoami`
*   **Execution:** The shell sees two distinct commands: `ping -c 4 127.0.0.1` and `whoami`.
*   **Result:** The system pings the address and then immediately prints the name of the current user.

---

## 4. Impact & Post-Exploitation

### Initial Access
You usually land as a low-privileged service user (e.g., `www-data`), not `root`.
*   **Why?** Service accounts have limited permissions to contain damage if the app is hacked.
*   **Restricted Shells:** These accounts often use `/usr/sbin/nologin` to prevent interactive logins.

### Enumeration Targets
Once access is gained, attackers look for:
*   **`/etc/passwd`:** World-readable; contains user information.
*   **`/etc/shadow`:** Root-readable only; contains password hashes (usually inaccessible initially).
*   **SUID Binaries:** The primary path for **Privilege Escalation** to become `root`.

---

## 5. The Defense: Separation of Code and Data

### The Fix
Do not use a shell to interpret the command. Use functions that accept command arguments as a **List** rather than a single string.

### Secure Code (Python Example)
```python
import subprocess
# SECURE: The OS treats the list items as arguments, not executable shell code.
subprocess.run(["ping", "-c", "4", user_input])
```

### Why this works:
If the user inputs `127.0.0.1; whoami`:
*   **Vulnerable (`os.system`):** The shell parses `;` as a command separator and runs both.
*   **Secure (`subprocess`):** The system looks for a single host named `"127.0.0.1; whoami"`. Since that host doesn't exist, the ping fails safely. The `whoami` command is never parsed or executed.