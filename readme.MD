Each new `SET` replaces the previous value for the same key when replayed.

### Persistence
- All writes go to `data.db`.
- Each write is **flushed and fsynced** to guarantee on-disk durability.
- On restart, the system replays `data.db` to rebuild the in-memory key–value map.

### CLI Interface
Commands are read from `STDIN` and results printed to `STDOUT`.

| Command | Description | Example |
|----------|--------------|----------|
| `SET <key> <value>` | Stores a key with value | `SET user Dhiresh` |
| `GET <key>` | Retrieves the value for a key | `GET user` → `Dhiresh` |
| `EXIT` | Exits the program | `EXIT` |

### Implementation Highlights
- Uses an **append-only file** for storage (`_atomic_append()`).
- Uses a **Python dictionary** rebuilt from log replay (`_load_log()`).
- Includes safe key validation and corruption handling.
- Supports silent error recovery and atomic updates.

---

## 🧩 File Structure

