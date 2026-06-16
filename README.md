# Redis from Scratch

A complete Redis server implementation built from scratch in Python. This project includes core Redis functionality with persistence, pub/sub messaging, TTL support, and various data structures.

## Features

### Core Features
- **Full Redis Protocol Support**: RESP (REdis Serialization Protocol) implementation
- **Data Structures**: Support for strings, hashes, lists, and sets
- **Key Expiration**: TTL (Time-To-Live) support with automatic cleanup
- **Pub/Sub Messaging**: Publish-Subscribe pattern implementation
- **Multi-client Support**: Handles multiple concurrent client connections

### Persistence
- **RDB Snapshots**: Point-in-time persistence snapshots
- **AOF (Append-Only File)**: Command logging for durability
- **Automatic Recovery**: Restore data from persistence files on startup
- **Configurable Persistence**: Customizable persistence intervals and modes

### Advanced Features
- **Expiration Management**: Automatic TTL-based key cleanup
- **Command Handling**: Extensive built-in command support
- **Non-blocking I/O**: Event-driven architecture using select for efficient client handling

## Project Structure

```
redis_server/
├── command_handler.py       # Main command processing
├── server.py                # Redis server core
├── storage.py               # Data store implementation
├── pubsub.py                # Pub/Sub manager
├── response.py              # Response formatting
├── commands/                # Command implementations
│   ├── base.py              # Base command class
│   ├── basic.py             # Basic commands (GET, SET, DEL, etc.)
│   ├── hash.py              # Hash commands (HGET, HSET, etc.)
│   ├── list.py              # List commands (LPUSH, RPUSH, etc.)
│   ├── set.py               # Set commands (SADD, SMEMBERS, etc.)
│   ├── expiration.py        # TTL commands (EXPIRE, TTL, etc.)
│   ├── pubsub.py            # Pub/Sub commands (PUBLISH, SUBSCRIBE, etc.)
│   ├── persistence.py       # Persistence commands (SAVE, BGSAVE, etc.)
│   └── info.py              # Info commands (INFO, etc.)
└── persistence/             # Persistence layer
    ├── manager.py           # Persistence coordination
    ├── rdb.py               # RDB snapshot handling
    ├── aof.py               # AOF file handling
    ├── config.py            # Persistence configuration
    └── recovery.py          # Data recovery from snapshots
```

## Installation

### Requirements
- Python 3.7+
- No external dependencies required

### Setup

```bash
# Clone or download the project
cd test

# Run the server
python main.py
```

## Usage

### Starting the Server

```python
python main.py
```

The server will start listening on `localhost:6379` by default.

### Connecting as a Client

You can use any Redis client library or the redis-cli tool:

```bash
redis-cli -h localhost -p 6379
```

Or in Python:

```python
import redis

client = redis.Redis(host='localhost', port=6379, decode_responses=True)
client.set('key', 'value')
print(client.get('key'))  # Output: value
```

## Supported Commands

### String Commands
- `GET`, `SET`, `GETSET`, `APPEND`, `STRLEN`, `INCR`, `DECR`, `INCRBY`, `DECRBY`

### Hash Commands
- `HGET`, `HSET`, `HMGET`, `HMSET`, `HGETALL`, `HDEL`, `HEXISTS`, `HLEN`, `HKEYS`, `HVALS`

### List Commands
- `LPUSH`, `RPUSH`, `LPOP`, `RPOP`, `LRANGE`, `LLEN`, `LINDEX`, `LSET`, `LTRIM`

### Set Commands
- `SADD`, `SREM`, `SMEMBERS`, `SCARD`, `SISMEMBER`, `SINTER`, `SUNION`, `SDIFF`

### Key Commands
- `DEL`, `EXISTS`, `KEYS`, `EXPIRE`, `TTL`, `TYPE`

### Pub/Sub Commands
- `PUBLISH`, `SUBSCRIBE`, `UNSUBSCRIBE`, `PSUBSCRIBE`, `PUNSUBSCRIBE`

### Persistence Commands
- `SAVE`, `BGSAVE`, `LASTSAVE`, `SHUTDOWN`

### Server Commands
- `INFO`, `PING`, `ECHO`, `SELECT`, `FLUSHDB`, `FLUSHALL`

## Configuration

### Custom Configuration

You can customize the server with a custom `PersistenceConfig`:

```python
from redis_server import RedisServer
from redis_server.persistence import PersistenceConfig

config = PersistenceConfig(
    save_interval=60,        # Save every 60 seconds
    use_rdb=True,            # Enable RDB snapshots
    use_aof=True,            # Enable AOF logging
    aof_fsync='always'       # Fsync after every write
)

server = RedisServer(
    host='localhost',
    port=6379,
    persistence_config=config
)
server.start()
```

## Architecture

### Event-Driven Design
The server uses a non-blocking I/O architecture with the `select` module to handle multiple concurrent clients efficiently.

### Command Handler
The `CommandHandler` processes incoming commands, routes them to appropriate command implementations, and manages responses.

### Persistence Layer
The persistence layer automatically handles:
- Snapshot creation (RDB)
- Command logging (AOF)
- Data recovery on startup
- Background saving operations

### Pub/Sub Manager
Manages channel subscriptions and message broadcasting across multiple connected clients.

## Performance Considerations

- **Non-blocking I/O**: Handles multiple connections simultaneously without threading overhead
- **Automatic Cleanup**: Background TTL-based key cleanup prevents memory buildup
- **Configurable Persistence**: Balance between durability and performance
- **In-memory Storage**: Fast data access with no disk I/O for reads

## Limitations

- Single-threaded main event loop
- In-memory only (data loss if process terminates without persistence)
- No clustering or replication support
- No Lua scripting support

## Future Enhancements

- [ ] Redis Cluster support
- [ ] Master-Slave replication
- [ ] Lua scripting
- [ ] Transactions (MULTI/EXEC)
- [ ] Blocking operations (BLPOP, BRPOP)
- [ ] Stream data type
- [ ] HyperLogLog
- [ ] Geo commands

## Testing

Run the server and connect with a Redis client to test functionality:

```python
import redis

client = redis.Redis(host='localhost', port=6379, decode_responses=True)

# Test basic operations
client.set('name', 'Redis')
print(client.get('name'))  # Output: Redis

# Test data structures
client.hset('user:1', mapping={'name': 'John', 'age': '30'})
print(client.hgetall('user:1'))  # Output: {'name': 'John', 'age': '30'}

# Test Pub/Sub
pubsub = client.pubsub()
pubsub.subscribe('channel1')
client.publish('channel1', 'Hello World')
```

## License

This project is provided as-is for educational purposes.

## Notes

- Data is persisted to disk using RDB snapshots and/or AOF files
- Default persistence directory is the project root
- Use `Ctrl+C` to gracefully shutdown the server
- The server performs automatic cleanup of expired keys every 100ms