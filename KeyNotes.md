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
