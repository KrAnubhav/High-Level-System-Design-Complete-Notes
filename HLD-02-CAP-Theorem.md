# HLD-02: CAP Theorem

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [What is CAP Theorem?](#what-is-cap-theorem)
3. [Distributed System with Replicated Data](#distributed-system-with-replicated-data)
4. [The Three Desirable Properties](#the-three-desirable-properties)
   - [C - Consistency](#c---consistency)
   - [A - Availability](#a---availability)
   - [P - Partition Tolerance](#p---partition-tolerance)
5. [The CAP Theorem Rule](#the-cap-theorem-rule)
6. [Why All Three Cannot Be Achieved Together](#why-all-three-cannot-be-achieved-together)
7. [Scenario 1: Trying to Achieve CAP (Not Possible)](#scenario-1-trying-to-achieve-cap-not-possible)
8. [Scenario 2: AP (Availability + Partition Tolerance)](#scenario-2-ap-availability--partition-tolerance)
9. [Scenario 3: CP (Consistency + Partition Tolerance)](#scenario-3-cp-consistency--partition-tolerance)
10. [Scenario 4: CA (Consistency + Availability)](#scenario-4-ca-consistency--availability)
11. [Real-World Trade-offs](#real-world-trade-offs)
12. [Summary](#summary)

---

## Introduction

**CAP Theorem** is a fundamental concept in high-level system design.

**Why is it important?**
- You might be directly asked to define and explain CAP theorem in interviews
- When designing a distributed system, you need to think about CAP **first**
- Your entire design changes based on trade-offs
- If trade-offs are not considered early, it becomes very difficult to change the system design later

**When to consider CAP?**
- In the **very starting phase** of system design
- Before making architectural decisions

---

## What is CAP Theorem?

**CAP Theorem** defines **desirable properties** of a **distributed system** with **replicated data**.

**Remember this definition:**
> CAP Theorem defines desirable properties of a distributed system with replicated data.

---

## Distributed System with Replicated Data

### Example Architecture

```
                    ┌─────────────┐
                    │ Application │
                    └─────────────┘
                          │
                ┌─────────┴─────────┐
                │                   │
                ▼                   ▼
         ┌──────────┐        ┌──────────┐
         │  DB Node │        │  DB Node │
         │     B    │◄──────►│     C    │
         │ (India)  │  Sync  │   (US)   │
         └──────────┘        └──────────┘
```

**Characteristics:**
- **Distributed System:** DB nodes spread across different continents
- **Replicated Data:** Data in B is the same as data in C
- **Synchronization:** B and C synchronize data with each other
- Application can query B or C
- User doesn't know which node they're querying

---

## The Three Desirable Properties

**CAP stands for:**
- **C** = Consistency
- **A** = Availability
- **P** = Partition Tolerance

---

## C - Consistency

**Definition:** After a successful write in any node, if you read from any other node, you should get the same data.

### Example

```
Initial State:
┌──────────┐        ┌──────────┐
│  Node B  │        │  Node C  │
│  A = 4   │◄──────►│  A = 4   │
└──────────┘        └──────────┘

Write Operation:
Application writes: A = 5 to Node B

After Write:
┌──────────┐        ┌──────────┐
│  Node B  │        │  Node C  │
│  A = 5   │───────►│  A = 5   │ (Replicated)
└──────────┘        └──────────┘

Read Operation:
Application reads from Node C → Should get A = 5
```

**Consistency means:**
- Getting **consistent data** from all nodes
- After successful write on Node B, reading from Node C should return the same value
- All nodes have the **same data** at any point in time

---

## A - Availability

**Definition:** Every request receives a response (whether success or failure). The system is always operational.

### Example

```
┌─────────────┐
│ Application │
└─────────────┘
      │
      ├──────────────┬──────────────┐
      │              │              │
      ▼              ▼              ▼
┌──────────┐   ┌──────────┐   ┌──────────┐
│  Node B  │   │  Node C  │   │  Node D  │
│ Response │   │ Response │   │ Response │
│ (Success │   │ (Success │   │ (Failure │
│  or Fail)│   │  or Fail)│   │  or Fail)│
└──────────┘   └──────────┘   └──────────┘
```

**Availability means:**
- **All nodes should respond**
- Response can be success or failure
- At least they should respond with something:
  - "Data fetched successfully" ✅
  - "Couldn't fetch the data" ❌
- As long as all DB nodes are responding → System is **available**

---

## P - Partition Tolerance

**Definition:** When communication between nodes breaks, but the system is still up and able to handle queries.

### Understanding Partition Tolerance

```
Before Partition:
┌──────────┐        ┌──────────┐
│  Node B  │◄──────►│  Node C  │
│          │  Sync  │          │
└──────────┘        └──────────┘
      ▲                  ▲
      │                  │
      └────── App ───────┘

After Partition (Communication Breakage):
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│          │            │          │
└──────────┘            └──────────┘
      ▲                      ▲
      │                      │
      └─────── App ──────────┘
         (Still works!)
```

**Key Points:**
- Communication between B and C is **broken**
- B and C cannot replicate data to each other
- **But** the application can still query B or C
- User doesn't know about the internal partition
- User only knows they're querying "the system"
- **Partition has occurred internally** (B is separate, C is separate)
- **System is still up** and responding to queries

**Partition Tolerance means:**
- System continues to operate despite network partitions
- Even if nodes can't communicate with each other, the system doesn't go down

---

## The CAP Theorem Rule

### The Fundamental Rule

```
        Consistency (C)
              ╱  ╲
             ╱    ╲
            ╱      ╲
           ╱   ╳╳   ╲
          ╱    ╳╳    ╲
         ╱     ╳╳     ╲
        ╱      ╳╳      ╲
       ╱       ╳╳       ╲
Availability ────────── Partition
    (A)                 Tolerance (P)
```

**Possible Combinations:**
✅ **CA** (Consistency + Availability)
✅ **CP** (Consistency + Partition Tolerance)
✅ **AP** (Availability + Partition Tolerance)

❌ **CAP** (All three together) - **NOT POSSIBLE**

**The center area (CAP) is not usable.**

---

## Why All Three Cannot Be Achieved Together

Let's explore all scenarios to understand why CAP is impossible.

---

## Scenario 1: Trying to Achieve CAP (Not Possible)

### Setup

```
┌──────────┐        ┌──────────┐
│  Node B  │        │  Node C  │
│  A = 5   │◄──────►│  A = 5   │
└──────────┘        └──────────┘
```

### Requirements
- **C (Consistency):** Data should be the same in both nodes at any time
- **A (Availability):** Both B and C should respond to queries
- **P (Partition Tolerance):** System should stay up even if communication breaks

### What Happens When Partition Occurs

```
Step 1: Partition Happens
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 5   │            │  A = 5   │
└──────────┘            └──────────┘

Step 2: Write Request to Node B
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 6   │────╳──────►│  A = 5   │ (Can't update!)
└──────────┘            └──────────┘
   Updated              Stale Data

Step 3: System State
- Node B has A = 6
- Node C has A = 5
- Communication is broken (partition)
- Both nodes are available (responding)
```

### Result

**Problem:** System became **inconsistent**
- If application queries Node B → Gets A = 6
- If application queries Node C → Gets A = 5 (stale data)

**Conclusion:**
- We achieved **A** (both nodes available)
- We achieved **P** (system still up despite partition)
- We **lost C** (consistency)

**Therefore, CAP together is NOT possible.**

---

## Scenario 2: AP (Availability + Partition Tolerance)

### Trade-off: Drop Consistency

```
Initial State:
┌──────────┐        ┌──────────┐
│  Node B  │        │  Node C  │
│  A = 5   │◄──────►│  A = 5   │
└──────────┘        └──────────┘

Partition Occurs:
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 5   │            │  A = 5   │
└──────────┘            └──────────┘

Write A = 6 to Node B:
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 6   │────╳──────►│  A = 5   │
└──────────┘            └──────────┘
  Updated              Inconsistent!
     ▲                      ▲
     │                      │
     └─── Both Available ───┘
```

### What We Achieved

✅ **Availability (A):**
- Node B responds (success or failure)
- Node C responds (success or failure)
- Both nodes are up and responding

✅ **Partition Tolerance (P):**
- Communication between B and C is broken
- System is still up
- Application can still query both nodes

❌ **Consistency (C):**
- Node B has A = 6
- Node C has A = 5
- Data is inconsistent across nodes

### Use Case
- Systems where **availability is more critical** than consistency
- Example: Social media feeds, DNS, caching systems

---

## Scenario 3: CP (Consistency + Partition Tolerance)

### Trade-off: Drop Availability

```
Initial State:
┌──────────┐        ┌──────────┐
│  Node B  │        │  Node C  │
│  A = 5   │◄──────►│  A = 5   │
└──────────┘        └──────────┘

Partition Occurs:
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 5   │            │  A = 5   │
└──────────┘            └──────────┘

To Maintain Consistency, Take Node C Down:
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 5   │            │   DOWN   │
└──────────┘            └──────────┘
     ▲                       ╳
     │                       ╳
     └─── Only B Available ──╳

Write A = 6 to Node B:
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 6   │            │   DOWN   │
└──────────┘            └──────────┘
  Updated               Not Serving
  Consistent!           Requests
```

### What We Achieved

✅ **Consistency (C):**
- Only Node B is serving requests
- All queries go to Node B
- Always get consistent data (A = 6)

✅ **Partition Tolerance (P):**
- Communication is broken
- System is still up (Node B is operational)
- Requests are being fulfilled

❌ **Availability (A):**
- Node C is down
- Not all nodes are available
- Only one node is responding

### Strategy
- **Take one node down** during partition
- Route all requests to the available node
- Maintain consistency by having single source of truth

### Use Case
- Systems where **consistency is critical**
- Example: Banking systems, financial transactions, inventory management

---

## Scenario 4: CA (Consistency + Availability)

### Trade-off: Drop Partition Tolerance

```
Normal Operation (No Partition):
┌──────────┐        ┌──────────┐
│  Node B  │        │  Node C  │
│  A = 5   │◄──────►│  A = 5   │
└──────────┘        └──────────┘
     ▲                  ▲
     │                  │
     └─── Both Available & Consistent

If Partition Occurs:
┌──────────┐   ╳╳╳╳╳╳╳   ┌──────────┐
│  Node B  │   Broken   │  Node C  │
│  A = 5   │            │  A = 5   │
└──────────┘            └──────────┘
     ╳                      ╳
     ╳                      ╳
     └───── SYSTEM DOWN ────┘
```

### What We Achieved

✅ **Consistency (C):**
- When system is up, all nodes have same data
- No inconsistency allowed

✅ **Availability (A):**
- When system is up, all nodes respond
- Both B and C are available

❌ **Partition Tolerance (P):**
- If partition happens → System goes down
- Cannot tolerate network failures
- Must stop accepting requests during partition

### Strategy
- **Stop the system** when partition occurs
- Don't allow writes during partition (would cause inconsistency)
- Don't allow reads from inconsistent nodes
- Wait until partition is resolved

### Problem
- If communication breaks for 10 minutes → System is down for 10 minutes
- Not acceptable in modern distributed systems

### Use Case
- Single-node databases (no distribution)
- Systems that can afford downtime
- **Rarely used in modern distributed architectures**

---

## Real-World Trade-offs

### The Golden Rule

**In modern distributed systems:**

✅ **Never trade off Partition Tolerance (P)**

**Why?**
- Distributed architecture is very common nowadays
- Communication breakage is a **common scenario**
- Network failures happen frequently
- Cannot afford to shut down the entire system for network issues

### The Real Choice

**You must choose between:**
- **CP** (Consistency + Partition Tolerance) → Drop Availability
- **AP** (Availability + Partition Tolerance) → Drop Consistency

```
┌─────────────────────────────────────────────────┐
│         Modern Distributed Systems              │
├─────────────────────────────────────────────────┤
│                                                 │
│  Always Keep: P (Partition Tolerance)           │
│                                                 │
│  Choose One:                                    │
│  ┌─────────────────┐   OR   ┌─────────────────┐│
│  │       CP        │        │       AP        ││
│  │  (Consistency)  │        │ (Availability)  ││
│  └─────────────────┘        └─────────────────┘│
│                                                 │
└─────────────────────────────────────────────────┘
```

### Interview Question

**Q: What would be your trade-off on Consistency, Availability, and Partition Tolerance?**

**Answer:**
- We will **never trade off Partition Tolerance**
- We will trade off **only between Consistency and Availability**
- Choice depends on the use case:
  - **Banking/Financial systems** → Choose **CP** (Consistency is critical)
  - **Social media/Caching** → Choose **AP** (Availability is critical)

---

## Summary

### Key Takeaways

1. **CAP Theorem Definition:**
   - Defines desirable properties of distributed systems with replicated data
   - C = Consistency, A = Availability, P = Partition Tolerance

2. **The Rule:**
   - Only **2 out of 3** properties can be achieved together
   - **CAP together is impossible**

3. **The Three Properties:**
   - **Consistency:** Same data across all nodes after successful write
   - **Availability:** All nodes respond (success or failure)
   - **Partition Tolerance:** System stays up despite network failures

4. **Possible Combinations:**
   - **AP:** Available + Partition Tolerant (Drop Consistency)
   - **CP:** Consistent + Partition Tolerant (Drop Availability)
   - **CA:** Consistent + Available (Drop Partition Tolerance - rarely used)

5. **Real-World Approach:**
   - Always keep **P** (Partition Tolerance)
   - Choose between **C** and **A** based on requirements

6. **Trade-off Decision:**
   - **Need consistency?** → Choose **CP** (Banking, transactions)
   - **Need availability?** → Choose **AP** (Social media, caching)

### Quick Reference Table

| Combination | What You Get | What You Lose | Use Case |
|-------------|--------------|---------------|----------|
| **AP** | Available + Partition Tolerant | Consistency | Social media, DNS, Caching |
| **CP** | Consistent + Partition Tolerant | Availability | Banking, Financial systems |
| **CA** | Consistent + Available | Partition Tolerance | Single-node DB (rare) |
| **CAP** | All three | Nothing | **IMPOSSIBLE** |

### Important for Interviews

**Remember:**
- CAP is considered in the **starting phase** of system design
- Trade-offs affect the entire architecture
- Partition Tolerance is **non-negotiable** in distributed systems
- The real choice is always between **Consistency** and **Availability**

---

**End of Lecture**

Understanding CAP Theorem is crucial for making architectural decisions in distributed system design. This forms the foundation for designing scalable, reliable systems.
