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
