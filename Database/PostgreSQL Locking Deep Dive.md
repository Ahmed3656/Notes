#### **Table-Level Locks**
PostgreSQL has 8 table-level locks. They differ in strength and what other locks they conflict with.

- **ACCESS SHARE**
    - Acquired by: `SELECT`, `COPY TO`
    - Purpose: prevents schema changes while reading
    - Conflicts only with: **ACCESS EXCLUSIVE**
- **ROW SHARE**
    - Acquired by: `SELECT ... FOR UPDATE/SHARE`
    - Purpose: reading rows while marking them for possible modification
    - Conflicts with: **EXCLUSIVE**, **ACCESS EXCLUSIVE**
- **ROW EXCLUSIVE**
    - Acquired by: `INSERT`, `UPDATE`, `DELETE`
    - Purpose: normal writes
    - Conflicts with: **SHARE**, **SHARE ROW EXCLUSIVE**, **EXCLUSIVE**, **ACCESS EXCLUSIVE**
- **SHARE UPDATE EXCLUSIVE**
    - Acquired by: `VACUUM`, `ANALYZE`, `CREATE INDEX CONCURRENTLY`
    - Purpose: lightweight maintenance lock
    - Conflicts with: **itself**, **SHARE**, **SHARE ROW EXCLUSIVE**, **EXCLUSIVE**, **ACCESS EXCLUSIVE**
- **SHARE**
    - Acquired by: `CREATE INDEX`
    - Purpose: index build while allowing reads
    - Conflicts with: **ROW EXCLUSIVE**, **SHARE UPDATE EXCLUSIVE**, **SHARE ROW EXCLUSIVE**, **EXCLUSIVE**, **ACCESS EXCLUSIVE**
- **SHARE ROW EXCLUSIVE**
    - Acquired by: `CREATE TRIGGER`
    - Purpose: allows reads but blocks most writes
    - Conflicts with: **ROW EXCLUSIVE**, **SHARE**, **SHARE ROW EXCLUSIVE**, **EXCLUSIVE**, **ACCESS EXCLUSIVE**
- **EXCLUSIVE**
    - Acquired by: `REFRESH MATERIALIZED VIEW CONCURRENTLY`
    - Purpose: blocks writes but allows plain reads
    - Conflicts with: **ROW SHARE**, **ROW EXCLUSIVE**, **SHARE UPDATE EXCLUSIVE**, **SHARE**, **SHARE ROW EXCLUSIVE**, **EXCLUSIVE**, **ACCESS EXCLUSIVE**
- **ACCESS EXCLUSIVE**
    - Acquired by: `DROP TABLE`, `TRUNCATE`, `VACUUM FULL`, schema changes
    - Purpose: strongest table lock, blocks everything
    - Conflicts with: **all other locks**

#### Conflict Matrix

(✓ = compatible, ✗ = conflict)

| Lock Type                  | Access Share | Row Share | Row Exclusive | Share Update Exclusive | Share | Share Row Exclusive | Exclusive | Access Exclusive |
| -------------------------- | ------------ | --------- | ------------- | ---------------------- | ----- | ------------------- | --------- | ---------------- |
| **Access Share**           | ✓            | ✓         | ✓             | ✓                      | ✓     | ✓                   | ✓         | ✗                |
| **Row Share**              | ✓            | ✓         | ✓             | ✓                      | ✓     | ✓                   | ✗         | ✗                |
| **Row Exclusive**          | ✓            | ✓         | ✓             | ✗                      | ✗     | ✗                   | ✗         | ✗                |
| **Share Update Exclusive** | ✓            | ✓         | ✗             | ✗                      | ✗     | ✗                   | ✗         | ✗                |
| **Share**                  | ✓            | ✓         | ✗             | ✗                      | ✓     | ✗                   | ✗         | ✗                |
| **Share Row Exclusive**    | ✓            | ✓         | ✗             | ✗                      | ✗     | ✗                   | ✗         | ✗                |
| **Exclusive**              | ✓            | ✗         | ✗             | ✗                      | ✗     | ✗                   | ✗         | ✗                |
| **Access Exclusive**       | ✗            | ✗         | ✗             | ✗                      | ✗     | ✗                   | ✗         | ✗                |

---

#### **Row-Level Locks**
Row-level locks protect individual tuples and allow high concurrency. They follow a strength hierarchy:
**FOR UPDATE > FOR NO KEY UPDATE > FOR SHARE > FOR KEY SHARE**

- **FOR UPDATE**
    - Acquired by:  `UPDATE` (on **unique** key columns), `DELETE`, `SELECT FOR UPDATE`
    - Strongest row lock; prevents other updates/deletes
- **FOR NO KEY UPDATE**
    - Acquired by: `UPDATE` on **non-unique** columns, `SELECT FOR NO KEY UPDATE`
    - Weaker than `FOR UPDATE`; allows `FOR KEY SHARE`
- **FOR SHARE**
    - Acquired by: `SELECT ... FOR SHARE`
    - Allows concurrent readers but blocks writers
- **FOR KEY SHARE**
    - Acquired by: `SELECT ... FOR KEY SHARE`, **foreign key checks**
    - Weakest row lock; allows most concurrent reads

**Note:** Row locks never block plain `SELECT` queries. They only conflict with other row-locking operations (`FOR UPDATE`, `FOR SHARE`, etc.).

#### Storage
- Row locks are **not kept in a lock table** but stored in each tuple’s **`xmax` field**.
- This makes them very lightweight compared to table locks.

#### Row Lock Conflict Matrix

(✓ = compatible, ✗ = conflict)

| Lock Type          | FOR KEY SHARE | FOR SHARE | FOR NO KEY UPDATE | FOR UPDATE |
| ------------------ | ------------- | --------- | ----------------- | ---------- |
| **FOR KEY SHARE**  | ✓             | ✓         | ✓                 | ✗          |
| **FOR SHARE**      | ✓             | ✓         | ✗                 | ✗          |
| **FOR NO KEY UPDATE** | ✓          | ✗         | ✗                 | ✗          |
| **FOR UPDATE**     | ✗             | ✗         | ✗                 | ✗          |

---

#### **PostgreSQL-Specific Behaviors**
- **Weak Locks:** ACCESS SHARE, ROW SHARE, ROW EXCLUSIVE use fast-path (max 16 per backend)
- **Advisory Locks:** Application-controlled locks on integer values, not tied to specific data

#### **MVCC Implementation in PostgreSQL**
PostgreSQL uses MVCC to give each transaction a **consistent snapshot** of the database without blocking readers/writers.

- **Tuple metadata:**
    - `xmin` = transaction ID that created the tuple
    - `xmax` = transaction ID that deleted/invalidated the tuple
- **Snapshot isolation:**
    - Each transaction sees data as of its snapshot.
    - Readers don’t block writers, and writers don’t block readers (except for conflicts).
- **HOT updates (Heap-Only Tuples):**
    - If an update doesn’t touch indexed columns, PostgreSQL can avoid index rewrites by chaining versions inside the heap.
    - **Requires:** `fillfactor < 100` and enough free space on page.
- **VACUUM:**
    - Reclaims dead tuples and prevents table bloat.
    - Required because old versions stay until no active transaction can see them.
    - **Autovacuum** runs automatically in the background.

#### **Transaction ID Wraparound**
PostgreSQL transaction IDs are **32-bit integers** (~4 billion transactions before cycling).

- **The Problem:** old tuples might appear “in the future” after wraparound —> corruption risk.
- **The Safeguard:** PostgreSQL shuts down to protect data if wraparound risk is detected.
- **The Solution:**
    - `VACUUM` marks old tuples as “frozen”
    - **Autovacuum** automatically performs wraparound protection on aging tables
    - Monitoring `pg_stat_all_tables` helps identify tables at risk
