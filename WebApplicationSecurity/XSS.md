# Cross-Site Scripting (XSS) Fundamentals

A guide to client-side injection attacks, focusing on execution types, contextual defenses, and the security features of modern web frameworks.

---

## 1. The Core Concept: Attacking the User
*   **Mechanism:** Injecting malicious JavaScript that executes in the victim's browser.
*   **Goal:** Steal session cookies, perform actions on behalf of the user, or redirect them to malicious sites.
*   **The Big Distinction:** Unlike SQLi (which attacks the backend database), **XSS attacks the client** (the user's browser).

---

## 2. Main Types of XSS

### Reflected XSS (Non-Persistent)
*   **How:** The payload is part of the request (e.g., in a URL parameter or search box) and is immediately returned ("reflected") by the server in the response.
*   **Attack Vector:** Usually requires **Social Engineering**, such as tricking a user into clicking a specially crafted link.

### Stored XSS (Persistent)
*   **How:** The payload is stored in the server's database (e.g., in a comment section, user bio, or forum post) and served to anyone who views that specific page.
*   **Why it's Dangerous:** No specific link is needed; the trap is waiting on a legitimate page for any unsuspecting user.

---

## 3. The Defense: Context-Aware Output Encoding
*   **The Goal:** Treat user input strictly as **Text**, not as executable **Code**.
*   **Mechanism:** Convert special characters into safe HTML entities (e.g., `<` becomes `&lt;`).

### Why "Context-Aware"?
Different parts of a web page require different encoding rules:
*   **HTML Body:** Encode `<` `>` `&` `"` `'`.
*   **Attributes:** Must prevent attackers from "breaking out" of attribute quotes.
*   **JavaScript:** HTML entities don't work inside script blocks; this requires Unicode escapes.
*   **Danger Zone:** Putting untrusted input directly into "sinks" like `href` (risk of `javascript:` protocol) or `<script>` tags.

---

## 4. Modern Frameworks (React, Angular, Vue)
*   **Default Behavior:** These frameworks automatically encode data before rendering it to the DOM, making them "secure by default" against most common XSS vectors.
*   **The Escape Hatch:** Developers can intentionally bypass this protection if they need to render raw HTML.
*   **React Specific Property:** `dangerouslySetInnerHTML`
    *   **Significance:** The name itself is a security warning. It indicates the developer is manually bypassing XSS protection. This is a primary target for security auditors during a code review.