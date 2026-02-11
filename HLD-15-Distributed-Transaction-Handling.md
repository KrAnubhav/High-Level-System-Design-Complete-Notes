# HLD-15: Distributed Transaction Handling

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is Transaction](#what-is-transaction)
3. [ACID Properties](#acid-properties)
4. [Problem: Distributed Transactions](#problem-distributed-transactions)
5. [Solution 1: Two-Phase Commit (2PC)](#solution-1-two-phase-commit-2pc)
6. [Solution 2: Three-Phase Commit (3PC)](#solution-2-three-phase-commit-3pc)
7. [Solution 3: Saga Pattern](#solution-3-saga-pattern)
8. [Summary](#summary)
9. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Distributed Transaction Handling

**Importance:**
- Asked in interviews (2+ years experience)
- Daily software engineering work
- Critical for microservices architecture

**Coverage:**
- ✅ What is transaction & ACID properties
- ✅ Problem with distributed transactions
- ✅ Two-Phase Commit (2PC)
- ✅ Three-Phase Commit (3PC)
- ✅ Saga Pattern

---

## What is Transaction

### Definition

```
Transaction = Set of operations performed against DB
            = Group of tasks executed together
```

**Example:**

```
Database:
A: ₹100
B: ₹50

Transaction: Transfer ₹100 from A to B

Operations:
1. Debit A: ₹100
2. Credit B: ₹100

Result:
A: ₹0
B: ₹150
```

**Visual:**

```
┌─────────────────────────────┐
│      Transaction            │
├─────────────────────────────┤
│ BEGIN TRANSACTION           │
│                             │
│ 1. UPDATE A SET balance =   │
│    balance - 100            │
│                             │
│ 2. UPDATE B SET balance =   │
│    balance + 100            │
│                             │
│ COMMIT                      │
└─────────────────────────────┘
```

---

## ACID Properties

### 1. Atomicity

```
All operations succeed OR all fail
No partial success
```

**Example:**

```
Transaction:
1. Debit A: ₹100 ✓ (Success)
2. Credit B: ₹100 ✗ (Failed)

Atomicity:
→ Rollback operation 1
→ A remains ₹100
→ All or nothing
```

---

### 2. Consistency

```
DB in consistent state before and after transaction
```

**Example:**

```
Before Transaction:
A: ₹100, B: ₹50
Total: ₹150 ✓

After Transaction (Success):
A: ₹0, B: ₹150
Total: ₹150 ✓

Consistency ensures total remains same
```

---

### 3. Isolation

```
Concurrent transactions appear sequential
```

---

### 4. Durability

```
After successful commit, data persists
Even if DB crashes
```

---

## Problem: Distributed Transactions

### Single Database Transaction

```
┌─────────────────────────────┐
│      Database T             │
├─────────────────────────────┤
│ A: ₹200                     │
│ B: ₹100                     │
└─────────────────────────────┘

BEGIN TRANSACTION ON T:
1. Withdraw from A: ₹100 ✓
2. Deposit to B: ₹100 ✓
COMMIT ✓

ACID properties maintained ✓
```

---

### Multiple Database Problem

```
E-commerce Purchase:

┌─────────────┐         ┌─────────────┐
│  Order DB   │         │Inventory DB │
├─────────────┤         ├─────────────┤
│ Orders: 100 │         │ Stock: 500  │
└─────────────┘         └─────────────┘

Transaction 1 (Order DB):
UPDATE orders SET count = 101
COMMIT ✓

Transaction 2 (Inventory DB):
UPDATE inventory SET stock = 400
ROLLBACK ✗ (Failed!)

Result:
Order DB: 101 ✓
Inventory DB: 500 ✗

INCONSISTENT! ✗
```

**Problem:**

```
Transaction is LOCAL to a database
- Each DB has own transaction manager
- Transaction 1 cannot rollback Transaction 2
- Different databases = Isolated transactions
- No global ACID guarantee
```

---

## Solution 1: Two-Phase Commit (2PC)

### Overview

```
2PC = Two phases to ensure distributed ACID

Phase 1: Voting (Prepare)
Phase 2: Decision (Commit)

Components:
- Transaction Coordinator
- Participants (microservices/databases)
```

---

### Phase 1: Prepare (Voting)

```
Coordinator asks: "Are you prepared to commit?"

Participants:
1. Execute update query
2. Lock rows
3. Make changes in DB
4. DO NOT commit yet
5. Respond: OK or NO
```

---

### Phase 2: Commit (Decision)

#### Success Case

```
All participants OK:
→ Coordinator sends COMMIT
→ Participants commit
→ Transaction complete ✓
```

#### Failure Case

```
Any participant NO:
→ Coordinator sends ABORT
→ All participants rollback
→ Transaction aborted ✓
```

---

### Log Files

**Purpose:** Recovery from failures

```
Coordinator Log:
├─ Prepare
└─ Commit/Abort

Participant Log:
├─ OK/NO
└─ Commit/Abort
```

---

### Failure Scenarios

#### Scenario 1: Prepare Message Lost

```
Solution: Participant timeout → ABORT
Safe to abort ✓
```

#### Scenario 2: OK Message Lost

```
Solution: Coordinator timeout → ABORT
Safe to abort ✓
```

#### Scenario 3: Commit Message Lost (BLOCKING!)

```
Problem: Participant BLOCKED!
Cannot decide on its own
Must wait for coordinator

BLOCKING PROTOCOL ✗
```

---

### 2PC Summary

**Advantages:**

```
✓ Ensures distributed ACID
✓ All participants commit or all abort
✓ Widely used
```

**Disadvantages:**

```
✗ Blocking protocol
✗ Participants blocked if coordinator fails
✗ Single point of failure
```

---

## Solution 2: Three-Phase Commit (3PC)

### Overview

```
3PC = Non-blocking improvement of 2PC

Phases:
1. Prepare (same as 2PC)
2. Pre-Commit (NEW - shares decision)
3. Commit (actual commit)

Key Difference:
Phase 2 shares coordinator's decision
Participants can decide independently
```

---

### Three Phases

```
Phase 1: Prepare (Voting)
- Same as 2PC
- Coordinator asks: "Prepared?"
- Participants respond: OK or NO

Phase 2: Pre-Commit (Decision Sharing)
- Coordinator shares decision
- NOT telling to commit
- Just informing: "I decided to commit/abort"
- Participants acknowledge

Phase 3: Commit (Actual Commit)
- Coordinator sends actual Commit/Abort
- Participants execute
```

---

### Failure Scenarios (Non-Blocking)

#### Scenario 1: Coordinator Fails After Pre-Commit

```
Participants check their logs:
- Pre-Commit received? YES
- Decision? COMMIT
- Action: Commit independently ✓

NON-BLOCKING ✓
```

#### Scenario 2: Coordinator Fails Before Pre-Commit

```
Participants communicate:
P1 asks P2: "Got Pre-Commit?"
P2: "No"
Both: "Safe to ABORT" ✓

NON-BLOCKING ✓
```

---

### 3PC Summary

**Advantages:**

```
✓ Non-blocking protocol
✓ Participants can decide independently
✓ Better availability than 2PC
```

**Disadvantages:**

```
✗ Very complex to implement
✗ Requires participant communication
✗ Not widely used (complexity)
```

---

## Solution 3: Saga Pattern

### Overview

```
Saga = Asynchronous distributed transaction
     = Long-running transaction
     = Sequential participant execution

Key Differences from 2PC/3PC:
- Asynchronous (not synchronous)
- No locks held across participants
- Compensating transactions for rollback
```

---

### When to Use Saga

```
Use Saga when:
1. Long-running transaction
2. Many participants (5+)
3. Sequential dependencies
4. Cannot hold locks for long time

Example:
Order → Payment → Inventory → Shipping → Notification
```

---

### Saga Flow

```
Sequential Execution:

P1 ✓ → P2 ✓ → P3 ✓ → P4 ✓ → P5 ✗

Each participant:
- Executes
- Commits (releases locks)
- Triggers next participant
```

---

### Compensating Transactions

```
When P5 fails, rollback chain:

P5 fails → Publish "Failed" event
P4 reads → Compensate → Publish "P4 Compensated"
P3 reads → Compensate → Publish "P3 Compensated"
P2 reads → Compensate → Publish "P2 Compensated"
P1 reads → Compensate → Publish "P1 Compensated"

All rolled back ✓
```

---

### 2PC vs 3PC vs Saga

```
┌──────────────┬─────────┬─────────┬─────────┐
│   Feature    │   2PC   │   3PC   │  Saga   │
├──────────────┼─────────┼─────────┼─────────┤
│ Nature       │  Sync   │  Sync   │  Async  │
│ Blocking     │   Yes   │   No    │   No    │
│ Locks        │  Held   │  Held   │Released │
│ Complexity   │  Medium │  High   │  Medium │
│ Use Case     │ Short   │ Short   │  Long   │
│ Consistency  │ Strong  │ Strong  │Eventual │
│ Popularity   │  High   │  Low    │  High   │
└──────────────┴─────────┴─────────┴─────────┘
```

---

## Summary

### Transaction Basics

```
Transaction = Set of DB operations
ACID: Atomicity, Consistency, Isolation, Durability
```

### Distributed Transaction Problem

```
Transaction is LOCAL to database
Multiple databases = No global ACID
Need coordination mechanism
```

### Solutions

**1. Two-Phase Commit (2PC):**
- Phases: Prepare → Commit
- Pros: Simple, widely used
- Cons: Blocking

**2. Three-Phase Commit (3PC):**
- Phases: Prepare → Pre-Commit → Commit
- Pros: Non-blocking
- Cons: Very complex

**3. Saga Pattern:**
- Nature: Asynchronous, sequential
- Pros: No long locks, scalable
- Cons: Eventual consistency

---

## Interview Tips

### Common Questions

**Q1: "What is the problem with distributed transactions?"**

```
Answer:
"Transaction is local to a database. In distributed systems with multiple databases, we can't guarantee global ACID properties.

Example:
Order DB: Transaction commits ✓
Inventory DB: Transaction fails ✗
Result: Inconsistent state

Solutions: 2PC, 3PC, Saga Pattern"
```

**Q2: "Explain Two-Phase Commit"**

```
Answer:
"2PC ensures distributed ACID through coordination.

Phase 1 - Prepare:
- Coordinator asks: 'Prepared to commit?'
- Participants execute, lock, respond OK/NO

Phase 2 - Commit:
- If all OK → COMMIT
- If any NO → ABORT

Pros: Strong consistency
Cons: Blocking if coordinator fails"
```

**Q3: "When would you use Saga over 2PC?"**

```
Answer:
"Use Saga for long-running, sequential transactions.

Saga: Asynchronous, eventual consistency
2PC: Synchronous, strong consistency

Use Saga when:
- Long-running (hours/days)
- Many participants
- Cannot hold locks long

Use 2PC when:
- Short transactions
- Strong consistency required"
```

### Key Points to Remember

```
1. Transaction = ACID properties
2. Distributed problem: Transaction local to DB
3. 2PC: Prepare → Commit (blocking)
4. 3PC: Prepare → Pre-Commit → Commit (non-blocking)
5. Saga: Asynchronous, compensating transactions
6. Choose based on duration, consistency, participants
```

---

**End of Lecture**

Distributed transaction handling is critical for microservices. Understanding 2PC (blocking, synchronous), 3PC (non-blocking, complex), and Saga (asynchronous, compensating) is essential. Choose based on transaction duration, consistency requirements, and system complexity.

**Key Takeaway:** Transaction is local to DB. Distributed systems need coordination: 2PC (simple, blocking), 3PC (complex, non-blocking), Saga (async, eventual consistency). Know the trade-offs!
