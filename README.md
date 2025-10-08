# 📜 ProofLock

**Immutable Document Registry Secured by Bitcoin's Proof-of-Work**

ProofLock is a tamper-proof, verifiable, and privacy-preserving attestation registry for digital documents, built on the Stacks blockchain and anchored by Bitcoin’s Proof-of-Work. It enables individuals and organizations to timestamp, validate, and audit content hashes without revealing the underlying data — ensuring integrity, transparency, and long-term verifiability.

---

## 🚀 System Overview

ProofLock allows users to register document attestations (by content hash) and assign them to recipients, creating immutable records on-chain. Each attestation links:

* The **issuer** (creator),
* The **subject** (recipient),
* The **document hash** (SHA-256, 32 bytes),
* The **block height** of registration.

Attestations can later be **verified** by re-submitting the document hash. Verification updates the record and maintains a count of successful verifications — all without exposing any private document data.

---

## 🏛️ Contract Architecture

### 📄 Core Components

| Component                 | Description                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------- |
| `create-attestation`      | Registers a new attestation, linking a document hash to a subject.                      |
| `verify-attestation`      | Validates a submitted hash against the stored record and updates verification metadata. |
| `get-attestation`         | Retrieves full details for a given attestation ID.                                      |
| `get-user-stats`          | Tracks the number of attestations created by each user.                                 |
| `hash-exists`             | Checks if a hash has been registered previously.                                        |
| `get-verification-count`  | Returns how many times a given hash has been verified.                                  |
| `update-protocol-version` | Admin-only function to update protocol version.                                         |

### 🧱 Data Model

#### 📚 `attestations` Map

Stores all attestations by a unique auto-incremented ID.

```clojure
{
  attestation-id: uint => {
    issuer: principal,
    subject: principal,
    content-hash: (buff 32),
    created-at: uint,
    block-height: uint,
    is-verified: bool
  }
}
```

#### 📊 `user-stats` Map

Tracks the number of attestations issued by a user.

```clojure
{ user: principal } => { total-attestations: uint }
```

#### 🔍 `hash-index` Map

Enables reverse lookup by hash and tracks verification attempts.

```clojure
{ content-hash: (buff 32) } => {
  attestation-id: uint,
  verification-count: uint
}
```

#### 🔐 State Variables

* `attestation-counter`: Global attestation count.
* `protocol-version`: Protocol version, adjustable by admin.
* `CONTRACT_OWNER`: Hardcoded to the transaction sender at deployment.

---

## 🔁 Data Flow

### Attestation Creation

1. **Input:** Issuer provides `subject` and `content-hash`.
2. **Validation:**

   * Subject is a valid principal (not burn address or self).
   * Content hash is exactly 32 bytes.
3. **Storage:**

   * New record is inserted into `attestations` and `hash-index`.
   * `attestation-counter` is incremented.
   * Issuer's `user-stats` is updated.

### Attestation Verification

1. **Input:** Caller submits `attestation-id` and `provided-hash`.
2. **Validation:**

   * Attestation must exist.
   * Hash must match the stored hash.
3. **Outcome:**

   * Marks the attestation as verified (if match).
   * Increments `verification-count` in `hash-index`.

---

## 🛡️ Privacy & Security

* 🔐 **Zero-Knowledge Privacy**: Only cryptographic hashes are stored on-chain.
* 📦 **Immutable Storage**: Leverages Clarity’s immutability and Bitcoin finality via Stacks.
* 🚫 **Burn Address Guard**: Ensures attestations can’t be issued to invalid principals.
* 🧪 **Input Sanitization**: Validates hash length, attestation ID, and addresses.
* 🔐 **Admin Gating**: Only the contract deployer can update protocol version.

---

## ⚙️ Public Interface Summary

| Function                  | Type      | Access     | Description                                   |
| ------------------------- | --------- | ---------- | --------------------------------------------- |
| `create-attestation`      | Public    | Anyone     | Registers a document attestation.             |
| `verify-attestation`      | Public    | Anyone     | Verifies an existing attestation by hash.     |
| `update-protocol-version` | Public    | Owner only | Admin function to increment protocol version. |
| `get-attestation`         | Read-only | Anyone     | Fetches attestation record by ID.             |
| `get-user-stats`          | Read-only | Anyone     | Returns total attestations by a user.         |
| `get-total-attestations`  | Read-only | Anyone     | Returns global attestation count.             |
| `get-protocol-version`    | Read-only | Anyone     | Returns current protocol version.             |
| `hash-exists`             | Read-only | Anyone     | Checks whether a hash is registered.          |
| `get-verification-count`  | Read-only | Anyone     | Gets how many times a hash has been verified. |

---

## 📦 Deployment & Upgrade Notes

* **Contract Owner** is set to `tx-sender` at deployment. This address is required to call `update-protocol-version`.
* All attestations and user stats are permanently stored on-chain; upgrades should preserve this state for continuity.
* Recommended to wrap external integrations (e.g., Web3 UI) with hash pre-checks using `hash-exists` for UX optimization.

---

## ✅ Example Use Cases

* **Proof of Document Creation:** Timestamp legal agreements, IP claims, or original works.
* **Audit Trails:** Register and verify operational records or logs.
* **Confidential Verification:** Validate data ownership without revealing contents.

---

## 📜 License

This smart contract is open-source and provided without warranty. Use at your own risk. Contributions welcome.
