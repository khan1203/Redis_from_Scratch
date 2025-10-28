## Redis_from_Scratch
Here I gonna build in-memory data-store that is - Real REDIS from scratch. It is often used as a database, cache, and message broker server. 
I gonna try to add all these features in it, in sha Allah.

| Feature                | Description                                                                            |
| ---------------------- | -------------------------------------------------------------------------------------- |
| 🧩 **Data Model**      | Key–Value store (similar to a Python dictionary or JavaScript object)                  |
| ⚡ **Speed**            | Very fast — since it stores data in memory rather than on disk  |
| 💾 **Persistence**     | Optional — can save data to disk periodically (AOF / RDB modes)                        |
| 🧵 **Single-threaded** | Uses an event loop, but can handle thousands of requests per second                    |
| 🌍 **Networked**       | Clients connect via TCP — can be accessed remotely                                     |
| 🔁 **Replication**     | Supports master-replica setup for scalability                                          |
| ☁️ **Use Cases**       | Caching, sessions, queues, pub/sub messaging, rate limiting, leaderboard ranking and so on |

