# BitPay Tags – Bitcoin-Native Payment Request Protocol

**Version:** `v1.0.0`
**Language:** Clarity (Stacks Smart Contracts)
**Author:** \[Stacks Developer - Senior Contributor]
**Contract Name:** `bitpay-tags.clar`
**Network:** Stacks Mainnet/Testnet
**Token Used:** sBTC (Stacks-wrapped Bitcoin)

---

## 📌 Summary

**BitPay Tags** is a decentralized, trust-minimized payment request protocol built on the **Stacks blockchain**, leveraging **sBTC** to enable Bitcoin-native settlement. The protocol allows users to issue timestamped, shareable payment requests ("tags") that can be fulfilled by any user in a trustless, on-chain manner.

---

## 🚀 System Overview

BitPay Tags streamlines peer-to-peer, freelance, and merchant payments using the Stacks blockchain:

* A **creator** generates a payment tag with an amount, recipient, memo, and expiration.
* A **payer** (typically the recipient) fulfills the tag by transferring sBTC to the recipient via the contract.
* The system tracks **state transitions** (pending → paid, expired, or canceled).
* Indexes allow users to easily query payment requests by creator or recipient.
* Events provide real-time payment observability.

This model reduces reliance on custodial solutions or traditional invoices and promotes interoperable, open financial primitives for Bitcoin-based economies.

---

## 📐 Contract Architecture

### ▶️ Core Components

| Component         | Description                                                      |
| ----------------- | ---------------------------------------------------------------- |
| `payment-tags`    | Main storage map for each tag (id-based)                         |
| `creator-index`   | Maps creator to a list of created tag IDs for efficient querying |
| `recipient-index` | Maps recipient to tag IDs where they are the beneficiary         |
| `contract-stats`  | Keyed metrics (e.g., tags-created, tags-fulfilled)               |
| `tag-counter`     | Global counter to assign unique tag IDs                          |
| `contract-paused` | Flag for emergency pause control                                 |

### ▶️ Tag States

| State      | Meaning                                  |
| ---------- | ---------------------------------------- |
| `pending`  | Active, awaiting fulfillment             |
| `paid`     | Fulfilled successfully                   |
| `expired`  | Expired before payment                   |
| `canceled` | Canceled by creator (before fulfillment) |

---

## 🔁 Data Flow

```mermaid
flowchart TD
  A[Create Payment Tag] -->|tx-sender| B[Stores in payment-tags map]
  B --> C[Update creator-index & recipient-index]
  C --> D[Emit "payment-tag-created"]

  E[Fulfill Payment Tag] -->|tx-sender| F[Check tag validity & expiration]
  F --> G[Call sBTC transfer]
  G --> H[Update tag state to PAID]
  H --> I[Emit "payment-tag-fulfilled"]

  J[Expire Payment Tag] -->|any user| K[Verify expiration]
  K --> L[Set state to EXPIRED]
  L --> M[Emit "payment-tag-expired"]

  N[Cancel Payment Tag] -->|creator only| O[Set state to CANCELED]
  O --> P[Emit "payment-tag-canceled"]
```

---

## 🧠 Features

* ✅ **Time-bound payment requests** with auto-expiry
* ✅ **Decentralized fulfillment** using sBTC token
* ✅ **Stateful transitions**: `pending`, `paid`, `expired`, `canceled`
* ✅ **Anti-spam**: enforces minimum amount & memo constraints
* ✅ **Indexing** for both creator and recipient queries
* ✅ **Event logs** for off-chain tracking and integration
* ✅ **Emergency pause control** (admin)
* ✅ **No double payment** or self-payments allowed

---

## 📦 Key Public Functions

### 🏷️ `create-payment-tag(...)`

Creates a new payment request.

* Validates amount, expiration, recipient
* Requires non-empty memo (if provided)
* Emits: `payment-tag-created`

---

### 💸 `fulfill-payment-tag(tag-id)`

Transfers sBTC to the recipient and marks tag as paid.

* Verifies pending state and expiry
* Emits: `payment-tag-fulfilled`

---

### ⛔ `cancel-payment-tag(tag-id)`

Allows creator to cancel unpaid tag.

* Only creator can cancel
* Emits: `payment-tag-canceled`

---

### ⏳ `expire-payment-tag(tag-id)`

Any user can expire a tag after its expiration block if still unpaid.

* Emits: `payment-tag-expired`

---

### 🛑 `toggle-contract-pause()`

Admin-only function to pause/resume contract.

---

## 🔍 Query Functions

| Function                        | Description                                    |
| ------------------------------- | ---------------------------------------------- |
| `get-payment-tag(tag-id)`       | Fetch detailed info for a specific tag         |
| `get-creator-tags(principal)`   | Returns list of tag IDs created by user        |
| `get-recipient-tags(principal)` | Returns list of tag IDs assigned to user       |
| `get-contract-info()`           | Returns metadata about the contract            |
| `get-contract-stats(key)`       | Gets internal metrics (e.g., `tags-fulfilled`) |
| `can-expire-tag(tag-id)`        | Checks if a tag is eligible for expiration     |
| `get-tag-counter()`             | Returns latest tag ID                          |
| `is-contract-paused()`          | Returns contract paused status                 |
| `get-multiple-tags([...])`      | Batch fetch multiple tags                      |

---

## ⚙️ Constants

| Constant                | Value   | Description                               |
| ----------------------- | ------- | ----------------------------------------- |
| `MIN-PAYMENT-AMOUNT`    | `u1000` | Prevents dust/spam tags (0.00001 sBTC)    |
| `MAX-EXPIRATION-BLOCKS` | `u4320` | Max expiration (\~30 days at 10min/block) |
| `MAX-TAGS-PER-USER`     | `u100`  | Limits tags per user in index             |

---

## 📦 Dependencies

* **sBTC Contract**: Transfer calls depend on the sBTC contract (update `SBTC-CONTRACT` with mainnet address).
* **Stacks Blockchain**: Relies on `stacks-block-height` for timing/expiration logic.

---

## 🔐 Security & Safeguards

* ✔️ Prevents self-payment (`tx-sender != recipient`)
* ✔️ Tag state transitions are enforced strictly
* ✔️ Cannot fulfill expired/canceled/paid tags
* ✔️ Admin pause function for incident response
* ✔️ Access-controlled cancellation (creator-only)

---

## 🧪 Testing Recommendations

* Ensure unit tests cover:

  * Fulfillment failure on expired/canceled tags
  * Accurate state transitions across lifecycle
  * Index correctness after tag creation
  * Double fulfillment attempts are rejected

---

## 🧾 Events

| Event Name               | Trigger Function        | Purpose                        |
| ------------------------ | ----------------------- | ------------------------------ |
| `payment-tag-created`    | `create-payment-tag`    | On new tag creation            |
| `payment-tag-fulfilled`  | `fulfill-payment-tag`   | On successful payment          |
| `payment-tag-canceled`   | `cancel-payment-tag`    | On tag cancellation by creator |
| `payment-tag-expired`    | `expire-payment-tag`    | On manual expiration           |
| `contract-pause-toggled` | `toggle-contract-pause` | On admin pause/unpause         |

---

## 📘 License

MIT or a permissive license of your choosing.

---

## ✉️ Contact & Contributions

Open for contributions and integrations.
To report bugs, suggest features, or request support for extending to Lightning/sats-denominated tags, open an issue or contact the core dev team.
