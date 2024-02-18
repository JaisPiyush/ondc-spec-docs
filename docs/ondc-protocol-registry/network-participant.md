---
sidebar_position: 2
---

# Network Participant
1. There are 3 possible participant types, for each domain:
    * Buyer App (BAP)
    * Seller App - Inventory seller node (BPP - ISN)
    * Seller App - Marketplace seller node or aggregator (BPP -MSN)

2. Each participant type defined above, will have entry in registry for their entity. Additionally, MSN(aggregator) seller apps will also have **1 entry for each of their seller-on-record (merchant)**.

3. Every participant on the registry will have 1 or more entries in the registry, with `ukid` for each such entry.

4. Signing & Verification of requests and responses using auth header is defined [here](/docs/ondc-protocol-v1.1-retail/signing-verification)
