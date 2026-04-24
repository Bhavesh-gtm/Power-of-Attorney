# 🔐 Delegated Authority — Soroban Smart Contract

> On-chain, expiry-enforced authority delegation built on the **Stellar** network with **Soroban**.

---

## Project Description

**Delegated Authority** is a Soroban smart contract that lets any Stellar account grant a scoped, time-limited authority to another account — and records that delegation immutably on-chain.

Think of it as a programmable power-of-attorney system for web3: a principal grants a delegate the right to act on their behalf for a specific purpose (a *scope*), but only until a chosen ledger height. When that ledger passes — or if the principal revokes early — the authority is gone, automatically, with no off-chain coordination required.

---

## What It Does

| Action | Who calls it | What happens |
|---|---|---|
| `delegate(delegator, delegate, scope, expires_at)` | Delegator | Records a new delegation scoped to a `Symbol` (e.g. `"vote"`, `"transfer"`) with a ledger-based deadline |
| `revoke(delegator, delegate, scope)` | Delegator | Marks the delegation as revoked before natural expiry |
| `is_active(delegator, delegate, scope)` | Anyone | Returns `true` only if the record exists, is not revoked, and hasn't expired |
| `get_delegation(delegator, delegate, scope)` | Anyone | Returns the full `Delegation` struct for off-chain inspection |
| `list_delegations(delegator)` | Anyone | Returns all `(delegate, scope)` pairs registered under a delegator |

Expired and revoked records remain readable on-chain forever — giving you a complete, auditable history of every authority ever granted or withdrawn.

---

## Features

### ⏱ Ledger-Native Expiration
Expiry is expressed as a **Soroban ledger sequence number** — the same clock the network uses internally. No timestamps, no oracles, no drift. Once `env.ledger().sequence() > expires_at`, `is_active` returns `false`, period.

### 🔑 Scope-Based Granularity
Each delegation targets a named **scope** (`Symbol`), so a single delegator can grant narrow, non-overlapping authorities:
```
delegator → delegate_A  scope="vote"      expires_at=500_000
delegator → delegate_B  scope="transfer"  expires_at=600_000
delegator → delegate_B  scope="admin"     expires_at=450_000
```
Revoking one scope leaves the others untouched.

### 🛡 Auth-Enforced Writes
All state-changing calls (`delegate`, `revoke`) require `delegator.require_auth()`, which Soroban resolves against Stellar's native signature / multisig rules. No custom auth logic to audit.

### 📋 On-Chain Index
A `DelegatorIndex` entry tracks every `(delegate, scope)` pair per delegator, so `list_delegations` is a single storage read — no log-scraping required.

### 📡 Events
Every successful delegation or revocation emits a Soroban event:
- `(DELEGATED, delegator, delegate)` → `(scope, expires_at)`
- `(REVOKED, delegator, delegate)` → `scope`

Horizon and RPC subscribers can index these for dashboards and notifications.

### 🚫 Safety Guards
- **Self-delegation rejected** — you cannot delegate to yourself.
- **Past expiry rejected** — `expires_at` must be strictly greater than the current ledger.
- **Double-revoke rejected** — revoking an already-revoked record panics cleanly.

### 🧪 Test Suite
Six unit tests ship with the contract covering the happy path, revocation, expiry fast-forward, past-expiry rejection, and self-delegation rejection — all runnable with a single command (see below).

---

## Project Structure

```
delegated-authority/
├── Cargo.toml          # Soroban SDK dependency + release profile
└── src/
    └── lib.rs          # Contract, data types, events, tests
```

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| Rust | stable | `rustup default stable` |
| `wasm32-unknown-unknown` target | — | `rustup target add wasm32-unknown-unknown` |
| Stellar CLI | ≥ 21.x | [install guide](https://developers.stellar.org/docs/tools/stellar-cli) |

---

## Getting Started

### 1 — Clone & enter the project
```bash
git clone https://github.com/your-org/delegated-authority.git
cd delegated-authority
```

### 2 — Run the tests
```bash
cargo test
```

Expected output:
```
running 6 tests
test tests::test_delegate_and_check_active ... ok
test tests::test_revoke ... ok
test tests::test_expired_delegation ... ok
test tests::test_list_delegations ... ok
test tests::test_past_expiry_rejected ... ok
test tests::test_self_delegation_rejected ... ok
```

### 3 — Build the WASM
```bash
cargo build --target wasm32-unknown-unknown --release
```

The optimised contract lands at:
```
target/wasm32-unknown-unknown/release/delegated_authority.wasm
```

### 4 — Deploy to Testnet
```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/delegated_authority.wasm \
  --source <YOUR_KEYPAIR_NAME> \
  --network testnet
```

Copy the returned `CONTRACT_ID` for the invocations below.

---

## Usage Examples

### Grant delegation
```bash
stellar contract invoke \
  --id $CONTRACT_ID \
  --source alice \
  --network testnet \
  -- delegate \
  --delegator $(stellar keys address alice) \
  --delegate  $(stellar keys address bob) \
  --scope     vote \
  --expires_at 5000000
```

### Check if active
```bash
stellar contract invoke \
  --id $CONTRACT_ID \
  --network testnet \
  -- is_active \
  --delegator $(stellar keys address alice) \
  --delegate  $(stellar keys address bob) \
  --scope     vote
```

### Revoke
```bash
stellar contract invoke \
  --id $CONTRACT_ID \
  --source alice \
  --network testnet \
  -- revoke \
  --delegator $(stellar keys address alice) \
  --delegate  $(stellar keys address bob) \
  --scope     vote
```

---

## Data Model

```rust
pub struct Delegation {
    pub delegator:  Address,   // Granting account
    pub delegate:   Address,   // Receiving account
    pub scope:      Symbol,    // Named authority (≤ 9 chars)
    pub expires_at: u32,       // Ledger sequence number deadline
    pub revoked:    bool,      // Explicit revocation flag
}
```

---

## Roadmap

- [ ] Multi-sig delegator support (threshold > 1)
- [ ] Renewable delegations (extend expiry)
- [ ] Sub-delegation (delegate → sub-delegate with narrowed scope)
- [ ] Contract-level administrator for emergency pause

---

## License

MIT — see `LICENSE` for details.



<img width="1493" height="636" alt="Screenshot From 2026-04-24 12-05-05" src="https://github.com/user-attachments/assets/02564532-85a0-42b6-b47b-1ff9bb7ad683" />
\mod test;

wallet adress: GDGZCFKO2MW2PTOI2CH6RM3DCLDWUG65ZHXUZPCVEWDKV3C5IPQROIII

contract adress: CDQAYEJONHVNIOCW4ZIGPDONYMBJOOM3QZDXPRZJR6GJ5LJVPRLWQYJM
https://stellar.expert/explorer/testnet/contract/CDQAYEJONHVNIOCW4ZIGPDONYMBJOOM3QZDXPRZJR6GJ5LJVPRLWQYJM




