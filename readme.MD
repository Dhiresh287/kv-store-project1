# 🧱 Persistent Key–Value Store (Project-1) — PostgreSQL-Style Documentation

## 1. Overview
This repository implements a **persistent append-only key–value store**, conceptually similar to how PostgreSQL uses its **WAL (Write-Ahead Log)** for durability.  
All operations (`SET`, `GET`, `EXIT`) are handled via a **command-line interface (CLI)** that interacts with an append-only log file `data.db`.

The project is designed as an introductory exercise in **database storage engine design**, persistence, and recovery mechanisms.

---

## 2. Architecture & Design

### 2.1 Data Model
Unlike a relational schema, the store has a single logical table:

| Column | Type | Description |
|---------|------|-------------|
| `key`   | TEXT | Primary identifier |
| `value` | TEXT | Value associated with key |

Each `SET` operation appends a record to `data.db` in plain-text form:

When the system restarts, the log is replayed to rebuild in-memory state, ensuring **last-write-wins** semantics.

---

### 2.2 Durability & WAL Analogy
This KV store mimics the **WAL (Write-Ahead Logging)** principle used in PostgreSQL:

| PostgreSQL Component | KVStore Equivalent |
|----------------------|--------------------|
| WAL Segment Files    | `data.db` append-only log |
| `fsync()` on COMMIT  | `_atomic_append()` flush & fsync per SET |
| Checkpoint / Replay  | `_load_log()` to rebuild state |
| Shared Buffer Cache  | In-memory Python dictionary `_kv` |

This approach guarantees data persistence even if the process or system crashes mid-operation.

---

## 3. Implementation

### 3.1 Core Source Files

### 3.2 Important Classes and Functions

| Name | Description |
|------|--------------|
| `KeyValueStore` | Core class handling `set()` and `get()` with persistence |
| `_atomic_append()` | Appends log entries and forces flush via `os.fsync()` |
| `_load_log()` | Replays existing `data.db` to restore latest state |
| `run_cli()` | Command loop for reading STDIN and writing STDOUT |
| `_validate_key()` | Ensures key format and length validity |

---

## 4. CLI Interface

### 4.1 Commands
| Command | Description | Example |
|----------|--------------|----------|
| `SET <key> <value>` | Insert or update a key’s value | `SET user Dhiresh` |
| `GET <key>` | Retrieve the current value | `GET user` → `Dhiresh` |
| `EXIT` | Terminate the program | `EXIT` |

### 4.2 Example Session
✅ The value persists because the append-only log is replayed.

---

## 5. Persistence Logic (SQL-style Explanation)
Below is an SQL-style pseudocode representation of how operations behave:

```sql
-- Append operation (SET)
INSERT INTO kv_log (ts, operation, key, value)
VALUES (NOW(), 'SET', 'username', 'Alice');

-- Replay phase (startup)
SELECT DISTINCT ON (key) key, LAST(value) OVER (PARTITION BY key ORDER BY ts DESC)
FROM kv_log;
