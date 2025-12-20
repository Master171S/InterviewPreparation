# Web Application Security: The "Big 4" Cheat Sheet

A high-level summary of the most critical web vulnerabilities, their primary targets, and essential remediation strategies.

---

## 1. SQL Injection (SQLi) 💉
*   **Target:** The Database.
*   **Concept:** The attacker manipulates SQL queries to view, modify, or delete unauthorized data.
*   **Types:**
    *   **In-Band (Classic):** Results and error messages are returned directly in the browser/response.
    *   **Blind:** No data is returned directly; the attacker infers data through True/False questions (**Boolean-based**) or response delays (**Time-based**).
*   **The Fix:** **Prepared Statements (Parameterized Queries).** This ensures the database treats user input strictly as data, never as executable code.

---

## 2. Cross-Site Scripting (XSS) 🎭
*   **Target:** The User (Client-Side).
*   **Concept:** Injecting malicious JavaScript that executes within the victim's browser session.
*   **Types:**
    *   **Reflected:** The payload is included in a link or request; requires social engineering to execute.
    *   **Stored:** The payload is saved permanently in the database (e.g., a comment section) and attacks anyone who visits the page.
*   **The Fix:** **Context-Aware Output Encoding.** Convert special characters (like `<`, `>`, `"`) into safe HTML entities before rendering them to the page.

---

## 3. Insecure Direct Object Reference (IDOR) 🔑
*   **Target:** Authorization / Access Control.
*   **Concept:** Often called the "Sequential Mistake." Accessing unauthorized objects (files, records, accounts) by simply changing an ID parameter (e.g., `id=100` ➡️ `id=101`).
*   **Core Failure:** **Authentication** succeeds (the server knows who you are), but **Authorization** fails (the server doesn't check if you have the right to see the specific data requested).
*   **The Fix:** Implement strict **Access Control Checks** on every request and use non-guessable identifiers (like UUIDs).

---

## 4. Command Injection (RCE) 💻
*   **Target:** The Operating System.
*   **Concept:** Passing unsafe user input directly to a system shell, allowing the execution of OS-level commands (e.g., `whoami`, `cat /etc/passwd`).
*   **Mechanics:** Using shell separators like `;` (run regardless of success) or `|` (pipe) to chain malicious commands onto legitimate ones.
*   **The Fix:** Avoid dangerous functions like `os.system()`. Use secure libraries that accept arguments as a **List** (e.g., Python's `subprocess.run(['cmd', 'arg'])`) so the shell never interprets the input as code.