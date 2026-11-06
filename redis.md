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
          
