### 📝 Personalized Notes: Layer 6 (Presentation Layer)

#### The Core Concept:
*   **The "Translator" Layer:** While Layer 7 deals with *what* you want to send, Layer 6 deals with *how it looks* (syntax and formatting).
*   **Role:** It ensures that the data sent by the application is readable by the system. It **formats, encrypts, and compresses** data so it can travel efficiently and safely.

***

#### Key Jobs (The "Formatting" Tasks):
*   **Translation:** Converting data between different formats (e.g., ASCII to EBCDIC) so two different computers can understand each other.
*   **Encryption/Decryption:** Scrambling data (like your bank password) into unreadable code (`&%#@*!`) so it’s safe on the wire, and unlocking it at the other end.
*   **Compression/Decompression:** Shrinking large data (like a raw movie) into a manageable size for transmission, and rebuilding it for viewing.

---

### 💡 Your Specific "Aha!" Moments & Clarifications:

#### **Confusion Point 1: "Is .mp4 just a name?"**
*   **The Insight:** No. You learned that **.mp4 isn't just a label**; it represents a **compression algorithm (codec)**. Layer 6 takes a massive "Raw" video file (which saves every pixel of every frame) and uses math to throw away redundant information (like a static blue background), turning a 3,000 GB file into a 2 GB stream.

#### **Confusion Point 2: The "Sticky Note" (SNI)**
*   **The Insight:** We discussed how the server needs to know which website you want (e.g., `bank.com`) before it can encrypt the connection. You correctly identified that this **"Server Name Indication" (SNI) is sent in plain text during the handshake**, meaning a hacker (or ISP) can see *which* site you are visiting, even if they can't read the content.