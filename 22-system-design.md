# Chapter 22 — Real-World System Design with Node.js

> At senior roles, system design interviews test if you can design **scalable, fault-tolerant, production-grade systems**.  
> Node.js fits well for **high I/O workloads** (APIs, real-time, streaming) but needs careful design for **CPU-bound** and **consistency-heavy** domains.  

---

## 📘 Case Studies

### 1. E-commerce System (Amazon-like)

**Services:**
- User Service (accounts, auth, profile)
- Product Service (catalog, inventory)
- Cart Service
- Order Service (checkout, payments)
- Payment Service (Stripe/PayPal integration)
- Notification Service (emails, SMS)

**Infra:**
- API Gateway → routes to services.
- DBs:
  - User → PostgreSQL/MySQL  
  - Product/Inventory → MongoDB or Elasticsearch (fast search)  
  - Orders → PostgreSQL/MySQL
- Message Queue → async events (`order_created`, `payment_success`).  
- Caching → Redis for hot products, sessions, rate limiting.

**Challenges & solutions:**
- Scalability → scale services independently.  
- Consistency → Saga + Outbox for orders/payments.  
- Search → Elasticsearch or AtlasSearch.  
- Hot product caching → Redis.

---

### 2. Real-time Chat App (WhatsApp-like)

**Services:**
- Auth Service
- Chat Service (messages, typing indicators)
- Notification Service (push notifications)
- Media Service (file uploads)

**Communication:**
- WebSockets (Socket.IO).  
- Sticky sessions + Redis Pub/Sub adapter for horizontal scaling.  

**Infra:**
- DB: MongoDB (messages), Redis (presence, sessions).  
- Queue: Kafka/RabbitMQ → reliable delivery.  

**Challenges & solutions:**
- Offline delivery → queue messages until user reconnects.  
- Typing indicators → ephemeral events (don’t persist).  
- Media → store in S3/GCS, not DB.

---

### 3. Video Streaming Service (Netflix-like)

**Services:**
- User Service  
- Content Service (movies, metadata)  
- Streaming Service (video chunks, manifests)  
- Recommendation Service (ML model, separate microservice)  
- Payment Service  

**Infra:**
- Storage: S3 + CDN (CloudFront/Akamai/Fastly).  
- Streams: Node handles manifests, signed URLs, and metadata; video delivered by CDN.  
- Scaling: CDN edge caching → millions of concurrent viewers.

**Challenges & solutions:**
- Bandwidth → use adaptive bitrate streaming (HLS/DASH).  
- Scaling → CDN + multi-region edge nodes.  
- Recommendation → ML service, Node just consumes.  
- QoE → monitor startup time, buffering ratio, cache hit ratio.

---

### 4. Banking/Payments (PayPal-like)

**Services:**
- Auth (multi-factor login)  
- Account (balance, transactions)  
- Payment (transfers, payouts)  
- Fraud Detection  
- Notification  

**Infra:**
- DB: PostgreSQL (ACID).  
- Consistency: two-phase commit (rare in practice) or Saga for transfers.  
- Event-driven audit logs.  
- Security: JWT + mTLS.  

**Challenges & solutions:**
- Data integrity → strong ACID, not eventual consistency.  
- Latency → keep transfers <200ms.  
- Scale → DB sharding + replicas.  
- Fraud detection → rule engines + ML anomaly detection.

---

## 🎯 Common Interview Questions (with your answers + polished corrections)

### Q1. How would you handle product search & filtering efficiently?

**Your answer (preserved):**  
> Using MongoDB + Elasticsearch (AtlasSearch in cloud).  

**Polished:**  
- Use **Elasticsearch** for full-text + faceted search.  
- Maintain indexes (category, price, rating).  
- Sync product DB with search index via CDC or event-driven pipeline.  

---

### Q2. How do you ensure an order is only placed if inventory exists?

**Your answer (preserved):**  
> Use Saga → check inventory before and after payment, cancel if inconsistent.  

**Polished:**  
- Reserve inventory **atomically** before confirming order.  
- Saga workflow:
  - Step1: Create order (pending).  
  - Step2: Reserve inventory.  
  - Step3: Process payment.  
  - Step4: Confirm order.  
- If payment fails → Saga compensates by releasing reserved inventory.  
- Outbox pattern ensures reliability between DB + event publish.  

---

### Q3. How do you handle offline messages in chat?

**Your answer (preserved):**  
> Buffer in Socket.IO + queues until reconnect.  

**Polished:**  
- Store undelivered messages in durable DB/queue (MongoDB + Kafka).  
- Each message has ID + ack.  
- On reconnect, deliver pending messages, deduplicate via ID.  
- Typing indicators remain ephemeral (not persisted).  

---

### Q4. How do you scale WebSockets across servers?

**Your answer (preserved):**  
> Sticky sessions + Redis Pub/Sub adapter.  

**Polished:**  
- Sticky sessions → client always connected to same worker.  
- Redis Pub/Sub (or Kafka/NATS) → broadcast events across workers.  
- Map socketId ↔ user in Redis for global awareness.  
- Use socket.io-redis or socket.io-adapter.  

---

### Q5. Why use Streams in Node for video?

**Your answer (preserved):**  
> Large video files → serve via Node streams in chunks.  

**Polished:**  
- Streams serve video in chunks using HTTP Range requests.  
- Avoids loading entire file into memory.  
- Clients fetch only required byte ranges (seek/skip).  
- Combine with CDN for global scale.  

---

### Q6. How do you scale Netflix-like app to 1M viewers?

**Your answer (preserved):**  
> Use S3 + transcoding to chunks + CDN caching + adaptive bitrate.  

**Polished:**  
- Transcode into renditions (240p–4K) + segments (HLS/DASH).  
- Store in S3 → replicate globally.  
- CDN caches segments; multi-CDN strategy for reliability.  
- Node handles manifests + auth tokens.  
- Pre-warm caches for popular shows.  
- QoE monitoring (startup <2s, buffer ratio <1%).  

---

### Q7. How do you ensure banking transactions are atomic?

**Your answer (preserved):**  
> Use PostgreSQL ACID, Saga, 2PC, audit logs.  

**Polished:**  
- Prefer **single authoritative DB** for critical transfers (ACID).  
- For cross-service flows → Saga with compensations.  
- Audit logs → immutable event log for compliance.  
- Outbox pattern to ensure consistency between DB + events.  

---

### Q8. How do you prevent fraud in payments?

**Your answer (preserved):**  
> JWT + mTLS auth, ACID DB, multiple checks.  

**Polished:**  
- Strong auth (MFA, JWT, mTLS).  
- Fraud detection service → ML anomaly detection + rule engine.  
- Rate limiting + velocity checks.  
- Device fingerprinting + IP reputation.  
- Secure audit logging + monitoring unusual patterns.  

---

## ⚡ Rapid-fire — Outputs

- **R1.** What’s the difference between Saga and 2PC?  
  - Saga = eventual consistency, compensations.  
  - 2PC = strict ACID but heavy & slow.

- **R2.** Why Redis in e-commerce?  
  - Cache hot products, store sessions, manage rate limits.

- **R3.** Why Kafka in chat?  
  - Durable message delivery, replayability, scale.

- **R4.** What’s idempotency in payments?  
  - Ensure retries don’t double-charge (use idempotency keys).

- **R5.** How do you reduce CDN costs?  
  - Maximize cache hit ratio, pre-warm caches, segment TTL tuning.

---

## 🏢 Top 10 MNC / FAANG-style Questions

**Q1.** Monolith vs Microservices trade-offs?  
- Monolith = simpler dev, strong consistency.  
- Microservices = scalable, fault-isolated, but complex (latency, discovery).

**Q2.** When use async messaging in Node microservices?  
- For decoupled workflows (notifications, analytics, order events).  
- When immediate response not needed.

**Q3.** Role of API Gateway?  
- Central entry, auth, rate limiting, SSL, aggregation, routing.

**Q4.** How do you ensure consistency across services?  
- Saga, Outbox, idempotency, retries, DLQs.

**Q5.** Scaling WebSockets?  
- Sticky sessions + Redis adapter.  
- For massive scale → Kafka/NATS pub-sub.

**Q6.** Why not just store video in DB?  
- DB not optimized for GB-scale binary. Use object storage (S3/GCS) + CDN.

**Q7.** What metrics matter for video QoE?  
- Startup latency, rebuffering ratio, average bitrate, CDN hit ratio.

**Q8.** What’s a dead-letter queue (DLQ)?  
- Queue for failed messages after retries; allows safe reprocessing.

**Q9.** Why mTLS between services in payments?  
- Ensures service-to-service identity & encrypted channel. Critical for financial data.
🔄 JWT + mTLS Together in Microservices

Think of it like two layers of trust:

mTLS → Service Identity

Service A and Service B both have certificates.

When they connect, TLS handshake validates both certificates.

This ensures Service A really is Service A, and Service B really is Service B (no imposters).

JWT → User Identity / Authorization

Service A calls Service B on behalf of the user.

It forwards a signed JWT representing the user.

Service B validates the JWT claims (role, scope, expiry).

**Q10.** How do you deploy Node microservices at scale?  
- Docker containers + Kubernetes for orchestration, scaling, service discovery.  
- CI/CD pipelines for independent deploys.

---

## 🎬 Netflix-Style Deep Dive (Whiteboard Summary)

**Requirements:**  
- Support millions of concurrent viewers.  
- VOD + Live.  
- Low startup (<2s), ABR, QoE monitoring.  
- Secure (DRM, signed URLs).  
- Cost-effective (CDN heavy use).

**Architecture flow:**  
1. **Upload:** Clients upload via signed S3 URLs.  
2. **Transcoding:** Worker fleet (FFmpeg) → renditions (multi-bitrate) → HLS/DASH segments.  
3. **Storage:** Store chunks + manifests in S3, versioned.  
4. **Delivery:** CDN edges cache segments globally; Node origin signs tokens, serves manifests.  
5. **Playback:** Client fetches manifest, adapts bitrate (ABR).  
6. **Monitoring:** QoE logs (startup, rebuffer) → Kafka → Analytics pipeline.  

**Key Trade-offs:**  
- CDN offloads 90%+ traffic → cheaper, faster.  
- Shorter segments → low latency, higher request load.  
- Pre-warm caches for trending shows.  
- Use LL-HLS/WebRTC for low-latency live events.  

**Interview sound-byte:**  
> “We transcode videos into multiple bitrates and segments, store in S3, serve via CDN, and control access with signed tokens. Node handles manifests, metadata, and QoE ingestion. CDN handles 95%+ of traffic. For consistency, we use outbox/event-driven workflows. For live, we use LL-HLS/WebRTC. Scaling is mainly about CDN edge capacity and autoscaled transcoder workers.”

---

## 📎 Final Notes

- Use case dictates DB: SQL for transactions, NoSQL/Elastic for catalogs.  
- Always mention **caching + queueing + observability**.  
- Consistency patterns: **Saga + Outbox**.  
- Scaling patterns: **Cluster, Redis, CDN, Kubernetes**.  
- Security: **JWT, mTLS, TLS everywhere**.  
- JWT is for authenticating the user/consumer, mTLS is for authenticating the services. Together, they give layered security in microservices.

---