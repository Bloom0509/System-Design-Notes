 ## Redis architecture
  - redis is an in-memory key-value store designed for extreme speed and low latency.
       - It is designed to keep everything in RAM rather than on disk
       ## why do we need redis?
       - low latency
       - fast speed
       ## high level architecture
       - client sends request
       - app checks redis first
       - if data found -> return instantly(cache hit)
       - else fetch from db ->store in redis ->return (cache miss)
       - redis is mostly used as cache layer , not the primary database
     ## capasity estimation
     ## Throughput
    - ~100K - 500k ops/sec per instance
        - Depends on:
          - Command complexity
          - Network
          - Hardware
        ## latency
        - Less than 1ms (<1ms). This is why it is used in:
           - Real-time systems
           - Leaderboards
           - Gaming
        ## Memory
        - All data is stored in Ram hence, hence it depends on RAM size.
        - Ex: 64GB Ram, then max ~60GB database
        - If your Redis node is 64 GB and your dataset grows to 70 GB, Redis needs an eviction policy.
        - LRU evicts the least recently used key. LFU evicts the least frequently used.
        - For session stores, LRU makes sense. For leaderboards or counters, you probably want no eviction and simply over-provision.
      ## core components
       ## RESP (redis serialization protocol)
        - RESP is the wire format redis uses to communicate between client and the server
        - It's intentionally dead simple -plain ASCII text, human-readable, easy to parse in any language
        - Every command your application sends and every reply redis returns, travel as RESP bytes over a TCP socket
        - The protocol encodes data using a type prefix character on each line
           - "+" means a simple string (like +ok)
           - "-" means an error
           - ":" means an integer
           - "$" means a bulk string with a declared byte length
           - "*" means an array. every redis command is just an array of bulk strings
           - "\r\n" in the end of line marker b/w parts of message
         ## Event Loop
         - redis is single threaded for command execution. But how can a single redis process handle tens of thousands of client connections?
         - Imagine 100,00 clients connected : client1, client2, client3 .... client 100,00
         - Naive thinking would love to use threads , 1 thread/client
         - But that's disastrous considering redis usage
         - why? every thread requires:
            - thread stack
            - cpu scheduling
            - context switching
            - synchronization
         - suppose each thread consumes 1mb stack
         - then 1,00,000 , 1mb=100gb ram just for thread stack
         ## how redis manages even though being single threaded?
         - Instead of continuously checking thousands of socket itself, redis simply calls:
         - epoll_wait()
         - and goes to sleep, when data arrives on any client connection, the kernel wakes redis and returns the list of ready sockets
         - Epoll is a linux service/api . Redis tells the kernel that you are watching the sockets. Tell me only when something interesting happens
         - Example:
         - Client A-> no data
         - Client B-> no data
         - Client C-> GET user:123
         - epoll_wait() returns socket C is ready
         - Redis only processes that socket
         - The event loop is higher-level reactor pattern implementation that uses epoll to detect ready sockets and then processes commands
         ## In Memory Storage Engine
         - Most people thinks redis store data like: unordered_map<string, string>
         - It doesn't. Redis storage engine is optimised for two things:
         - fast lookup
         - low memory usage
         - redis doesn't use one internal representation per data type
         - It uses multiple encodings and switches between them automatically based on the size and content of your data
         - This is invisible to you as a client, but it's responsible for Redis using 5-10x less memory than a naive implementation.
         -      // Simplified Redis object structure
                    struct redisObject {
                        unsigned type,      // -> data type 
                        unsigned encoding,  // -> how redis is going to store raw, unsigned
                        unsigned lru,       // -> LFU, LRU
                        int refcount,      // -> memory management
                        void *ptr      // -> pointer to actual value
                    } robj;  
        - Every single value, whether a string, a list, or a sorted set, is wrapped in this struct   
