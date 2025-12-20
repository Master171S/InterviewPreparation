# Insecure Direct Object Reference (IDOR)

A critical access control vulnerability where an application uses user-supplied input to access objects directly without sufficient authorization checks.

---

## 1. The Concept
IDOR occurs when an application exposes a reference to an internal implementation object without validating that the user is authorized to access it. These objects can include:
*   Database keys
*   Files
*   Directories
*   User accounts

---

## 2. The "Sequential Mistake"
A common scenario involves an attacker changing a URL parameter to access different records.
*   **Example:** Changing `?id=1005` ➡️ `?id=1006`
*   If the application returns the data for user 1006 to user 1005, an IDOR vulnerability exists.

---

## 3. The Core Failure
IDOR is a failure of **Authorization**, not **Authentication**.

*   **Authentication:** The server successfully identifies who you are. ✅
*   **Authorization:** The server fails to check if you actually have permission to view or modify the specific data you requested. ❌

---

## 4. The Remediation

### Access Control Checks
Verify permissions for the specific object at the code level every time a request is made. The application must confirm that the current session user owns or is permitted to see the requested ID.

### Indirect References
*   **UUIDs:** Use non-guessable, complex identifiers (like `550e8400-e29b-41d4-a716-446655440000`) instead of sequential integers.
*   **Session-based Maps:** Use temporary, per-session maps to refer to objects so that IDs are not exposed directly to the user.