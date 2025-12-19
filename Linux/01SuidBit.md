# Linux Permissions & SUID Mastery

## 1. Standard Permissions (rwx)
Permissions are grouped into three sets of three characters: **User (Owner)**, **Group**, and **Others**.

*   **Read (r):** View file contents.
*   **Write (w):** Modify file contents.
*   **Execute (x):** Run the file as a program.
*   **No Permission (-):** Placeholder meaning "access denied" for that slot.

### The Structure (The "Rule of Three")
A permission string is 10 characters long. Example: `drwxr-xr--`

1.  **File Type (1 char):** `d` (directory) or `-` (file).
2.  **User (3 chars):** `rwx` (Owner has full control).
3.  **Group (3 chars):** `r-x` (Group has Read + Execute).
4.  **Others (3 chars):** `r--` (Everyone else has Read only).

**Key Clarification:** There are no delimiters (hyphens) separating the groups. The hyphens you see are part of the permissions themselves (indicating a missing permission).

---

## 2. Numeric Notation (Octal)
We use numbers to change permissions (e.g., `chmod`).

*   **r** = 4
*   **w** = 2
*   **x** = 1
*   **-** = 0

**Calculation:** Sum the values for each group.
*   `rwx` = 4 + 2 + 1 = **7**
*   `r-x` = 4 + 0 + 1 = **5**
*   `r--` = 4 + 0 + 0 = **4**

**Result Command:** `chmod 754 filename`

---

## 3. SUID (Set User ID)
*   **Indicator:** An `s` replaces the `x` in the User section (e.g., `-rwsr-xr-x`).
*   **Concept:** When executed, the program runs with the permissions of the **Owner** (usually root), not the current user.
*   **Security Risk:** If a standard user can run a SUID binary (like Python or Vim), they can potentially spawn a shell. Since the binary runs as root, the shell will also be root. This is a common **Privilege Escalation** vector.