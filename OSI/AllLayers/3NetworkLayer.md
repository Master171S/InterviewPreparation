### 📝 Personalized Notes: Layer 3 (Network Layer)

#### The Core Concept:
*   **The "Navigator" Layer:** While Layer 4 handles the delivery to the specific app, Layer 3 handles the **journey across the internet** to get to the right building (Computer).
*   **Role:** It connects different networks together. It is responsible for **Logical Addressing** and **Path Selection (Routing)**.

***

#### Key Jobs (The "Navigation" Tasks):

*   **Logical Addressing (IP Addresses):**
    *   Assigns a unique ID to every device so it can be located globally.
    *   **Structure:** It works like a street address (City + House Number) → (**Network ID** + **Host ID**).

*   **Routing (Path Selection):**
    *   **Routers** talk to each other to map the world. They decide the **best path** for a packet to travel from Source A to Destination B, hopping across many different networks.

*   **Encapsulation (Packets):**
    *   Takes the Segment from Layer 4, adds **IP headers** (Source & Destination IP), and creates a **Packet**.

---

### 💡 Your Specific "Aha!" Moments & Clarifications:

#### **Confusion Point 1: What happens if a router gets confused?**
*   **The Insight:** **Time to Live (TTL).** You learned that every packet has a self-destruct mechanism. It’s not a timer in seconds, but a **"hop counter."** Every time a packet passes through a router, the TTL number goes down by 1. If it hits 0, the packet is killed. This prevents **"zombie packets"** from spinning in loops forever.

#### **Confusion Point 2: Router vs. Switch**
*   **The Insight:** **Inter-Network vs. Intra-Network.**
    *   **Routers (Layer 3)** connect **different networks** (e.g., Your Home $\leftrightarrow$ The Internet).
    *   **Switches (Layer 2)** connect devices **within the same network** (e.g., Your Laptop $\leftrightarrow$ Your Printer).