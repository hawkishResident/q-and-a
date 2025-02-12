# Technical Interview Answer Guide

## Node.js Core Concepts

### What is Node.js?
- A JavaScript runtime built on Chrome's V8 JavaScript engine
- Uses an event-driven, non-blocking I/O model
- Perfect for data-intensive real-time applications
- Single-threaded with event loop, but uses libuv for thread pool operations

### Node.js Bottlenecks

1. CPU-intensive Operations:
   - Heavy computations block the event loop
   - Complex calculations (crypto, compression)
   - JSON parsing with large payloads
   - Regular expressions on large strings
   Solutions:
   - Worker Threads for parallel processing
   - Break large computations into smaller chunks
   - Offload to microservices
   - Use Node.js clustering to utilize multiple cores

2. Memory Issues:
   - Memory leaks from event listeners not being removed
   - Keeping large data in variables instead of streams
   - Improper caching strategies (infinite caching)
   - Large session data in memory
   - Buffer operations without size limits
   Solutions:
   - Implement proper cleanup of event listeners
   - Use streams for large data processing
   - Implement cache expiration
   - Monitor memory usage with tools like heapdump
   - Set buffer size limits

3. Blocking Operations:
   - Synchronous file operations (readFileSync)
   - Synchronous crypto operations
   - Large synchronous loops
   - Direct database queries without pooling
   Solutions:
   - Use async/await and Promises
   - Implement connection pooling
   - Use streaming APIs
   - Break large operations into chunks

4. Poor Database Handling:
   - Too many open connections
   - Not using connection pooling
   - Unoptimized queries
   - Large result sets loaded in memory
   Solutions:
   - Implement connection pooling
   - Use pagination
   - Optimize queries
   - Stream large result sets

5. Network Related:
   - Too many concurrent connections
   - Large payload sizes
   - Unhandled socket timeouts
   - Keep-alive issues
   Solutions:
   - Implement rate limiting
   - Use compression
   - Proper error handling
   - Connection timeouts

6. Event Loop Blocking:
   - Long running callbacks
   - Nested callbacks (callback hell)
   - Unhandled Promise rejections
   - Recursive operations without breaks
   Solutions:
   - Use async/await
   - Implement proper error handling
   - Break long operations into smaller tasks
   - Use setImmediate() to yield to event loop

### Event Loop, Micro/Macro Tasks
Event Loop Phases:
1. timers (setTimeout, setInterval)
2. pending callbacks
3. idle, prepare
4. poll (I/O)
5. check (setImmediate)
6. close callbacks

Microtasks:
- Process.nextTick()
- Promise callbacks
- queueMicrotask()
Execute between each phase

Macrotasks by Event Loop Phase:

1. Timers Phase:
   - setTimeout() callbacks
   - setInterval() callbacks
   - Timing functions scheduled in previous loops

2. Pending I/O Callbacks Phase:
   - Postponed I/O callbacks from previous loop
   - TCP error callbacks
   - UDP error callbacks
   - Completed read/write operations
   - Some system operations (like TCP ECONNREFUSED)

3. Idle, Prepare Phase:
   - Internal Node.js operations
   - Not typically used by developers

4. Poll Phase:
   - New I/O operations
   - fs.readFile() callbacks
   - fs.writeFile() callbacks
   - Network operations (http requests)
   - Database operations callbacks
   - Retrieves new I/O events
   - Can block here if certain conditions met

5. Check Phase:
   - setImmediate() callbacks
   - Execute immediately after poll phase
   - Often used to break up long operations

6. Close Phase:
   - Socket.on('close', ...)
   - Server shutdown callbacks
   - Connection termination handlers
   - Resource cleanup operations

Important Notes:
- Each phase maintains its own FIFO queue of callbacks (**NOTE:** queue which specifically handles macrotasks. These phase-specific queues are macrotask queues, not microtask queues.)
- Process.nextTick() and Promises can interrupt between any phase (**NOTE:** this is microtask queue that is being executed in between phases)
- I/O operations are mainly handled in Poll phase but their callbacks might be executed in Pending phase
- Timers are checked and executed in order of their timeout duration
- setImmediate() is designed to execute after I/O events

## Express.js and Middleware

### What is Middleware?
- Functions that have access to request, response objects, and next middleware
- Can:
  - Execute code
  - Modify req/res objects
  - End request-response cycle
  - Call next middleware
- Types:
  - Application-level
  - Router-level
  - Error-handling
  - Built-in
  - Third-party

### Design Patterns in Node.js

Types of Patterns:
1. Creational Patterns:
   - Factory: Creating objects without specifying exact class
   - Singleton: Single instance for database connections
   - Builder: Complex object construction
   - Prototype: Cloning existing objects

2. Structural Patterns:
   - Adapter: Interface compatibility
   - Facade: Simplified interface
   - Proxy: Control access to objects
   - Decorator: Dynamic functionality addition

3. Behavioral Patterns:
   - Observer: Event handling in Node.js
   - Strategy: Switchable algorithms
   - Chain of Responsibility: Middleware in Express
   - Command: Encapsulate operations

Common Node.js Examples:
1. Middleware Pattern (Chain of Responsibility)
   - Used in Express middleware
   - Sequential processing of requests

2. Observer Pattern
   - Event emitters in Node.js
   - Pub/sub implementations

3. Factory Pattern
   - Creating objects without specifying exact class
   - Common in ORM implementations

4. Singleton Pattern
   - Database connections
   - Configuration objects

## Databases

### CAP Theorem
- States that a distributed system can only provide two of three guarantees:
  - Consistency: All nodes see the same data at the same time
  - Availability: Every request receives a response
  - Partition tolerance: System continues to operate despite network failures
- Real-world applications:
  - MongoDB: CP (Consistency/Partition Tolerance) in default configuration
  - Cassandra: AP (Availability/Partition Tolerance)
  - PostgreSQL: CA (Consistency/Availability) in single node, CP in distributed setup

### Database Indexing
Definition:
- Data structure that improves the speed of data retrieval operations
- Trade-off between read and write performance
- Consumes additional storage space

Types of Indexes:
1. Single Column Index
   - Basic index on one column
   - Good for unique constraints
   - Example: Primary key index

2. Composite Index
   - Multiple columns in specific order
   - Order matters for query optimization
   - Best for frequently combined columns

3. Partial Index
   - Index subset of rows
   - Reduces index size
   - Good for filtered queries

Impact on Operations:
- Improves:
  - SELECT query performance
  - JOIN operations
  - ORDER BY operations
  - GROUP BY operations
- Slows down:
  - INSERT operations
  - UPDATE operations
  - DELETE operations

Best Practices:
- Index frequently queried columns
- Avoid over-indexing (impacts write performance)
- Consider column cardinality
- Regular index maintenance
- Monitor index usage

### MongoDB vs PostgreSQL
MongoDB:
- Schema-less
- Better for rapidly changing data
- Horizontal scaling
- Document-based (good for nested/complex data structures like comments, user profiles, product catalogs)
- Better for data with varying structure

PostgreSQL:
- ACID compliance (**NOTE:** Atomicity: all or nothing transactions, Consistency: data integrity rules, Isolation: concurrent transaction handling, Durability: committed data is saved)
- Complex queries/joins
- Structured data/flat data
- Better for relational data
- Strong consistency

### Redis
Uses:
1. Caching
2. Session management
3. Rate limiting
4. Real-time analytics
5. Pub/sub messaging

Power failure:
- Data is lost upon power failure as Redis is in-memory by default
- No special configuration needed - this is expected behavior
- For persistence, would need to configure RDB/AOF (but this is a different use case)

## AWS & Kubernetes

### AWS Services with K8s
- EKS (Elastic Kubernetes Service)
- ECR for container registry
- ELB for load balancing
- IAM for authentication
- CloudWatch for monitoring
- VPC for networking

### CI/CD/CD
1. Continuous Integration:
   - Regular code merging
   - Automated testing
   - Build verification

2. Continuous Delivery:
   - Automated deployment to staging
   - Manual production deployment
   - Environment consistency (**NOTE:** same code, dependencies, and configurations across dev/staging/prod)

3. Continuous Deployment:
   - Fully automated pipeline (**NOTE:** code commit triggers automatic testing, building, and deployment to production)
   - Required strong testing because:
     - No manual verification before production
     - Bugs immediately affect users
     - Rollback needs to be automated
     - Test coverage must be comprehensive

## Architecture

### Microservices vs Monolith

Monolith Pros:
- Simple deployment (single unit)
- Easy debugging (all code in one place)
- Less operational complexity
- Better performance (no network calls)
- Easier testing

Monolith Cons:
- Hard to maintain as codebase grows
- Scaling requires entire app deployment
- Single point of failure
- Technology stack is fixed
- Slower development in large teams

Microservices Pros:
- Independent deployment
- Technology flexibility
- Easier scaling
- Better fault isolation
- Parallel development
- Smaller, focused teams

Microservices Cons:
- Complex deployment orchestration
- Network latency
- Data consistency challenges
- More difficult testing
- Complex service discovery
- More operational overhead (monitoring/tracing)

### Monitoring Tools
1. Prometheus
   - Metrics collection
   - Time-series data
   - Alert management

2. Grafana
   - Visualization
   - Dashboards
   - Multiple data sources

4. Datadog
   - APM (Application Performance Monitoring)
   - Infrastructure monitoring
   - Log management
   - Real-time analytics
   - Custom metrics

5. New Relic
   - Full-stack observability
   - Real-time monitoring
   - Error tracking

## Docker & Containers

### Containerization
- Process of packaging application and its dependencies together
- Ensures consistent environment across different stages
- Provides isolation from other applications
- Makes applications portable across different platforms

### What is Docker?
- Platform for developing, shipping, and running applications
- Uses container technology
- Key components:
  - Dockerfile (build instructions)
  - Docker image (template for container)
  - Docker container (running instance)
  - Docker registry (image storage)
  - Docker compose (multi-container management)

### Docker vs VM
Docker:
- Uses containerization technology
- Shares host OS kernel
- Lightweight (MBs vs GBs)
- Faster startup (seconds)
- Less resource intensive
- Better for microservices
- Less secure (shared kernel)

VM:
- Complete OS isolation
- Better security (full isolation)
- More resource intensive
- Slower startup (minutes)
- Better for full OS needs
- Better for different OS needs

### Container Orchestration
Kubernetes features:
- Auto-scaling (adjusts number of pods based on load)
- Self-healing (automatically replaces failed containers)
- Load balancing (distributes traffic)
- Rolling updates (zero-downtime deployments)
- Config management (external configuration)
- Secret management (secure sensitive data)

## Network Protocols

### TLS Working
1. Handshake:
   - Client hello (cipher suites)
   - Server hello (chosen cipher)
   - Certificate exchange
   - Key exchange
   - Finished

2. Session:
   - Symmetric encryption
   - Data integrity
   - Perfect forward secrecy

### TCP vs UDP

TCP (Transmission Control Protocol):
- Connection-oriented protocol
- Three-way handshake (SYN, SYN-ACK, ACK)
- Guaranteed delivery with acknowledgments
- In-order packet delivery
- Flow control (prevents overwhelming receiver)
- Congestion control
- Error checking and retransmission
- Larger header size (20 bytes)

TCP Use cases:
- Web browsing (HTTP/HTTPS)
- Email (SMTP)
- File transfer (FTP)
- Remote administration (SSH)
- Any application needing reliable data delivery
- HLS (video streaming)

UDP (User Datagram Protocol):
- Connectionless protocol
- No handshake
- No guarantee of delivery
- No packet order guarantee
- No flow control
- No congestion control
- Simple error checking (checksum only)
- Smaller header size (8 bytes)
UDP Use cases:
- Video streaming
- Online gaming
- VoIP
- DNS queries
- Any application prioritizing speed over reliability
`
## Scaling

### Strategies
1. Vertical Scaling (Scale Up):
   - Adding more power to existing machine (CPU, RAM, SSD)
   - Easier to implement, but has hardware limits
   - More expensive with diminishing returns
   - No application architecture changes needed
   - Single point of failure remains

2. Horizontal Scaling (Scale Out):
   - Adding more machines to handle load
   - More complex to implement but virtually unlimited
   - Cost-effective (can use commodity hardware)
   - Requires load balancing and distributed architecture
   - Better fault tolerance through redundancy

3. Database Scaling:
   - Read Replicas: Copies of main database for read operations only, master handles writes
   - Sharding: Splitting data across multiple databases based on a key (e.g., user_id, region)
   - Partitioning: Breaking single large table into smaller ones based on logical divisions (e.g., by date, category)

## Message Queues

### RabbitMQ vs Kafka

RabbitMQ:
- Traditional message broker for point-to-point communication
- Smart broker, dumb consumer model
- Messages are deleted after consumption
- Good for: task queues, request/reply, pub/sub patterns
- Complex routing capabilities (exchanges, bindings)
- Guaranteed message delivery
- Lower latency, lower throughput

Kafka:
- Distributed streaming platform
- Dumb broker, smart consumer model
- Messages persist for configured time
- Good for: log aggregation, event sourcing, stream processing
- Simple routing (topics only)
- At-least-once delivery
- Higher latency, higher throughput

## Big O Notation

Big O Notation is a mathematical notation that describes the performance or complexity of an algorithm:
- Describes upper bound of growth rate of an algorithm
- Shows how runtime/space requirements grow as input size grows
- Helps compare algorithm efficiency at scale
- Always considers worst-case scenario
- Ignores constants and smaller terms (O(2n) becomes O(n))

### Common Complexities
- O(1): Constant time - operation always takes same time regardless of input size (array access, hash map lookup)
- O(log n): Logarithmic - input size is repeatedly divided (binary search, balanced tree operations)
- O(n): Linear - time grows linearly with input (simple loop, array search)
- O(n log n): Log Linear - common in efficient sorting (quicksort, mergesort)
- O(n²): Quadratic - nested iterations (bubble sort, insertion sort)
- O(2^n): Exponential - doubles with each addition to input (recursive fibonacci, subset generation)

Nested loop = O(n * m) where n, m are loop sizes - means outer loop runs n times, inner loop runs m times for each outer iteration

Common Example:
```javascript
// O(n) - linear time
for(let i = 0; i < n; i++) {
    console.log(i);
}

// O(n²) - quadratic time
for(let i = 0; i < n; i++) {
    for(let j = 0; j < n; j++) {
        console.log(i, j);
    }
}
```