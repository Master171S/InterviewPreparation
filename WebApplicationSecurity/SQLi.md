# Web Application Security: SQL Injection (SQLi) Master Note

A comprehensive guide to understanding SQLi types, their operational impact during an attack, and modern defense mechanisms.

---

## 1. Types of SQL Injection (SQLi)

### In-band SQLi (Classic)
The most common type. It uses the same communication channel for both the attack and the results.
*   **Error-based SQLi:** Exploits database error messages (e.g., "Syntax error in query...") to reveal information about the database structure.
*   **Union-based SQLi:** Uses the `UNION` operator to combine the results of the original query with results from a payload query, allowing the attacker to retrieve data from other tables.

### Inferential (Blind) SQLi
The server does not return data directly (no error messages or data dumps), so attackers must infer information.
*   **Boolean-based:** Sends payloads that result in a `TRUE` or `FALSE` condition. The attacker observes differences in the page content or HTTP response size to deduce data.
*   **Time-based:** Sends payloads that tell the database to wait (e.g., `SLEEP(5)`) if a condition is true. The attacker measures the time it takes for the server to respond.

### Out-of-band SQLi
Used when the attacker cannot use the same channel to launch the attack and gather results (e.g., when the application suppresses all output).
*   **Mechanism:** Relies on the database making external calls (like DNS or HTTP requests) to a server controlled by the attacker to exfiltrate data.

---

## 2. Why is Blind SQLi "Noisy" and "Slow"? (Interview Crucials)

### "Noisy" = Logs
When you say "noisy," interviewers specifically want to hear about **server logs**.
*   **Comparison:** In a classic SQLi, one request could dump the whole database. In Blind SQLi, you might send 50,000 requests just to guess one administrator password character by character.
*   **The Result:** The system administrator checks the `access.log` and sees thousands of weird requests from a single IP address. It lights up security monitoring tools (like a SIEM) like a Christmas tree 🎄.

### "Slow" = Network Latency
*   You have to wait for the server to respond (or sleep) for every single guess. Guessing a 20-character hash one bit at a time over a slow connection can take hours.

---

## 3. The Defense (Remediation)

### Primary Fix: Prepared Statements (Parameterized Queries)
*   **The Core Concept: Separation of Code and Data.**
*   The database receives the SQL code template first, and the user input is treated strictly as **data** later. They never mix, so the input cannot be executed as a command.

### Secondary Fix: Input Sanitization
*   Attempts to filter out "bad" characters (like `'` or `OR`).
*   **Note:** This is a secondary defense layer, not a cure, as it can often be bypassed with clever encoding.

---

## 4. Real-World Context
*   **Sony Pictures Hack:** A classic example of SQLi where a lack of validation allowed attackers to dump sensitive user tables directly to the browser.