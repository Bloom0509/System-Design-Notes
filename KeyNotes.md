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
