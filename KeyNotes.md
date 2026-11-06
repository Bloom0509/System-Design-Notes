## what is Client Server Architecture?
- As its core, Client Server Architecture is a model where :
  - client requests data or services
  - server process those requests and responds with a result .
- This is the foundation of how web works. This communication happens over the HTTP/HTTPS protocol which follows request/response cycle.  
- Client never talks directly to the database, direct access is dangerous and inefficient.
  ### 2 types of Architecture- 2Tier and 3Tier
  - 2 Tier -> The simplest form where clients and servers communicate directly.
  - 3 Tier -> A middleware layer, often called the application layer, is added between the client and server.
  - N Tier -> Involves multiple intermediary layers, such as security and business logic, to manage complex data processing and ensure security.
## Monolithic vs Microservice Architecture
- A monolith means the entire application is built as one single unit- everything(login,payments,dashboard,admin panel) is written together and deployed together.
- Microservices break the system into small separate parts where each part (or service) is responsible for just one job like handling users ,payments or notifications
- You have many small services each doing specific tasks
- Each service can be developed ,tested and deployed on its own
- These services talk to each other using APIs( like REST,gRPC)
- Each service can even use its own database
## Synchronous vs Asynchronous Communication
 In system design, when two services or components communicate ,the way they exchange messages can be:
  - synchronous : The sender sends the request and wait for response before doing anything else
  - Asynchronous  The sender sends the request and immediately  moves on without waiting for any response.
  - This affects the speed, scalability , complexity and reliability of your system.
 Synchronous communication happens in real time, client sends a request (usually HTTP) and HTTPs on real world.
 The server processes it and sends a response . The client is blocked until the server replies
 If the server crashes or its too slow, the client also waits or fails.
  - Asynchronous communication happens indirectly or in the background , sender puts a message into queue or sends a event, when ack or completed. The worker or listener picks it up later and processes it.
 ## Load Balancer
 - A load balancer is a system component that distributes incoming network traffic across multiple servers to ensure no single server gets overwhelmed.
 - The load balancer sits between users and your application servers.
 - It decides which server gets which request.
 - If one server is down, the balancer redirects traffic to the healthy ones.
 ## Without a load balancer:
 - A sudden spike in users can crash a single server.
 - One failed server can make your whole app unavailable.
 - It's hard to scale your system or update servers without downtime
 ## With a load balancer:
 - You get higher availability (no single point of failure).
 - Your system can scale horizontally (add more servers easily).
 - Traffic is optimized and handled efficiently.
 ## Types of load Balancer
   ## Static Load Balancer
  - Traffic is assigned based on a set rotation, pre-assigned weights, or a hash formula, regardless of how busy or slow a server might be at that exact moment.
  - Predetermined path (server status)
   ## Dynamic Load Balancer
  - The load balancer actively communicates with or monitors servers to see how many active connections they have, how fast they are responding, or how much CPU/memory they are using.
  - routes traffic based on server health/performance
  - Difficult to configure
   ## Based on OSI
  ## Layer 4 (Network/Transport Layer) Load Balancers
   - Routes traffic using transport-level information like source/destination IP addresses, port numbers, and TCP or UDP protocols without inspecting content. Ideal for high-speed routing like multiplayer gaming.
   - Content of packet is not  important
   - fast
  ## Layer 7 (Application Layer) Load Balancers
   - Inspects application-level data—such as HTTP headers, URL paths, cookies, and request methods—to make content-aware routing decisions (e.g., routing /api traffic to one server pool and /images to another).
   - Content of packet is important for making routing decision.
   - generally slow
  ## Round Robin Algorithm
   - The algorithm operates in a fixed, cyclical loop, passing incoming requests to the next available server in sequential order.
  ## Least Connection Algorithm
   - Dynamic load balancing algorithm that routes incoming requests to the server with the fewest active, open connections at that moment.
  ## sticky session
   - A routing technique where a load balancer consistently directs all subsequent requests from a specific user to the same backend server for the duration of their session.
   - shopping cart,dashboard
   - Great user experience
  ## Caching
   - Caching is the process of storing frequently accessed data in a faster storage layer so future requests can be served faster, instead of recalculating or  fetching the data again from a slower source (like a database or API).
   ## Types of caching
  - Client-side Caching: Browser caching stores webpage resources on a local computer when a user visits a webpage. On the next visit, the browser loads the page from local resources, reducing server load and improving page load speed.
  - CDN Caching: Content Delivery Networks (like Cloudflare, Akamai) cache static files near users to reduce latency.
  - Application-level Caching: Backend servers use in-memory caches (e.g., Redis, Memcached) to store frequently used data like: popular products,leaderboard
  - Database caching: Cache results of frequent queries so they aren't recomputed.
   ## Cache Hit
   - The requested data is found inside the cache
   ## Cache Miss
   - The requested data is not found in the cache
   ## Cache Algorithm
  ## LRU (Least Recently Used
  - Evicts the item that has not been accessed for the longest time.
  - Action: Evicts the item that hasn't been touched for the longest time.
  - Before: [A, B, C] (A was used longest ago)
  - Evicted: A
  - After: [B, C, D]

  ## FIFO (First-In, First-Out)
  - Evicts the oldest item that was loaded into the cache first, regardless of how often or recently it was used.
  - Action: Evicts the item that entered the cache first, regardless of how often it was used.
  - Before: [A, B, C] (A entered the cache first)
  - Evicted: A
  - After: [B, C, D]
  - Note: Even if A was accessed a million times right before D arrived, FIFO still kicks it out because it is the oldest arrival.
    
  ## LFU (Least Frequently Used)
  - Tracks how many times an item has been accessed and evicts the one with the lowest total count.
  - Action: Evicts the item with the lowest access count.
  - Before: [A (5 hits), B (12 hits), C (2 hits)]
  - Evicted: C (It only has 2 hits)
  - After: [A (5 hits), B (12 hits), D (1 hit)]

  ## CDN (Content Delivery Network)  
  - A content Delivery Network (CDN) is a network of servers located around the world that cache and deliver static content(like images,CSS,javascript,videos) from the server closest to the user
  - imagine ordering a product online 
  - Without CDN-> All orders are shipped from one warehouse in one city -slow and overload
  - with CDN-> There are warehouses across the country , so you get your item from the nearest one, fast and efficiently.
  ## CDN working -> CDN relies on 3 types of servers
  - Origin servers: origin servers contain the original versions of content, and they function as the source of truth. Whenever content needs to be updated, changes are made on the origin server.
  - Edge servers: edge servers are located in multiple geographical locations around the world, also called points of presence (pops). The edge servers within these pops cache content that is copied from origin servers, and they are responsible for delivering that content to nearby users.
  - DNS servers: when a client sends a request to an origin server, DNS servers respond with the name of a paired edge server from which the content can be served faster
  ## what content is cached?
   - mostly static assets
   - HTML,CSS, javascript
   - Images, videos, PDF
   - Fonts and icons

 ## SQL vs NoSQL
 - A database is critical in any Application, So when building a system need to know what kind of Database we should use.
 - The choice between NoSQL vs. SQL databases impacts the efficiency of data retrieval, storage, and processing.
   ## SQL
    - SQL (Structured Query Language) databases are relational
    - Data is stored in tables with predefined schemas.
    - A relationship between the two tables (or more) can be created using a foreign key.
    - They have a fixed schema, and all data ingested into the database must be precisely aligned to predefined formatting standards.
    - Use SQL queries to fetch or update data.
    - Ex: PostgreSQL, MySQL, Microsoft SQL Server, Oracle
    ## NoSQL
    - NoSQL (Not Only SQL) databases are non-relational.
    - Data can be stored in key-value, document, graph, or column formats — often schema-less
    - Designed for horizontal scaling
    - Supports different models
    - Key-Value (Redis, DynamoDB)
    - Document (MongoDB, CouchDB)
    - Wide-Column (Cassandra, HBase)
    - Graph (Neo4j, ArangoDB)
    ## when to use what?
    - ACID compilance -SQL
    - schema flexible- NoSQL
    - complex query- SQL
    - low read latency -NoSQL
    - data has complex relationship- SQL
    - high speed caching session data- NoSQL
    - Transaction related query- SQL (consistency)
    ## ACID
    - A-atomiticity-> success or failure, no partial transaction
    - C-consistency-> database should be in a valid state
    - I-isolation-> transaction don't interfere with each other, no dirty read/write
    - D-duration-> once committed changes are permanent even though it crash. transction are never loss
  ## Horizontal vs Vertical Scaling
   - Scaling is the process of increasing your system's capacity to handle more traffic, more users, or more data.
   - There are two main types of scaling strategies: Vertical Scaling and Horizontal Scaling
  ## Vertical Scaling (Scaling Up)
   - You make a single server more powerful by adding more: CPU, RAM, DISK ,Network Bandwidth
   - InShort: You keep the same machine, just upgrade its hardware. No architecture changes — just bigger box.
  ## Pros
   - Simple as no need to change architecture
   - Easy to manage
   - Fast read/write as everything is one machine
  ## Cons
   - There's a limit as we can't upgrade one server infinitely
   - Bigger configuration are costly
   - Single point failure
  ## Horizontal Scaling (Scaling Out)
   - You add more machines (nodes) to handle the increased load. Each machine handles a portion of the traffic or data.
   - Imagine you're running an app server : So, instead of upgrading your existing server, you add 3 more servers behind a load balancer.
   - Each request is now distributed across all servers.
   ## Pros
   - High Fault Tolerance and Redundancy: If one server crashes, other nodes keep running, and a load balancer can route traffic away from the failed machine.
   - Virtually Unlimited Scalability: You are not constrained by the physical hardware limits of a single computer (such as max RAM or CPU slots); you can keep adding more nodes as demand grows.
   - No Downtime for Upgrades: You can bring new servers online or patch existing ones incrementally without taking the entire application offline.
   - Cost-Effectiveness at Scale: You can use cheaper commodity hardware or auto-scaling cloud instances instead of paying exponentially more for high-end, enterprise-class single servers.
  ## Cons
   - Increased System Complexity: Managing a distributed network requires extra components like load balancers, service discovery, and monitoring.
   - Data Consistency Challenges: Keeping data synchronized across multiple database replicas or shards is difficult and can introduce latency or conflict issues.
   - Application Architecture Changes: Apps often need to be stateless or redesigned so that user sessions and data do not depend on a single local server memory.
   - Higher Initial Overhead: Setting up orchestration (such as Kubernetes), sharding, and replication can make the early stages more expensive and time-consuming to build.
  ## Database sharding   
   - Sharding is a database architecture pattern where large databases are split into smaller, faster, more manageable parts called shards.
   - Each shard is an independent database instance that holds a subset of the total data.
     ## why sharding?
      - Huge table Size and slow Queries
      - Can't scale vertically
      - High read/write load on one DB
      - Single point of failure
     ## How sharding works?
      - Chosing a shard key
      - Applying the sharding algorithms
     ## how shard key is chosen?
      - The key must have a large number of unique values (e.g., user_id, order_uuid, or email).
      - Low-cardinality fields (like status: [active, inactive] or country if 90% of users are in one country) fail to distribute data evenly.    
     ## Types of sharding algorithms
       ## Hash-Based Sharding
        - Apply a hash function to the sharding key
        - Modulo by number of shards -> to pick target shard
        - shard_id = hash(user_id) % 4 # Suppose 4 shards
        - Cons: Hard to add/remove shards (hash changes = full rebalancing)
       ## Range Based Sharding
        - Split data by key ranges
        - Example:
           - Users with ID 1–100K -> shard1
           - Users with ID 100K-200K -> shard2
        - Cons: some ranges grow faster
        ## Directory Based Sharing
        - In directory-based sharding you maintain a lookup table (directory) that maps each sharding key value to a specific shard.
        - Cons: The lookup directory becomes a single point of failure and a potential performance bottleneck, requiring aggressive caching , 1 extra lookup and directory must be sync with data migration.
