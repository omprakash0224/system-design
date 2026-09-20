# System Design & Distributed Systems Knowledge Base

> A curated repository documenting end-to-end system design case studies, distributed system architectural patterns, core building blocks, and back-of-the-envelope estimation guides as I learn and master large-scale system design.

---

## Table of Contents

- [Overview](#overview)
- [System Design Case Studies](#system-design-case-studies)
- [System Design Blueprint (Framework)](#system-design-blueprint-framework)
- [Core Fundamentals & Building Blocks](#core-fundamentals--building-blocks)
- [Back-of-the-Envelope Estimation Cheat Sheet](#back-of-the-envelope-estimation-cheat-sheet)
- [Roadmap & Upcoming Topics](#roadmap--upcoming-topics)
- [References & Recommended Reading](#references--recommended-reading)

---

## Overview

Designing resilient, highly available, and horizontally scalable distributed systems requires a blend of theoretical fundamentals (CAP theorem, consensus algorithms, replication models) and practical engineering trade-offs (storage engines, caching hierarchies, fan-out strategies).

This repository serves as a personal knowledge hub containing:
- **Production-grade System Design documents** modeled after real-world tech giants.
- **Deep dives** into critical components (Snowflake IDs, consistent hashing, LSM-trees vs. B-Trees, etc.).
- **Visual architecture diagrams** created with Mermaid.js.
- **Formulas and heuristics** for estimation and capacity planning.

---

## System Design Case Studies

| Case Study | Key Challenges & Concepts Covered | Document |
| :--- | :--- | :--- |
| **Twitter / X — Home Timeline (News Feed)** | Hybrid Fan-out (Push vs. Pull), Celebrity Problem, Snowflake ID, Redis ZSET timeline cache, real-time trend detection, cursor pagination | [View Design](news_feed_system_design.md) |

*(More case studies will be added as they are created — see [Roadmap](#roadmap--upcoming-topics))*

---

## System Design Blueprint (Framework)

When approaching any system design problem, each case study follows a standardized 6-step blueprint:

1. **Requirements Clarification**
   - **Functional Requirements:** Core features, user flows, input/output specifications.
   - **Non-Functional Requirements:** Availability ($99.99\%$), Latency ($p99 \le 200\text{ ms}$), Consistency (Strong vs. Eventual), Throughput.
2. **Capacity Estimation & Scale (Back-of-the-Envelope)**
   - Traffic estimation (Write QPS, Read QPS, Peak multipliers).
   - Storage estimation (daily, 5-year, replication factor).
   - Memory/Cache requirements ($80/20$ Pareto rule).
   - Network bandwidth (egress/ingress).
3. **API & Interface Design**
   - RESTful / gRPC / GraphQL contract definitions.
   - Request and response payloads, status codes, authentication headers.
4. **Data Models & Storage Selection**
   - Entity Relationship (ER) diagrams.
   - SQL vs. NoSQL vs. Wide-Column vs. Time-Series vs. Graph database trade-offs.
   - Sharding / Partition keys and indexing strategies.
5. **High-Level & Detailed Architecture**
   - Component diagrams, data flow sequences, asynchronous messaging (Kafka/RabbitMQ).
   - Cache hierarchy (CDN, L1 application cache, L2 distributed cache).
6. **Deep Dives, Bottlenecks & Edge Cases**
   - Hotspot / celebrity partitions.
   - Failure modes, circuit breakers, fallback patterns, disaster recovery.

---

## Core Fundamentals & Building Blocks

### 1. Scaling & Architecture Patterns
- **Vertical Scaling (Scale Up)** vs. **Horizontal Scaling (Scale Out)**
- **Stateless vs. Stateful Services**
- **Load Balancing:** Layer 4 (TCP/UDP) vs. Layer 7 (HTTP/HTTPS), Algorithms (Round Robin, Weighted Least Connections, IP Hash, Consistent Hashing).
- **API Gateway & Reverse Proxy:** Rate limiting (Token Bucket, Leaky Bucket, Sliding Window Log), authentication, SSL termination, request routing.

### 2. Data Storage & Partitioning
- **RDBMS vs. NoSQL:** ACID guarantees vs. BASE properties.
- **Storage Engines:** B-Tree (PostgreSQL, MySQL InnoDB) vs. LSM-Tree (ScyllaDB, Cassandra, RocksDB).
- **Database Partitioning & Sharding:** Horizontal vs. Vertical sharding, range-based, hash-based, directory-based.
- **Replication:** Single-Leader, Multi-Leader, Leaderless (Dynamo-style quorum $R + W > N$).
- **Distributed Transactions:** Two-Phase Commit (2PC), Saga Pattern (Orchestration vs. Choreography).

### 3. Caching Strategies
- **Cache Patterns:** Cache-Aside, Read-Through, Write-Through, Write-Around, Write-Back.
- **Eviction Policies:** LRU (Least Recently Used), LFU (Least Frequently Used), FIFO, TTL.
- **Cache Pitfalls & Mitigations:**
  - *Cache Avalanche:* Use jitter/randomized TTLs.
  - *Cache Penetration:* Bloom filters or caching null results.
  - *Cache Stampede / Thundering Herd:* Mutex locks, pre-computation.

### 4. Messaging & Asynchronous Processing
- **Message Queues vs. Event Streams:** RabbitMQ / SQS vs. Apache Kafka / Apache Pulsar.
- **Delivery Guarantees:** At-most-once, At-least-once, Exactly-once (idempotency keys).
- **Push vs. Pull:** Long Polling, WebSockets, Server-Sent Events (SSE).

---

## Back-of-the-Envelope Estimation Cheat Sheet

### Powers of Two & Storage Conversions
| Power | Approximate Value | Metric Name |
| :--- | :--- | :--- |
| $2^{10}$ | $1{,}000$ ($10^3$) | 1 Kilobyte (KB) |
| $2^{20}$ | $1{,}000{,}000$ ($10^6$) | 1 Megabyte (MB) |
| $2^{30}$ | $1{,}000{,}000{,}000$ ($10^9$) | 1 Gigabyte (GB) |
| $2^{40}$ | $1{,}000{,}000{,}000{,}000$ ($10^{12}$) | 1 Terabyte (TB) |
| $2^{50}$ | $10^{15}$ | 1 Petabyte (PB) |

### Latency Numbers Every Programmer Should Know
| Operation | Latency |
| :--- | :--- |
| L1 Cache Reference | $\approx 0.5\text{ ns}$ |
| Branch Mispredict | $\approx 5\text{ ns}$ |
| L2 Cache Reference | $\approx 7\text{ ns}$ |
| Mutex Lock / Unlock | $\approx 25\text{ ns}$ |
| Main Memory (RAM) Reference | $\approx 100\text{ ns}$ |
| Read $1\text{ MB}$ sequentially from RAM | $\approx 250{,}000\text{ ns}$ ($250\ \mu\text{s}$) |
| SSD Random Read | $\approx 150\ \mu\text{s}$ |
| Read $1\text{ MB}$ sequentially from SSD | $\approx 1\text{ ms}$ |
| HDD Seek | $\approx 10\text{ ms}$ |
| Read $1\text{ MB}$ sequentially from HDD | $\approx 20\text{ ms}$ |
| Send packet California to Netherlands & back | $\approx 150\text{ ms}$ |

### Handy Rules of Thumb
- **Seconds in a day:** $86{,}400 \approx 10^5 \text{ seconds}$.
- **QPS Conversion:** $\frac{X\text{ million requests/day}}{100{,}000\text{ s}} \approx 10 \times X \text{ QPS}$ (Average).
- **Peak QPS:** Typically assume $2\times$ to $3\times$ average QPS.
- **Cache Sizing (80/20 Rule):** Cache $20\%$ of daily read traffic in RAM.

---

## Roadmap & Upcoming Topics

- [x] **Twitter / X — Home Timeline (News Feed)**
- [ ] **TinyURL / Bitly** — Distributed URL Shortener with Base62 encoding & deduplication
- [ ] **WhatsApp / Slack** — Real-Time 1:1 and Group Chat System (WebSockets, Presence, Mnesia/Cassandra)
- [ ] **YouTube / Netflix** — Video Streaming & Transcoding Pipeline (Chunking, CDN, Adaptive Bitrate DASH/HLS)
- [ ] **Uber / Lyft** — Proximity Service & Geohashing (Quadtree, Google S2, Spatial Indexing)
- [ ] **Google Drive / Dropbox** — Distributed File Storage & Block Synchronization (Chunking, Merkle Trees)
- [ ] **Distributed Rate Limiter** — Redis Token Bucket with Lua scripting
- [ ] **Distributed Web Crawler** — Politeness, Deduplication, Frontier Queues
- [ ] **Ticketmaster / Flash Sale System** — Distributed Locking, Inventory Reservation, Anti-Overselling

---

## References & Recommended Reading

- **Books:**
  - *Designing Data-Intensive Applications (DDIA)* — Martin Kleppmann
  - *System Design Interview (Vols. 1 & 2)* — Alex Xu & Sahn Lam
  - *Database Internals* — Alex Petrov
  - *Site Reliability Engineering (SRE)* — Google
- **Engineering Blogs:**
  - [Meta Engineering](https://engineering.fb.com/)
  - [Netflix TechBlog](https://netflixtechblog.com/)
  - [Uber Engineering](https://eng.uber.com/)
  - [Amazon Science & Architecture](https://www.amazon.science/)
  - [High Scalability Archive](http://highscalability.com/)
