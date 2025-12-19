# Linux File System & Text Processing Study Guide

## 1. Critical File System Hierarchy
Understanding where sensitive data lives and where you can write files is crucial for exploitation.

### The Password Files

**`/etc/passwd`**
*   **Permissions:** World-readable (`rw-r--r--`).
*   **Content:** Usernames, UIDs, Home Directories, Shells.
*   **Attacker Value:** Used for user enumeration (finding valid usernames to attack).

**`/etc/shadow`**
*   **Permissions:** Root-readable only (`rw-r-----`).
*   **Content:** Password hashes.
*   **Attacker Value:** If read, hashes can be cracked offline using tools like John the Ripper or Rainbow Tables.

### The Temporary Directory (`/tmp`)
*   **Permissions:** World-writable (`drwxrwxrwt`).
*   **The Sticky Bit (t):** The `t` at the end ensures that while anyone can write files, only the file owner (or root) can delete them.
*   **Attacker Value:** This is the standard "drop zone" for attackers. If you get code execution but have no privileges, you upload your exploit scripts here because it is guaranteed to be writable.

---

## 2. Text Processing & Redirection
Navigating large datasets in the terminal requires understanding data streams.

### Piping (`|`)
Used to pass the output of one command as input to another.
*   **Example:** `cat network_scan.txt | grep open` (Finds all lines containing "open").

### Standard Data Streams & Redirection
Linux uses three standard streams, each with a numeric ID:
*   **0 (Stdin):** Standard Input (Keyboard/Data coming in).
*   **1 (Stdout):** Standard Output (The normal results you see on screen).
*   **2 (Stderr):** Standard Error (Error messages like "Permission denied").

### Hiding Errors (`2>/dev/null`)
To filter out error messages (like when scanning the whole system), you redirect the Error Stream (#2) to `/dev/null` (the system blackhole).
*   **Syntax:** `command 2>/dev/null`
*   **Meaning:** "Take stream #2 and send it to nowhere."