# Chapter 21 — Microservices with Node.js (Detailed Version)

> Microservices are a core system-design topic for senior roles. This chapter covers monolith vs microservices, communication patterns, API gateways, discovery, resilience, data consistency, security, deploy & monitoring, and advanced interview Qs — all in the context of Node.js.

---

## 📘 1. Monolith vs Microservices

### Monolithic Architecture
- All features (auth, payments, products, orders, UI) in one codebase + one database.  
- Deploy = ship the whole app.  
- Scaling = scale the entire app (even if only one module is bottlenecked).

**Pros**
- Easy to start.
- Simpler debugging (everything in one place).
- One DB → strong consistency.

**Cons**
- Hard to scale specific modules (must scale whole app).
- Deployments are risky (small bug may break entire app).
- Large team conflicts in one repo.
- Slow builds/tests.

### Microservices Architecture
- App split into independent services (Auth, Product, Payment, Order, ...).  
- Each service has own DB + logic. Communicate via APIs or message queues.

**Pros**
- Independent scaling.
- Teams own services.
- Fault isolation (one service failure doesn't crash others).
- Faster CI/CD per service.

**Cons**
- More complex (network latency, retries).
- Data consistency challenges (distributed DBs).
- Requires monitoring, discovery, CI/CD pipelines.

**Interview line:**  
> “Monolith is simple but rigid; microservices are complex but scalable.”

---

## 📘 2. Communication Between Services

### Synchronous (direct)
- REST (HTTP + JSON), gRPC (binary + strongly typed).  
- Use when you need immediate response (e.g., checkout -> payment).

### Asynchronous (indirect)
- Message brokers: Kafka, RabbitMQ, NATS, Redis Pub/Sub.  
- Producers publish events; consumers subscribe. Good for decoupling and scalability (e.g., order_created → inventory/notification).

**Interview line:**  
> “Use synchronous calls for immediate response; use async pub/sub to decouple and scale.”

---

## 📘 3. API Gateway
- Single entry point for clients (mobile, web).  
- Routes requests to correct microservice.  
- Handles auth (JWT/OAuth2), rate limiting, caching, SSL termination, response aggregation.

**Tools:** NGINX, Kong, AWS API Gateway, custom Express gateway.

**Without gateway:** clients must know each service URL.  
**With gateway:** client talks to one URL; gateway forwards.

---

## 📘 4. Service Discovery
- Services move (dynamic IPs in containers/K8s). Static config isn't scalable.
- Dynamic discovery: Eureka, Consul, etcd, Zookeeper.  
- Kubernetes provides built-in DNS + service discovery and Service Mesh (Istio, Linkerd).

**Interview line:**  
> “Service discovery ensures services find each other dynamically in a distributed system.”

---

## 📘 5. Resilience Patterns

### Circuit Breaker
- Prevents repeated failing calls. If downstream fails, circuit opens temporarily. Example lib: `opossum`.

### Retry + Backoff
- Retry failed calls with increasing delays (exponential backoff).

### Bulkhead
- Isolate failures (separate pools/limits so one overloaded component doesn’t take others down).

### Timeouts
- Always set HTTP timeouts to avoid request pileups.

**Interview line:**  
> “Circuit breakers and retries prevent cascading failures across microservices.”

---

## 📘 6. Data Consistency

### Monolith
- Single DB → ACID transactions easier.

### Microservices
- Each service owns its DB → distributed consistency required.

**Patterns**
- Eventual Consistency — services sync over time via events.
- **Saga Pattern** — split long transaction into local steps; use compensating transactions if something fails.
  - Example order saga: create order → charge payment → reserve inventory; on failure: run compensations.
- **Outbox Pattern** — write event to outbox table within DB transaction; publish outbox reliably to broker later.

**Interview line:**  
> “Microservices trade immediate strong consistency for eventual consistency using Saga and Outbox patterns.”

---

## 📘 7. Security in Microservices
- Centralize auth at API Gateway (JWT/OAuth2).  
- Service-to-service auth (mTLS, API keys, service mesh).  
- Encrypt in transit (TLS) and at rest.  
- Principle of least privilege between services.

---

## 📘 8. Deployment & Monitoring
- Containerize with Docker; orchestrate with Kubernetes.  
- CI/CD per service for independent deploys.  
- Observability: logs (ELK), metrics (Prometheus+Grafana), tracing (Jaeger/OpenTelemetry).

**Interview line:**  
> “In microservices, observability (logs, metrics, traces) is as critical as code.”

---

## 🎯 Interview Qs (Chapter 21) — initial set (preserved)

> Now that you have full context, here are the Qs (you answered them; your answers are preserved below):

**Q1.** What are 3 main differences between monolith and microservices?  
**Q2.** How would you make two Node.js microservices talk synchronously vs asynchronously?  
**Q3.** Why is an API Gateway useful in microservices?  
**Q4.** Explain Circuit Breaker pattern with an example.  
**Q5.** How do you maintain consistency across services with their own databases?

---

### Your original answers (preserved exactly as you wrote them)

> **Q1 (preserved):**  
> So three main differences between monolith and microservices are, in monolith, in initial part coding is very easy for smaller teams, 1 DB that means data is consistent, debugging can be easy, deployment can be easy. In microservices, every minor service is a separate service like for payment we use different service, for orders we use different service, and for users for different service, admin different service. In microservices, deployment is very complex, but over the time of when the code grows or large team grows, in large teams debugging can be easy in microservices. DB, there is a distributed DBs, in microservices scalability, we can easily scale the services. In monolith, scaling becomes an issue if bottleneck is only one service and then we have to scale the entire app. If one small issue can entirely grow, one small issue can entirely crash the whole app.

> **Q2 (preserved):**  
> In microservices, communication between the services happens using two ways. First is synchronous, second is in-synchronous. In-synchronous, we use REST APIs method like HTTPs. For example, when we try to place an order, we need to call the payment service immediately. For that reason, we use the synchronous services, HTTP. Now, asynchronous communication we use using message brokers like Kafka, Revit, MQ, or Redis PubShops. This makes the service like the publisher and the subscribers decoupled. We use this method when we don't want expecting the data immediate. Like when an order is created, a notification service, even to the notification service for triggering a notification.

> **Q3 (preserved):**  
> API Gateway in a microservice is important because it creates a single entry for all the clients. Like it can be mobile, web, or any third party. It is a single entry point. The request coming from various clients, the API Gateway decides which request goes to which microservices. Like from the front end, there will be a single server URL and for all the things. Then in API Gateway, we have to decide like this, either this go in the user service, admin service, or any product service. Here we can handle the authentication part, rate limiting part, caching part, and SSL termination part. Without API Gateway, we have to know each URL. For example, in our structure, if we have admin service, user service, and product service, then the client front end has to know which service to call when.

> **Q4 (preserved):**  
> Circuit Breaker Pattern Circuit Breaker Pattern in microservices is useful like if after a failed service, we have to protect our service from the repeated failed calls. If a service is failing, then what we have to do is to like the open the request and turn it closed in order to protect our services. And with each retry, we are adding some delays and these delays are exponential. Like for the first retry, like one second, two seconds, three seconds, like the time grows bulk and we have to make sure like each failure is isolated. For example, like the one service overload cannot crash, should not be able to crash the other services and we should use timeouts between the repeated HTTP calls.

> **Q5 (preserved):**  
> Data consistency in microservices is one of the major concerns. In monoliths, we have a 1DB, so there is an easy transaction. In microservices, we have a separate DB and distributed consistency. So, we use eventual consistency or consistency, since the services communicate via the events, data may be temporarily inconsistent, like for 1 second or 2 seconds, but eventually it syncs after a certain period of time. We can also use the Saga pattern. In the Saga pattern, long transactions are split into smaller local transactions. If one transaction fails, then we have to run a compensating transaction to undo all the things. For example, when we want to create an order, in the first, this is a Saga pattern, it is a long transaction which is split into multiple, first the order has to be created, then the payment should be initiated with that order ID, then the payment happened or not, then inventory check and notification check and updating the user and everything. If any of the steps fails, then we have to cancel the entire order. This is known as the Saga pattern, where we have to run a compensating transaction to undo all the things. One of the examples is also Outbox pattern. In the Outbox pattern, like service writes event in a table, in Outbox table inside the DB and later event is later published via the broker and ensures DB write publish.

---

## ✅ Polished (interview-ready) refinements & model answers

> Use these to answer concisely in interviews — they are tightened, exact, and easy to drop in a panel.

**Q1 (polished)** — *3 main differences between monolith and microservices*  
1. **Deployment & CI/CD:** Monolith = single deploy; Microservices = many independent deploys.  
2. **Scaling model:** Monolith scales vertically/horizontally as a unit; Microservices scale per service (more efficient).  
3. **Operational complexity:** Monolith = simpler local debugging and single DB transactions; Microservices = distributed systems issues (network, discovery, consistency) but better team ownership.

**Q2 (polished)** — *Synchronous vs Asynchronous communication*  
- **Synchronous:** HTTP/REST or gRPC for immediate responses. Example: Order → Payment (need immediate success).  
- **Asynchronous:** Message broker (Kafka/RabbitMQ/Redis) for decoupling and durability. Example: OrderCreated → Inventory/Notification consumers process later.

**Q3 (polished)** — *Why API Gateway?*  
> “API Gateway provides a single client-facing endpoint, centralizes cross-cutting concerns (auth, rate-limiting, SSL, caching), and can aggregate calls to multiple services.”

**Q4 (polished)** — *Circuit Breaker with example*  
> “Circuit Breaker detects repeated failures to a downstream service and opens the circuit to stop calls temporarily. Example: If Payment service times out repeatedly, the breaker opens, causing clients to get a fast failure fallback (e.g., queue the action) until the downstream recovers.”

**Q5 (polished)** — *Maintaining consistency across services with own DBs*  
- Techniques: **Eventual consistency**, **Saga** (orchestrated or choreographed), **Outbox** pattern, and compensating transactions.  
- Use idempotent events, retries, and dead-letter queues to avoid message loss.

---

## 🟢 Advanced Microservices Interview Questions (Q6–Q10) — you were asked to answer; your answers are preserved below, followed by polished versions.

### Q6 (you answered) — *Auth across microservices* (your preserved answer)
> In microservices, we handle authentication and authorization across multiple services. So, say in a microservices structure, in API gateway, we can authenticate the user using JWT and while accessing any service, we can send the authorization token in the headers and for each individual service, we can check if the user is authorized or not. By in the middleware, by decrypting the header authorization token and check it against the if it exists or not in by decrypting using JWT.

**Q6 (polished)** — *Auth & AuthZ across microservices*  
> “Centralize authentication at the API Gateway (validate JWT/OAuth token) and propagate identity via signed JWT or a short-lived service token. Each service runs lightweight authorization middleware to validate scopes/roles. For service-to-service calls, use mTLS or service tokens; use an identity provider (Auth0/Cognito/Keycloak) in complex setups.”

---

### Q7 (you answered) — *Orchestration vs Choreography* (your preserved answer)
> Kubernetes controlling services is kind of related to the serviceability of the services... In this case, service or the client have to manually recognize them and call them like where they are placed. So, in an ideal case scenario ... I will prefer Kubernetes

**Q7 (polished)** — *Orchestration vs Choreography*  
> “**Orchestration** = a central coordinator (Kubernetes for infra; an orchestrator for sagas) controls flows. **Choreography** = services react to events (publish/subscribe) without a central brain. For an event-driven Node.js system, choreography is natural; for complex multi-step workflows requiring strict control, orchestration is better. Kubernetes handles infra orchestration (scheduling, discovery, scaling).”

---

### Q8 (you answered) — *Payment succeeded but Notification failed* (your preserved answer)
> Since a payment service is successful, but a notification failed, so in this, this is one of the common examples of the data inconsistency, data inconsistency, and we can handle it by the Saga pattern, Saga pattern like, once the service is, once the payment is successful, a notification service becomes inactive. So when a transaction doesn't get fulfilled, we can also assign it like the failing mechanism, like a side effect of the failing. So what we can do here is, like if the sending of the notification fails, then we can assign a salary task or a sync task in like using any event and schedule it for after sub time or with a retry time. So circuit brokers, like kind of a circuit where retrying after a certain period of time.

**Q8 (polished)** — *Design to handle Payment success & Notification failure*  
> “Design for eventual consistency: Payment publishes a `payment_succeeded` event to a durable broker. Notification service consumes and retries with exponential backoff if sending fails; after X retries moves the message to a DLQ for manual or automated retry. Alternatively, keep a `notifications` table and let a worker reprocess failed notifications. This ensures payment stays committed while notification is retried or compensated.”

---

### Q9 (you answered) — *Rate limiting design* (your preserved answer)
> A rate limiting mechanism in Node.js Microservices, so we can, like whenever any request comes from any user in its middleware, we can store the number of requests against its IP address of a user and whenever a new request comes, we just check if the user has any record in the Redis related to the rate limiting and if a certain IP address has more than certain period of a request in a particular point of a time, like 100 requests in last 30 seconds, then we immediately stop the request and we set the key with a TTL.

**Q9 (polished)** — *Rate limiting in Node microservices*  
> “Implement middleware using Redis for distributed counters (INCR + EXPIRE): per-IP or per-user windowed counters. Use token bucket or sliding window algorithms for fairness. Popular libs: `express-rate-limit` with a Redis store, or custom logic using `INCR` and TTL. For global quotas, use API Gateway rate-limiting features.”

---

### Q10 (you answered) — *Role of Docker & Kubernetes* (your preserved answer)
> We use docker in microservices in to dockerize or to containerize our OS or the system or the virtual machine so that our dependencies and requirements related to that specific projects remain inside a container only. ... Kubernetes we use in microservices like when we use when we deploy the dockerized container they all have the dynamic IPs and to hand and to maintain the serviceability of these containers like these IPs are changed changing frequently after a shutdown or a restart so each service should know where the other service should exist ... and we use the kubernetes for the serviceability

**Q10 (polished)** — *Docker & Kubernetes role*  
> “Docker packages each microservice and its dependencies into a reproducible container. Kubernetes orchestrates containers: scheduling, scaling, self-healing, service discovery, and rolling updates. Together they provide consistent deployments, automatic scaling, and reliable service management — far superior to manually running services on VMs.”

---

## ⚡ Rapid-fire — quick mental checks

- **R1** — *Synchronous vs async example*  
  - `OrderService` calls `PaymentService` via HTTP (sync).  
  - `OrderService` publishes `order_created` event (async); inventory consumes it.

- **R2** — *Outbox pattern*  
  - Transaction: insert order row + insert event into outbox table in same DB tx → later, a publisher reads outbox and publishes reliably.

- **R3** — *Dead-letter queue*  
  - When message processing fails after retries, send to DLQ for manual inspection.

- **R4** — *Circuit Breaker libs*  
  - Node: `opossum`, `cockatiel`.

- **R5** — *Idempotency*  
  - For retries, design idempotent handlers (use idempotency key to avoid double charges).

---

## 🏢 Top 10 MNC / FAANG-style Questions (Chapter 21) — Quick Answers

1. **3 main differences Monolith vs Microservices** — deployment model, scaling granularity, operational complexity.  
2. **Synchronous vs Asynchronous communication** — REST/gRPC vs broker (Kafka/RabbitMQ). Use sync for immediate needs; async for decoupling.  
3. **API Gateway use** — single entry, centralize auth, rate-limit, caching, response aggregation.  
4. **Circuit Breaker** — stops calling failing downstreams to prevent cascading failure; example: open after X failures, then half-open probe.  
5. **Consistency across DBs** — Saga, Outbox, eventual consistency, compensating transactions.  
6. **Auth across services** — Gateway validates JWT, services verify, use mTLS/service tokens for S2S.  
7. **Orchestration vs Choreography** — central coordinator vs event-driven workers; K8s orchestrates infra.  
8. **Payment success / notification fail** — publish events, use retries + DLQ, keep core transaction committed.  
9. **Rate limiting** — Redis counters, token bucket, enforce at gateway or service middleware.  
10. **Why Docker + K8s** — Docker packages; K8s orchestrates at scale (autoscale, service discovery, rolling updates).

---

## 📎 Final Answer Key & Interview Scripts

- Use the **polished** answers above in interviews — they’re concise and sound like a senior engineer.  
- **Short sound-bytes to memorize:**
  - “Microservices trade simplicity for modularity — more moving parts but better team autonomy and scaling.”  
  - “Use API Gateway for cross-cutting concerns; use message brokers for decoupling.”  
  - “Sagas & Outbox patterns help with eventual consistency; use DLQs + retries to avoid silent data loss.”  
  - “Docker gives reproducibility; Kubernetes gives orchestration, scaling, and discovery.”

---
