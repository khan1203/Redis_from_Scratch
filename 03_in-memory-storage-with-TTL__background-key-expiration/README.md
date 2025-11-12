# 🧠 In-Memory Storage with TTL & Background Key Expiration

In this time, we extend a basic Redis-like server to support **Time-To-Live (TTL)** functionality and **background key expiration**, closely mimicking real Redis behavior within a single-threaded event loop. ⚙️

---

## 🚀 Key Features Overview

| 🔢 Feature | 🧩 Description |
|-------------|----------------|
| ⏳ **TTL Support** | Implements `EXPIRE`, `EXPIREAT`, `TTL`, `PTTL`, and `PERSIST` commands for key lifecycle management. |
| 💤 **Lazy Expiration** | Expired keys are removed **on access**, minimizing CPU overhead during idle time. |
| 🔥 **Active Expiration** | Background cleanup process periodically scans and removes expired keys. |
| 🧮 **Memory Tracking** | Real-time tracking of total memory usage for monitoring and optimization. |
| 🧰 **Enhanced Storage** | Type-aware data storage system that maintains expiration metadata alongside the key-value data. |

---

## 🧱 Core Concepts

### 🕐 Time-To-Live (TTL)
Each key can have a **TTL**—a countdown (in seconds or milliseconds) after which the key will automatically expire and be deleted.

### ⚙️ Expiration Strategies
Redis-inspired expiration techniques:
- **Lazy Expiration** → Check key expiration **on read/write**.
- **Active Expiration** → Periodic background task removes expired keys to free memory proactively.

### 💾 Memory Awareness
The system continuously monitors:
- Total memory used
- Number of active vs expired keys
- Efficiency of expiration mechanisms

---

## 🧑‍💻 Commands Implemented

| 🧠 Command | 📝 Description |
|-------------|----------------|
| `EXPIRE key seconds` | Set TTL in seconds for a key |
| `EXPIREAT key timestamp` | Set key to expire at a specific UNIX timestamp |
| `TTL key` | Get remaining TTL in seconds |
| `PTTL key` | Get remaining TTL in milliseconds |
| `PERSIST key` | Remove the TTL, making the key persistent |

---

## 🧩 Architecture Notes

- Built on a **single-threaded event loop**, simulating Redis-style concurrency.
- Utilizes **non-blocking operations** for active expiration tasks.
- Integrates expiration logic directly into command handling and event scheduling.

---

## 📈 Future Enhancements
- Add support for **per-key statistics** 📊  
- Implement **LRU-based eviction policy** 🧹  
- Extend to support **clustered key expiration** ⚡  

---

## 🧠 Summary
This repo provides a deep dive into **how Redis manages expiring data** efficiently while remaining single-threaded. You’ll gain hands-on experience with **TTL mechanisms**, **event-driven expiration**, and **memory tracking** — all essential for building performant in-memory systems.

---

💡 *Inspired by Redis’s elegant design philosophy — “Simplicity, Performance, and Control.”*
