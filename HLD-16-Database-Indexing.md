# HLD-16: Database Indexing (RDBMS)

---

## 📋 Table of Contents
1. [Introduction](#introduction)
2. [How Table Data is Stored](#how-table-data-is-stored)
3. [Data Pages](#data-pages)
4. [Data Blocks](#data-blocks)
5. [B+ Tree Data Structure](#b-tree-data-structure)
6. [How DBMs Uses B+ Tree](#how-dbms-uses-b-tree)
7. [Clustered Index](#clustered-index)
8. [Non-Clustered Index](#non-clustered-index)
9. [Query Execution Flow](#query-execution-flow)
10. [Summary](#summary)
11. [Interview Tips](#interview-tips)

---

## Introduction

**Topic:** Database Indexing (RDBMS)

**Importance:**
- Very frequently asked in interviews
- Critical for database performance
- Essential for daily software engineering

**Three Prerequisites to Understand Indexing:**
1. How table data is actually stored
2. What type of indexing exists
3. Data structure used for indexing (B+ Tree)

**Common Misconception:**

```
❌ Simplified View:
Table → Create Index → Fast Query

✓ Reality:
Table → Data Pages → Data Blocks → B+ Tree → Index Pages → Clustered/Non-Clustered
```

---

## How Table Data is Stored

### Logical vs Physical Representation

**Logical Representation (What we see):**

```
Employee Table:
┌────────────┬──────────┬─────────────┐
│ Employee_ID│   Name   │   Address   │
├────────────┼──────────┼─────────────┤
│     1      │    A     │   Addr1     │
│     2      │    B     │   Addr2     │
│     3      │    C     │   Addr3     │
│     4      │    D     │   Addr4     │
└────────────┴──────────┴─────────────┘

This is just a LOGICAL representation
NOT how data is physically stored!
```

**Physical Representation (Reality):**

```
Data is stored in DATA PAGES
Managed by DBMs (Database Management System)
```

---

## Data Pages

### Structure

```
Data Page Size: 8 KB (8192 bytes)

┌─────────────────────────────────────┐
│         HEADER (96 bytes)           │
│  - Page Number                      │
│  - Free Space                       │
│  - Checksum                         │
│  - Metadata                         │
├─────────────────────────────────────┤
│    DATA RECORDS (8060 bytes)        │
│                                     │
│  Row 1 Data                         │
│  Row 2 Data                         │
│  Row 3 Data                         │
│  ...                                │
│  (Actual table rows stored here)    │
│                                     │
├─────────────────────────────────────┤
│     OFFSET ARRAY (36 bytes)         │
│  [0] → Row pointer                  │
│  [1] → Row pointer                  │
│  [2] → Row pointer                  │
│  ...                                │
└─────────────────────────────────────┘

Total: 96 + 8060 + 36 = 8192 bytes (8 KB)
```

---

### Offset Array Purpose

```
Offset Array = Array of pointers to rows

Example:
┌─────────────────────────────────────┐
│ DATA RECORDS:                       │
│  Row 2 (64 bytes)                   │
│  Row 4 (64 bytes)                   │
│  Row 5 (64 bytes)                   │
│  Row 3 (64 bytes)                   │
│  Row 1 (64 bytes)                   │
├─────────────────────────────────────┤
│ OFFSET ARRAY:                       │
│  [0] → Points to Row 2              │
│  [1] → Points to Row 4              │
│  [2] → Points to Row 5              │
│  [3] → Points to Row 3              │
│  [4] → Points to Row 1              │
└─────────────────────────────────────┘

Purpose: Maintains logical ordering
(Will be clear with Clustered Index)
```

---

### Capacity Calculation

```
Row Size: 64 bytes
Data Records Space: 8060 bytes

Rows per Page = 8060 / 64 = 125 rows

One 8 KB data page can store ~125 rows
(if each row is 64 bytes)
```

**Example Data Page:**

```
┌─────────────────────────────────────┐
│ HEADER: Page ID = 1                 │
├─────────────────────────────────────┤
│ DATA RECORDS:                       │
│  Row 1: [1, A, Addr1]               │
│  Row 2: [2, B, Addr2]               │
│  Row 3: [3, C, Addr3]               │
│  Row 4: [4, D, Addr4]               │
│  Row 5: [5, E, Addr5]               │
│  ...                                │
│  (up to 125 rows)                   │
├─────────────────────────────────────┤
│ OFFSET ARRAY:                       │
│  [0] → Row 2                        │
│  [1] → Row 4                        │
│  [2] → Row 5                        │
│  [3] → Row 3                        │
│  [4] → Row 1                        │
└─────────────────────────────────────┘
```

---

### Multiple Data Pages

```
Table with 10,000 rows:

125 rows per page
10,000 / 125 = 80 pages needed

┌──────────┐  ┌──────────┐  ┌──────────┐       ┌──────────┐
│  Page 1  │  │  Page 2  │  │  Page 3  │  ...  │ Page 80  │
│ 125 rows │  │ 125 rows │  │ 125 rows │       │ 125 rows │
└──────────┘  └──────────┘  └──────────┘       └──────────┘

DBMs creates and manages all data pages
```

---

## Data Blocks

### Definition

```
Data Block = Minimum amount of data for one I/O operation
           = Physical storage unit on disk
```

**Key Points:**

```
✓ Managed by: Underlying storage system (HDD/SSD)
✗ NOT managed by: DBMs

DBMs controls: Data Pages
Storage controls: Data Blocks
```

---

### Data Block Size

```
Size Range: 4 KB to 32 KB
Most Common: 8 KB

┌─────────────────────────────────────┐
│        Data Block (8 KB)            │
├─────────────────────────────────────┤
│      Data Page 1 (8 KB)             │
└─────────────────────────────────────┘

If Data Block = 8 KB:
→ 1 Data Block = 1 Data Page

If Data Block = 32 KB:
→ 1 Data Block = 4 Data Pages (8 KB each)
```

---

### Page to Block Mapping

```
DBMs maintains mapping:

┌──────────────┬──────────────┐
│  Data Page   │  Data Block  │
├──────────────┼──────────────┤
│   Page 1     │   Block 1    │
│   Page 2     │   Block 1    │
│   Page 3     │   Block 5    │
│   Page 4     │   Block 2    │
│   ...        │   ...        │
└──────────────┴──────────────┘

DBMs has NO control over which block
Pages can be scattered across disk
```

**Visual:**

```
Disk (Physical Storage):
┌─────────────────────────────────────┐
│  Block 1: [Page 1][Page 2]          │
│  Block 2: [Page 4]                  │
│  Block 3: [Empty]                   │
│  Block 4: [Empty]                   │
│  Block 5: [Page 3]                  │
│  ...                                │
└─────────────────────────────────────┘

DBMs Mapping Table:
Page 1 → Block 1
Page 2 → Block 1
Page 3 → Block 5
Page 4 → Block 2
```

---

## B+ Tree Data Structure

### Why B+ Tree?

```
Without Indexing:
- Linear search through all pages
- Time Complexity: O(n)
- 1 million rows = 1 million checks (worst case)

With B+ Tree Indexing:
- Balanced tree search
- Time Complexity: O(log n)
- 1 million rows = ~20 checks (worst case)
```

---

### B-Tree Basics

**B-Tree Properties:**

```
1. Maintains sorted data
2. All leaves at same level
3. M-order B-Tree:
   - Each node has at most M children
   - Each node has at most M-1 keys

Example: 3-order B-Tree
- Max children: 3
- Max keys per node: 2
```

**Node Structure (3-order):**

```
┌─────────────────────────────────────┐
│  [Ptr1] [Key1] [Ptr2] [Key2] [Ptr3] │
└─────────────────────────────────────┘

Ptr1: Points to children < Key1
Ptr2: Points to children >= Key1 and < Key2
Ptr3: Points to children >= Key2

Max 2 keys, Max 3 pointers
```

---

### B-Tree Construction Example

**Insert sequence: 9, 33, 75, 41, 98, 214, 126, 55, 72**

**Step 1: Insert 9**

```
┌─────┐
│  9  │
└─────┘
```

**Step 2: Insert 33**

```
┌──────────┐
│  9 | 33  │
└──────────┘

Node full (2 keys max)
```

**Step 3: Insert 75**

```
Need to insert: 9, 33, 75
Sorted: 9, 33, 75
Middle (33) goes to parent

      ┌────┐
      │ 33 │
      └────┘
      /    \
┌─────┐  ┌─────┐
│  9  │  │ 75  │
└─────┘  └─────┘

33 left → < 33 (9)
33 right → >= 33 (75)
```

**Step 4: Insert 41**

```
41 > 33 → Go right
Insert in sorted order

      ┌────┐
      │ 33 │
      └────┘
      /    \
┌─────┐  ┌──────────┐
│  9  │  │ 41 | 75  │
└─────┘  └──────────┘
```

**Step 5: Insert 98**

```
98 > 33 → Go right
Need: 41, 75, 98
Middle (75) goes to parent

      ┌──────────┐
      │ 33 | 75  │
      └──────────┘
      /    |    \
┌─────┐ ┌────┐ ┌────┐
│  9  │ │ 41 │ │ 98 │
└─────┘ └────┘ └────┘
```

**Step 6: Insert 214**

```
214 > 75 → Go right
Insert in sorted order

      ┌──────────┐
      │ 33 | 75  │
      └──────────┘
      /    |     \
┌─────┐ ┌────┐ ┌───────────┐
│  9  │ │ 41 │ │ 98 | 214  │
└─────┘ └────┘ └───────────┘
```

**Step 7: Insert 126**

```
126 > 75 → Go right
Need: 98, 126, 214
Middle (126) goes to parent

Parent becomes: 33, 75, 126
Middle (75) goes up

         ┌────┐
         │ 75 │
         └────┘
        /      \
  ┌────┐      ┌─────┐
  │ 33 │      │ 126 │
  └────┘      └─────┘
  /    \      /     \
┌───┐ ┌───┐ ┌───┐ ┌─────┐
│ 9 │ │41 │ │98 │ │ 214 │
└───┘ └───┘ └───┘ └─────┘
```

**Step 8: Insert 55**

```
55 > 33, 55 < 75 → Go to 41
Insert: 41, 55

         ┌────┐
         │ 75 │
         └────┘
        /      \
  ┌────┐      ┌─────┐
  │ 33 │      │ 126 │
  └────┘      └─────┘
  /    \      /     \
┌───┐ ┌────────┐ ┌───┐ ┌─────┐
│ 9 │ │ 41|55 │ │98 │ │ 214 │
└───┘ └────────┘ └───┘ └─────┘
```

**Final Tree (after 72):**

```
         ┌────┐
         │ 75 │
         └────┘
        /      \
  ┌────┐      ┌─────┐
  │ 33 │      │ 126 │
  └────┘      └─────┘
  /    \      /     \
┌───┐ ┌───┐ ┌───┐ ┌─────┐
│ 9 │ │55 │ │98 │ │ 214 │
└───┘ └───┘ └───┘ └─────┘
     /   \
  ┌───┐ ┌───┐
  │41 │ │72 │
  └───┘ └───┘

Leaf nodes (bottom): 9, 41, 55, 72, 98, 214
SORTED ORDER ✓
```

---

### B+ Tree vs B-Tree

```
B+ Tree = B-Tree + Additional Features

Key Differences:

1. Leaf nodes are linked:
   ┌───┐   ┌───┐   ┌───┐   ┌───┐
   │ 9 │──►│41 │──►│55 │──►│72 │
   └───┘   └───┘   └───┘   └───┘

2. Leaf nodes hold actual index values
   - Root/Intermediate: For searching only
   - Leaf: Actual indexed column values

3. Root/Intermediate can have deleted values
   - Used for tree navigation
   - May not exist in actual data
```

---

## How DBMs Uses B+ Tree

### Example Table

```
Employee Table:
┌────────────┬──────────┐
│ Employee_ID│   Name   │
├────────────┼──────────┤
│     19     │    E1    │
│     25     │    E2    │
│     30     │    E3    │
│     17     │    E4    │
│      6     │    E5    │
│      1     │    E6    │
│      5     │    E7    │
└────────────┴──────────┘

Index on: Employee_ID
```

---

### B+ Tree Construction with Page Pointers

**Assumptions:**
- 3-order B-Tree
- Each data page holds 3 rows max

**Insert 19 (Row 1):**

```
B+ Tree:
┌────┐
│ 19 │ → Points to Page 1
└────┘

Data Pages:
┌──────────────────┐
│ Page 1:          │
│  Row: [19, E1]   │
└──────────────────┘

Mapping:
Page 1 → Block 1
```

**Insert 25 (Row 2):**

```
B+ Tree:
┌──────────┐
│ 19 | 25  │
└──────────┘
  ↓    ↓
Page 1 Page 1

Data Pages:
┌──────────────────┐
│ Page 1:          │
│  [19, E1]        │
│  [25, E2]        │
└──────────────────┘
```

**Insert 30 (Row 3):**

```
B+ Tree:
┌──────────────┐
│ 19 | 25 | 30 │
└──────────────┘
  ↓    ↓    ↓
Page 1 Page 1 Page 1

Split (middle = 25):

      ┌────┐
      │ 25 │
      └────┘
      /    \
┌─────┐  ┌──────────┐
│ 19  │  │ 25 | 30  │
└─────┘  └──────────┘
  ↓         ↓    ↓
Page 1    Page 1 Page 1

Data Pages:
┌──────────────────┐
│ Page 1:          │
│  [19, E1]        │
│  [25, E2]        │
│  [30, E3]        │
└──────────────────┘

Page 1 is FULL (3 rows)
```

**Insert 17 (Row 4):**

```
B+ Tree position: 17 < 25 → Left
Neighbor: 19 → Page 1

Try to insert in Page 1:
Page 1 already has 3 rows (FULL!)

PAGE SPLITTING occurs:

      ┌────┐
      │ 25 │
      └────┘
      /    \
┌──────────┐  ┌──────────┐
│ 17 | 19  │  │ 25 | 30  │
└──────────┘  └──────────┘
  ↓    ↓       ↓    ↓
Page 1 Page 1 Page 2 Page 2

Data Pages:
┌──────────────────┐  ┌──────────────────┐
│ Page 1:          │  │ Page 2:          │
│  [17, E4]        │  │  [25, E2]        │
│  [19, E1]        │  │  [30, E3]        │
└──────────────────┘  └──────────────────┘

Mapping:
Page 1 → Block 1
Page 2 → Block 1
```

**Insert 6 (Row 5):**

```
B+ Tree position: 6 < 25 → Left
Check neighbors: 17 → Page 1

Page 1 has space (2 rows):

      ┌────┐
      │ 25 │
      └────┘
      /    \
┌─────────────┐  ┌──────────┐
│ 6 | 17 | 19 │  │ 25 | 30  │
└─────────────┘  └──────────┘
  ↓   ↓    ↓      ↓    ↓
Page 1 Page 1 Page 1 Page 2 Page 2

Data Pages:
┌──────────────────┐
│ Page 1:          │
│  [17, E4]        │
│  [19, E1]        │
│  [6, E5]         │
└──────────────────┘
```

**Insert 1 (Row 6):**

```
B+ Tree position: 1 < 25 → Left
Neighbor: 6 → Page 1

Page 1 FULL (3 rows)!

PAGE SPLITTING:

Split: 1, 6, 17, 19
Middle: 6 (goes up)

      ┌──────────┐
      │  6 | 25  │
      └──────────┘
      /    |    \
┌──────┐ ┌────────┐ ┌──────────┐
│  1   │ │ 17|19  │ │ 25 | 30  │
└──────┘ └────────┘ └──────────┘
   ↓       ↓   ↓      ↓    ↓
Page 3   Page 1 Page 1 Page 2 Page 2

Data Pages:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Page 1:      │  │ Page 2:      │  │ Page 3:      │
│  [17, E4]    │  │  [25, E2]    │  │  [1, E6]     │
│  [19, E1]    │  │  [30, E3]    │  └──────────────┘
└──────────────┘  └──────────────┘

Mapping:
Page 1 → Block 1
Page 2 → Block 1
Page 3 → Block 2
```

**Final B+ Tree:**

```
      ┌──────────┐
      │  6 | 25  │
      └──────────┘
      /    |    \
┌──────┐ ┌────────┐ ┌──────────┐
│  1   │ │ 17|19  │ │ 25 | 30  │
└──────┘ └────────┘ └──────────┘
   ↓       ↓   ↓      ↓    ↓
Page 3   Page 1 Page 1 Page 2 Page 2

Leaf nodes (sorted): 1, 17, 19, 25, 30 ✓
```

---

### Key Observations

```
1. B+ Tree maintains sorted order
   Leaf: 1, 17, 19, 25, 30

2. Each leaf node points to data page
   1 → Page 3
   17 → Page 1
   19 → Page 1
   25 → Page 2
   30 → Page 2

3. Page splitting when full
   - Creates new page
   - Redistributes rows
   - Updates pointers
   - Updates mapping

4. DBMs determines best page using B+ Tree
   - Traverse tree
   - Find logical position
   - Check neighbor's page
   - Insert or split
```

---

## Clustered Index

### Definition

```
Clustered Index:
Order of rows inside data pages matches order of index

Physical row ordering = Index ordering
```

---

### How It Works

**Example Table:**

```
┌────────────┬──────────┬─────────┐
│ Employee_ID│   Name   │ Address │
├────────────┼──────────┼─────────┤
│      1     │    A     │  Addr1  │
│      4     │    C     │  Addr3  │
│      5     │    D     │  Addr4  │
│      2     │    B     │  Addr2  │
└────────────┴──────────┴─────────┘

Rows inserted in order: 1, 4, 5, 2
```

**Data Page (Physical Storage):**

```
┌─────────────────────────────────────┐
│ DATA RECORDS (insertion order):     │
│  [1, A, Addr1]                      │
│  [4, C, Addr3]                      │
│  [5, D, Addr4]                      │
│  [2, B, Addr2]                      │
├─────────────────────────────────────┤
│ OFFSET ARRAY (index order):         │
│  [0] → Points to [1, A, Addr1]      │
│  [1] → Points to [2, B, Addr2]      │
│  [2] → Points to [4, C, Addr3]      │
│  [3] → Points to [5, D, Addr4]      │
└─────────────────────────────────────┘

Index on Employee_ID: 1, 2, 4, 5
Offset array maintains this order!
```

**Visual:**

```
B+ Tree (Clustered Index):
┌─────────────────┐
│  1, 2, 4, 5     │ (sorted)
└─────────────────┘

Data Page Traversal (via offset):
Offset[0] → Row 1
Offset[1] → Row 2
Offset[2] → Row 4
Offset[3] → Row 5

Result: 1, 2, 4, 5 (sorted) ✓

Offset array provides logical ordering!
```

---

### Clustered Index Rules

**Rule 1: Only ONE clustered index per table**

```
Why?
- Row order can match only ONE index
- Cannot order by Employee_ID AND Name simultaneously

Example:
Employee_ID: 1, 2, 4, 5
Name: A, B, C, D

Can order by:
- Employee_ID (1, 2, 4, 5) OR
- Name (A, B, C, D)

NOT both!
```

**Rule 2: Primary Key = Clustered Index (by default)**

```
Table with Primary Key:
┌────────────┬──────────┐
│ Employee_ID│   Name   │ ← Primary Key
├────────────┼──────────┤
│     1      │    A     │
│     2      │    B     │
└────────────┴──────────┘

DBMs automatically:
- Uses Employee_ID as clustered index
- Creates B+ Tree on Employee_ID
- Orders data pages by Employee_ID
```

**Rule 3: No Primary Key → Hidden Column**

```
Table without Primary Key:
┌──────────┬─────────┐
│   Name   │ Address │
├──────────┼─────────┤
│    A     │  Addr1  │
│    B     │  Addr2  │
└──────────┴─────────┘

DBMs internally creates:
┌────────────┬──────────┬─────────┐
│ _RowID     │   Name   │ Address │ ← Hidden
├────────────┼──────────┼─────────┤
│     1      │    A     │  Addr1  │
│     2      │    B     │  Addr2  │
└────────────┴──────────┴─────────┘

_RowID:
- Auto-increment (1, 2, 3, ...)
- Unique, Not Null
- Used as clustered index
- NOT visible to user
```

**Rule 4: Adding Primary Key Later**

```
Scenario:
1. Table created without primary key
2. DBMs creates hidden _RowID column
3. 1 million rows inserted
4. B+ Tree built on _RowID
5. User adds primary key on Employee_ID

What happens:
✗ Discard _RowID B+ Tree
✓ Rebuild B+ Tree on Employee_ID
✓ Reorder all data pages
✓ Update all pointers
✓ Reshuffle rows

EXPENSIVE OPERATION!
```

---

## Non-Clustered Index

### Definition

```
Non-Clustered Index:
- Secondary index
- Does NOT control row ordering
- Can have MULTIPLE per table
```

---

### How It Works

**Example:**

```
Table:
┌────────────┬──────────┬─────────┐
│ Employee_ID│   Name   │ Address │
├────────────┼──────────┼─────────┤
│     19     │    A     │  Addr1  │
│     25     │    B     │  Addr2  │
│     30     │    D     │  Addr3  │
│     17     │    D     │  Addr4  │
└────────────┴──────────┴─────────┘

Clustered Index: Employee_ID (Primary Key)
Non-Clustered Index: Name (Secondary Index)
```

**Clustered Index (Employee_ID):**

```
B+ Tree:
      ┌────┐
      │ 25 │
      └────┘
      /    \
┌──────────┐  ┌──────────┐
│ 17 | 19  │  │ 25 | 30  │
└──────────┘  └──────────┘
  ↓    ↓       ↓    ↓
Page 1 Page 1 Page 2 Page 2

Controls row ordering ✓
```

**Non-Clustered Index (Name):**

```
B+ Tree:
      ┌───┐
      │ B │
      └───┘
      /   \
┌─────┐  ┌─────┐
│  A  │  │  D  │
└─────┘  └─────┘
  ↓        ↓
Page 1   Page 2

Does NOT control row ordering ✗
Just points to pages
```

---

### Multiple Non-Clustered Indexes

```
Table with 1 million rows:

Clustered Index (Primary Key):
- Employee_ID
- 1 B+ Tree
- Controls row ordering

Non-Clustered Indexes:
- Name → B+ Tree (1 million nodes)
- Address → B+ Tree (1 million nodes)
- Department → B+ Tree (1 million nodes)

Total: 4 B+ Trees!
```

---

### Overhead of Non-Clustered Indexes

**Memory Overhead:**

```
1 million rows
3 non-clustered indexes

Each B+ Tree:
- 1 million leaf nodes
- Intermediate nodes
- Stored in INDEX PAGES
- Index pages stored in data blocks

Memory required:
- Data pages (actual rows)
- Index pages (B+ Trees)
- Mapping tables
```

**Maintenance Overhead:**

```
Insert 1 new row:

1. Update Clustered Index B+ Tree
   - Find position
   - Insert node
   - Possible page split
   - Update pointers

2. Update Non-Clustered Index 1
   - Find position
   - Insert node
   - Possible page split
   - Update pointers

3. Update Non-Clustered Index 2
   - Same process

4. Update Non-Clustered Index 3
   - Same process

4 B+ Tree updates for 1 insert!
```

**Delete 1 row:**

```
1. Delete from Clustered Index
   - Remove node
   - Update pointers
   - Possible tree rebalancing

2. Delete from Non-Clustered Index 1
3. Delete from Non-Clustered Index 2
4. Delete from Non-Clustered Index 3

4 B+ Tree updates for 1 delete!
```

---

### When to Create Indexes

```
✓ Create Index:
- Frequently queried columns
- WHERE clause columns
- JOIN columns
- ORDER BY columns

✗ Avoid Index:
- Rarely queried columns
- Frequently updated columns
- Small tables
- Columns with low cardinality

Trade-off:
Fast Reads ↔ Slow Writes
```

---

## Query Execution Flow

### Search Query Example

```
Query:
SELECT * FROM Employee WHERE Employee_ID = 35;

Clustered Index on Employee_ID
```

**Step-by-Step:**

```
Step 1: Load Index Pages
┌─────────────────────────────────┐
│ Index Page Mapping:             │
│ Index Page 1 → Block 10         │
│ Index Page 2 → Block 11         │
│ Index Page 3 → Block 12         │
└─────────────────────────────────┘

Load index pages from disk to memory

Step 2: Traverse B+ Tree
      ┌────┐
      │ 25 │
      └────┘
      /    \
     ...   ...
           /  \
        ┌────┐ ┌────┐
        │ 35 │ │ 50 │
        └────┘ └────┘
          ↓
       Page 5

Found: 35 → Points to Page 5

Step 3: Find Data Block
┌─────────────────────────────────┐
│ Data Page Mapping:              │
│ Page 5 → Block 25               │
└─────────────────────────────────┘

Step 4: Load Data Block
Load Block 25 from disk to memory

Step 5: Read Data
Extract Page 5 from Block 25
Find row with Employee_ID = 35
Return data
```

**Visual:**

```
┌──────────────┐
│ Index Pages  │ ← Step 1: Load
└──────────────┘
       ↓
┌──────────────┐
│  B+ Tree     │ ← Step 2: Traverse
│  Traversal   │    Find Page 5
└──────────────┘
       ↓
┌──────────────┐
│ Page Mapping │ ← Step 3: Page 5 → Block 25
└──────────────┘
       ↓
┌──────────────┐
│  Data Block  │ ← Step 4: Load Block 25
└──────────────┘
       ↓
┌──────────────┐
│  Read Data   │ ← Step 5: Extract row
└──────────────┘
```

---

### Without Index (Linear Search)

```
Query:
SELECT * FROM Employee WHERE Employee_ID = 35;

No Index on Employee_ID

Step 1: Load Page 1
Check all rows in Page 1
Employee_ID = 35? No

Step 2: Load Page 2
Check all rows in Page 2
Employee_ID = 35? No

Step 3: Load Page 3
...

Step 100: Load Page 100
Check all rows in Page 100
Employee_ID = 35? Yes! Found

Time Complexity: O(n)
100 pages loaded!
```

---

## Summary

### Data Storage Hierarchy

```
Logical Table
     ↓
Data Pages (8 KB)
├─ Header (96 bytes)
├─ Data Records (8060 bytes)
└─ Offset Array (36 bytes)
     ↓
Data Blocks (4-32 KB)
     ↓
Physical Disk
```

### B+ Tree

```
Purpose: Fast searching (O(log n))
Structure:
- Root/Intermediate: Navigation
- Leaf: Actual index values (sorted)
- Leaf nodes linked

Used by DBMs to:
- Determine which page to insert
- Fast search
- Maintain sorted order
```

### Clustered Index

```
Definition: Row order = Index order
Rules:
- Only ONE per table
- Primary key by default
- Uses offset array for ordering

Controls:
- Physical row ordering
- Data page organization
```

### Non-Clustered Index

```
Definition: Secondary index
Rules:
- MULTIPLE per table
- Does NOT control row ordering

Overhead:
- Additional B+ Trees
- Index pages
- Maintenance cost
```

### Index Pages

```
B+ Trees stored in Index Pages
Index Pages stored in Data Blocks
DBMs maintains mapping:
- Index Page → Data Block
- Data Page → Data Block
```

---

## Interview Tips

### Common Questions

**Q1: "How is table data physically stored in a database?"**

```
Answer:
"Table is a logical representation. Physically, data is stored in DATA PAGES.

Data Page Structure (8 KB):
- Header (96 bytes): Page number, free space, checksum
- Data Records (8060 bytes): Actual rows
- Offset Array (36 bytes): Pointers to rows

Example:
Row size: 64 bytes
Rows per page: 8060 / 64 = 125 rows

Data pages are stored in DATA BLOCKS on disk.
DBMs maintains Page → Block mapping.

Key Point: DBMs controls pages, storage system controls blocks."
```

**Q2: "What is the difference between clustered and non-clustered index?"**

```
Answer:
"Clustered Index:
- Controls physical row ordering
- Only ONE per table
- Primary key by default
- Uses offset array to maintain order
- Faster for range queries

Non-Clustered Index:
- Does NOT control row ordering
- MULTIPLE per table
- Secondary indexes
- Additional B+ Tree overhead
- Faster for specific lookups

Example:
Table: Employee_ID (PK), Name

Clustered (Employee_ID):
- Rows ordered: 1, 2, 3, 4, 5
- Offset: [0→1, 1→2, 2→3, 3→4, 4→5]

Non-Clustered (Name):
- Separate B+ Tree
- Points to data pages
- No row reordering"
```

**Q3: "Why can there be only one clustered index?"**

```
Answer:
"Physical rows can be ordered in only ONE way.

Example:
Employee_ID: 1, 2, 4, 5
Name: A, B, C, D

Cannot order rows by:
- Employee_ID (1, 2, 4, 5) AND
- Name (A, B, C, D) simultaneously

Clustered index uses offset array to maintain order.
One offset array = One ordering.

Non-clustered indexes don't control ordering,
so we can have multiple."
```

**Q4: "What happens when you insert a row?"**

```
Answer:
"Step 1: Traverse B+ Tree
- Find logical position for new row
- Identify target data page

Step 2: Load Data Page
- Check page mapping (Page → Block)
- Load page from disk to memory

Step 3: Insert Row
- If space available: Insert
- If page full: Page Splitting
  - Create new page
  - Redistribute rows
  - Update B+ Tree pointers
  - Update page mapping

Step 4: Update Indexes
- Update clustered index B+ Tree
- Update all non-clustered index B+ Trees

Example:
Insert Employee_ID = 35
- Traverse B+ Tree: 35 between 30 and 40
- Find: 30 in Page 5
- Load Page 5
- Insert 35 in Page 5
- Update B+ Tree pointer"
```

**Q5: "Why not create indexes on all columns?"**

```
Answer:
"Overhead of multiple indexes:

1. Memory Overhead:
   - Each index = Separate B+ Tree
   - 1 million rows = 1 million nodes per index
   - Stored in index pages
   - Consumes disk space

2. Maintenance Overhead:
   - Insert: Update ALL B+ Trees
   - Delete: Update ALL B+ Trees
   - Update: Update ALL affected B+ Trees

Example:
5 indexes on 1 million row table:
- Insert 1 row = 5 B+ Tree updates
- Possible page splits in each
- Slow writes

Trade-off:
- More indexes = Faster reads, Slower writes
- Fewer indexes = Slower reads, Faster writes

Best Practice:
- Index frequently queried columns
- Avoid indexing frequently updated columns"
```

**Q6: "Explain B+ Tree and why it's used for indexing"**

```
Answer:
"B+ Tree = Balanced tree for fast searching

Properties:
- M-order: Max M children per node
- Sorted data
- All leaves at same level
- Time Complexity: O(log n)

Structure (3-order):
- Max 2 keys per node
- Max 3 children per node

Example:
Insert: 9, 33, 75, 41, 98

      ┌──────────┐
      │ 33 | 75  │
      └──────────┘
      /    |    \
┌─────┐ ┌────┐ ┌────┐
│  9  │ │ 41 │ │ 98 │
└─────┘ └────┘ └────┘

Search 41:
- Start root: 41 > 33, 41 < 75
- Go middle child
- Found: 41

Steps: 2 (vs 5 linear search)

B+ Tree Advantage:
- Leaf nodes linked (range queries)
- Leaf nodes have actual values
- Root/Intermediate for navigation

Why for indexing:
- 1 million rows: ~20 steps (vs 1 million linear)
- Sorted order maintained
- Efficient inserts/deletes"
```

**Q7: "What is page splitting?"**

```
Answer:
"Page splitting occurs when inserting into a full page.

Scenario:
Page capacity: 3 rows
Current: [17, 19, 25] (FULL)
Insert: 30

Process:
1. Page full, cannot insert
2. Create new page
3. Redistribute rows:
   - Page 1: [17, 19]
   - Page 2: [25, 30]
4. Update B+ Tree pointers:
   - 17 → Page 1
   - 19 → Page 1
   - 25 → Page 2
   - 30 → Page 2
5. Update page mapping:
   - Page 2 → Block X

Impact:
- New page created
- Disk I/O for new block
- B+ Tree restructuring
- Pointer updates

Expensive operation!
Frequent splits = Performance degradation"
```

### Key Points to Remember

```
1. Data Storage:
   Logical Table → Data Pages → Data Blocks → Disk

2. Data Page: 8 KB
   - Header: 96 bytes
   - Data Records: 8060 bytes
   - Offset Array: 36 bytes

3. B+ Tree: O(log n) search
   - Sorted, balanced
   - Leaf nodes linked

4. Clustered Index:
   - ONE per table
   - Controls row ordering
   - Primary key default

5. Non-Clustered Index:
   - MULTIPLE per table
   - Secondary indexes
   - Overhead

6. Index Trade-off:
   Fast Reads ↔ Slow Writes
```

### Do's ✅

**1. Explain with Examples**
```
"Clustered index example: Employee_ID orders rows 1, 2, 4, 5
Non-clustered example: Name index doesn't reorder rows"
```

**2. Mention Offset Array**
```
"Offset array maintains logical ordering for clustered index
Physical rows may be jumbled, offset provides sorted access"
```

**3. Discuss Trade-offs**
```
"More indexes = Faster reads but slower writes
Each insert updates all B+ Trees"
```

### Don'ts ❌

**1. Don't Confuse Page and Block**
```
❌ "Page and block are same"
✓ "Page managed by DBMs (8 KB), Block by storage (4-32 KB)"
```

**2. Don't Forget Overhead**
```
❌ "Create indexes on all columns"
✓ "Each index has memory and maintenance overhead"
```

**3. Don't Ignore Page Splitting**
```
❌ "Insert is simple"
✓ "Insert may cause page splitting, expensive operation"
```

---

**End of Lecture**

Database indexing is critical for performance. Understanding data pages, B+ Trees, and clustered vs non-clustered indexes is essential. Clustered index controls row ordering (one per table), non-clustered are secondary (multiple allowed). B+ Tree provides O(log n) search. Index carefully—trade-off between read speed and write overhead.

**Key Takeaway:** Data stored in pages (8 KB), pages in blocks. B+ Tree for fast search. Clustered index = row ordering (one only). Non-clustered = secondary (multiple). Index wisely—overhead exists!
