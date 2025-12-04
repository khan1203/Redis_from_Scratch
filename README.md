# Redis from Scratch: A Complete Implementation Guide

## 🚀 Overview

Welcome to **Redis from Scratch**, a comprehensive implementation of a Redis-like in-memory database server built from the ground up. This project demonstrates how Redis works internally by implementing core features including single-threaded event loop architecture, key-value storage with TTL, persistence mechanisms (AOF and RDB), and a publish/subscribe messaging system.

## 📊 Features Implemented

### 🧠 Core Architecture
- **Single-threaded event loop** with non-blocking I/O
- **Connection multiplexing** using `select()` system call
- **Redis Serialization Protocol (RESP)** for client communication
- **Command pipelining** support

### 💾 Data Storage
- **In-memory key-value store** with Python dictionaries
- **Time-To-Live (TTL)** support for automatic key expiration
- **Hybrid expiration strategy**: lazy + active cleanup
- **Memory tracking** with real-time usage monitoring

### 🔄 Persistence
- **Append-Only File (AOF)** for command-level durability
- **RDB snapshots** for point-in-time backups
- **Configurable sync policies** (always, everysec, no)
- **Background rewriting** for AOF compaction

### 📢 Messaging
- **Publish/Subscribe system** with channel-based messaging
- **Fire-and-forget delivery** model
- **Pattern subscriptions** support
- **Channel management** commands

### 🛠️ Commands Supported
- **Basic**: SET, GET, DEL, EXISTS, KEYS, FLUSHALL, TYPE
- **TTL**: EXPIRE, EXPIREAT, TTL, PTTL, PERSIST
- **Persistence**: SAVE, BGSAVE, LASTSAVE, BGREWRITEAOF
- **Pub/Sub**: SUBSCRIBE, UNSUBSCRIBE, PUBLISH, PUBSUB
- **Utility**: PING, ECHO, INFO, QUIT

## 🏗️ System Architecture

<img width="1271" height="509" alt="Screenshot from 2025-12-04 10-49-52" src="https://github.com/user-attachments/assets/7fd6eb8f-8c9d-40d4-b177-10cdc30e1861" />


## 📁 Project Structure

```
redis_from_scratch/
├── main.py                          # Entry point
├── redis_server/
│   ├── __init__.py
│   ├── server.py                    # Main event loop and networking
│   ├── command_handler.py           # Command routing and execution
│   ├── storage.py                   # In-memory data store with TTL
│   ├── pubsub.py                    # Publish/Subscribe implementation
│   ├── response.py                  # RESP protocol formatter
│   ├── commands/                    # Command implementations
│   │   ├── basic.py                 # SET, GET, DEL, etc.
│   │   ├── expiration.py            # TTL commands
│   │   ├── persistence.py           # SAVE, BGSAVE, etc.
│   │   ├── pubsub.py                # Pub/Sub commands
│   │   └── ...
│   └── persistence/                 # Persistence layer
│       ├── manager.py               # Persistence orchestration
│       ├── aof.py                   # AOF implementation
│       ├── rdb.py                   # RDB implementation
│       ├── config.py                # Configuration management
│       └── recovery.py              # Data recovery
└── data/                            # Persistence files (created at runtime)
```

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- `telnet` (for testing)
- Basic understanding of Redis commands

### Installation

1. Clone the repository:
```bash
git clone https://github.com/poridhioss/Redis_from_scratch.git
cd Redis_from_scratch
```

2. Install dependencies (if any):
```bash
pip install -r requirements.txt
```

### Running the Server

Start the Redis server:
```bash
python main.py
```

You should see output similar to:
```
Single-threaded Redis server listening on localhost:6379...
Data recovered from persistence files (if any).
Ready to accept connections.
```

## 🔧 Testing the Implementation

### Basic Connectivity Test

1. Open a new terminal and connect using telnet:
```bash
telnet localhost 6379
```

2. You should receive a welcome message:
```
+OK Server ready
```

3. Test basic commands:
```
PING
+OK

SET mykey "Hello Redis"
+OK

GET mykey
$11
Hello Redis
```

### Testing Key Expiration (TTL)

1. Set a key with expiration:
```
SET session:user123 "data" EX 10
+OK

TTL session:user123
:8  # Remaining seconds

# Wait 10 seconds...
GET session:user123
$-1  # Key expired
```

2. Use EXPIREAT with a specific timestamp:
```
SET temp "value"
+OK

EXPIREAT temp 1893456000  # Unix timestamp
:1

TTL temp
:31536000  # 1 year from now
```

### Testing Persistence (AOF)

1. Enable AOF (if not already enabled in config):
```
CONFIG SET appendonly yes
+OK
```

2. Perform write operations:
```
SET persistent:1 "data1"
SET persistent:2 "data2"
DEL persistent:1
```

3. Check AOF file:
```bash
cat data/appendonly.aof
```

4. Restart server and verify data recovery:
```bash
# Stop server with Ctrl+C
python main.py
# Reconnect with telnet
GET persistent:2
$5
data2
```

### Testing RDB Snapshots

1. Create manual snapshot:
```
SAVE
+OK  # Synchronous save

BGSAVE
+Background saving started  # Asynchronous save
```

2. Check RDB file:
```bash
ls -la data/dump.rdb
```

3. Test automatic snapshots (configured in `persistence/config.py`):
```bash
# Modify many keys quickly to trigger automatic save
```

### Testing Pub/Sub System

**Terminal 1 (Subscriber):**
```bash
telnet localhost 6379
SUBSCRIBE news alerts
```

**Terminal 2 (Subscriber 2):**
```bash
telnet localhost 6379
SUBSCRIBE news
```

**Terminal 3 (Publisher):**
```bash
telnet localhost 6379
PUBLISH news "Breaking news!"
:2  # Delivered to 2 subscribers

PUBSUB CHANNELS
*2
$4
news
$6
alerts
```

### Testing Multiple Clients Concurrently

Open multiple terminal windows and connect simultaneously:
```bash
# Terminal 1
telnet localhost 6379
SET counter 0
INCR counter

# Terminal 2
telnet localhost 6379
GET counter
:1

# Terminal 3
telnet localhost 6379
INCR counter
GET counter
:2
```

### Testing Command Pipelining

Send multiple commands at once:
```bash
echo -e "SET a 1\nSET b 2\nGET a\nGET b" | telnet localhost 6379
```

## ⚙️ Configuration

Configuration is managed in `redis_server/persistence/config.py`. Key settings:

```python
DEFAULT_CONFIG = {
    'appendonly': True,               # Enable AOF
    'appendfsync': 'everysec',        # Sync policy: always/everysec/no
    'rdb_enabled': True,              # Enable RDB snapshots
    'rdb_filename': 'dump.rdb',       # RDB file name
    'rdb_save_conditions': [          # Automatic save triggers
        (900, 1),     # 1 change in 15 minutes
        (300, 10),    # 10 changes in 5 minutes
        (60, 10000),  # 10000 changes in 1 minute
    ],
    'aof_rewrite_percentage': 100,    # Rewrite when 100% bigger
    'aof_rewrite_min_size': 64,       # Minimum size in MB
}
```

## 🔍 Implementation Details

### Single-Threaded Event Loop

The core of the system is a single-threaded event loop using Python's `select()` for I/O multiplexing:

```python
def _event_loop(self):
    while self.running:
        # Monitor sockets for activity
        readable, _, _ = select.select(
            [self.server_socket] + list(self.clients.keys()),
            [], [], 0.05  # 50ms timeout
        )
        
        # Handle new connections
        if self.server_socket in readable:
            self._accept_new_connection()
        
        # Handle client data
        for client_socket in readable:
            if client_socket != self.server_socket:
                self._handle_client_data(client_socket)
        
        # Background tasks (every 100ms)
        if time.time() - self.last_background_run >= 0.1:
            self._run_background_tasks()
```

### Hybrid Expiration Strategy

1. **Lazy Expiration**: Keys are checked for expiration on every access
2. **Active Expiration**: Background thread samples and removes expired keys

```python
def _is_key_valid(self, key):
    """Check if key exists and hasn't expired"""
    if key not in self._data:
        return False
    
    value, _, expiry_time = self._data[key]
    if expiry_time and expiry_time <= time.time():
        # Key expired - remove it
        self._remove_expired_key(key, value)
        return False
    
    return True
```

### AOF Implementation

AOF logs every write command to disk:

```python
def log_command(self, command, *args):
    """Log write command to AOF file"""
    if command.upper() not in WRITE_COMMANDS:
        return
    
    # Format: timestamp COMMAND args
    timestamp = int(time.time())
    line = f"{timestamp} {command.upper()} {' '.join(args)}\n"
    
    # Write to buffer
    self.buffer.write(line.encode('utf-8'))
    
    # Sync based on policy
    if self.sync_policy == 'always':
        self._fsync()
```

### RDB Implementation

RDB creates binary snapshots of the entire dataset:

```python
def save_rdb(self, data_store, filename):
    """Save current dataset to RDB file"""
    temp_filename = filename + '.tmp'
    
    with open(temp_filename, 'wb') as f:
        # Write magic header
        f.write(b'REDIS0001')
        
        # Serialize all keys with their data
        for key, (value, data_type, expiry) in data_store.items():
            self._write_key(f, key, value, data_type, expiry)
        
        # Write checksum
        if self.config['rdb_checksum']:
            f.write(self._calculate_checksum())
    
    # Atomic rename
    os.rename(temp_filename, filename)
```

### Pub/Sub Implementation

Channel-based messaging with fire-and-forget delivery:

```python
class PubSubManager:
    def __init__(self):
        self.channels = defaultdict(set)  # channel -> set of clients
        self.client_subscriptions = defaultdict(set)  # client -> set of channels
    
    def publish(self, channel, message):
        """Publish message to all subscribers of a channel"""
        subscribers = self.channels.get(channel, set())
        delivered = 0
        
        for client in subscribers:
            try:
                # Build RESP response: ["message", channel, message]
                response = ResponseBuilder.array([
                    ResponseBuilder.bulk_string("message"),
                    ResponseBuilder.bulk_string(channel),
                    ResponseBuilder.bulk_string(message)
                ])
                client.send(response)
                delivered += 1
            except:
                # Client disconnected
                self._cleanup_client(client)
        
        return delivered
```

## 📊 Performance Characteristics

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| GET/SET | O(1) | Hash table lookup |
| KEYS | O(n) | Scans all keys |
| EXPIRE | O(1) | Updates expiration metadata |
| PUBLISH | O(n) | n = number of subscribers |
| AOF append | O(1) | Appends to file buffer |
| RDB save | O(n) | n = number of keys |

## 🧪 Testing Scenarios

### Scenario 1: Cache with Expiration
```bash
# Set cache with 5-second TTL
SET cache:user:123 "{'name': 'John', 'age': 30}" EX 5

# Use cache
GET cache:user:123

# Cache automatically expires after 5 seconds
```

### Scenario 2: Session Management
```bash
# Create session
SET session:abc123 "{'user_id': 456, 'logged_in': true}" EX 3600

# Check session
TTL session:abc123
:3598

# Refresh session
EXPIRE session:abc123 3600
```

### Scenario 3: Real-time Notifications
```bash
# Client 1 subscribes
SUBSCRIBE notifications:user:789

# Client 2 publishes
PUBLISH notifications:user:789 "You have a new message!"
```

### Scenario 4: Data Persistence Workflow
```bash
# Enable both persistence mechanisms
CONFIG SET appendonly yes
CONFIG SET save "900 1 300 10 60 10000"

# Perform critical operations
SET order:1001 "confirmed"
SET payment:1001 "processed"

# Force save
SAVE

# Verify persistence
INFO persistence
```

## 🔧 Troubleshooting

### Common Issues

1. **"Connection refused" error**:
   - Ensure server is running: `python main.py`
   - Check port 6379 is not blocked

2. **Commands not recognized**:
   - Check command spelling (Redis commands are case-insensitive)
   - Ensure you're using supported commands

3. **Memory usage high**:
   - Check for keys without TTL
   - Use `INFO memory` to analyze usage
   - Consider implementing eviction policies

4. **Persistence files not created**:
   - Verify write permissions in `data/` directory
   - Check configuration settings

### Debug Mode

Enable verbose logging by modifying `server.py`:
```python
class RedisServer:
    def __init__(self, debug=False):
        self.debug = debug
        # ...
    
    def _log(self, message):
        if self.debug:
            print(f"[DEBUG] {message}")
```

## 📚 Learning Resources

### Key Concepts to Explore
1. **I/O Multiplexing**: How `select()` enables single-threaded concurrency
2. **Copy-on-Write**: RDB snapshot mechanism
3. **Redis Protocol (RESP)**: Client-server communication format
4. **Memory Management**: Python's dict implementation and memory overhead
5. **Persistence Trade-offs**: AOF vs RDB strengths and weaknesses

### Extending the Project

1. **Add new data types**: Lists, Sets, Sorted Sets, Hashes
2. **Implement transactions**: MULTI/EXEC commands
3. **Add replication**: Master-slave replication
4. **Implement clustering**: Sharding across multiple instances
5. **Add Lua scripting**: Embedded script execution

## 🎯 Conclusion

This Redis-from-scratch implementation demonstrates the core principles of in-memory databases:

1. **Simplicity**: Single-threaded design eliminates concurrency issues
2. **Performance**: Memory-based operations with O(1) complexity
3. **Durability**: Multiple persistence strategies for different use cases
4. **Extensibility**: Modular architecture for adding new features

By understanding and experimenting with this implementation, you'll gain deep insights into how production systems like Redis are built and optimized.

---

