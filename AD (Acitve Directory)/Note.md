# Active Directory Fundamentals (Interview Prep)

## 1. The Geography (Structure)
• **Domain (The Castle):** The core security boundary (e.g., `CorpHotel.local`). Has its own admins and users.

• **Tree (The Region):** A group of domains sharing a namespace (e.g., `UK.CorpHotel.local` and `US.CorpHotel.local`).

• **Forest (The Empire):** The collection of all domains and trees. The ultimate security boundary.

## 2. Organization & Rules (Confusion Point ⚠️)
• **OU (Organizational Unit):** The Folder. It organizes objects (e.g., "HR Computers"). It does not apply settings by itself.

• **GPO (Group Policy Object):** The Rule Book. It contains settings (e.g., "Disable USB").

• **The Logic:** You create a GPO (Rule) and link it to an OU (Folder).

## 3. Ports & Services (Confusion Point ⚠️)
• **Port 445 (`microsoft-ds`):** SMB (File Sharing).
  • Recall: In the mock interview, you thought this was "Directory Services." It is not. It is for accessing shared files/printers.

• **Port 389:** LDAP (Lightweight Directory Access Protocol).
  • This is the actual "Directory Service" port used to query users and groups.

• **Port 88:** Kerberos. The authentication service.

## 4. Authentication Protocols
• **NTLM (Legacy):** Uses Challenge-Response.
  • Analogy: The "Banana" trick. Server challenges you; you prove you know the password hash without sending it.

• **Kerberos (Modern):** Uses Tickets.
  • Analogy: The "Carnival." You get a TGT (Wristband) from the KDC, then exchange it for Service Tickets (Ride Tickets).

• **Zero Knowledge Proof (Confusion Point ⚠️):**
  • Recall: You asked if the server knows your password. NO. The server only validates the Hash (NTLM) or the Ticket (Kerberos). It never sees the plain password.

## 5. The Crown Jewels: Post-Exploitation (Confusion Point ⚠️)
• **LSASS.exe (Local Security Authority Subsystem Service):**
  • The "Security Guard" process in Windows.
  • Stores credentials (Hashes, Tickets) in RAM to allow Single Sign-On.
  • Goal: Dump the memory of this process.

• **Mimikatz:**
  • The tool used to dump LSASS.
  • Metasploit command: `load kiwi`.
  • Recall: You missed these names in the interview. Memorize LSASS (Process) and Mimikatz (Tool).

## 6. Common Attacks Cheat Sheet