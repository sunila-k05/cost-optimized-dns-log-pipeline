# DNS Log Processing Pipeline (Buffer-First Architecture)

## Problem
Design a DNS log pipeline with:
- 5,000 logs/sec baseline  
- 100,000 logs/sec spike (10 min)  
- Zero data loss  
- Budget ≤ $800/month  
- Buffer-first architecture  

---

## Solution Overview
A cost optimized, buffer first system where logs are **persisted before processing** to handle spikes safely.

DNS Logs → Vector (Disk Buffer) → S3 (Parquet + ZSTD) → ClickHouse → Reports

---

## Design Approach (Layer-wise)

### 1. Ingestion
- **Tool:** Vector (runs on DNS hosts)
- **Why:** Lightweight + disk buffering support
- **Throughput:** Handles 100K logs/sec via local buffering

### 2. Buffering (Core Layer)
- **Tool:** Disk buffer (100GB) + S3
- **Why:** Disk is cheaper than RAM, S3 is durable and scalable
- **Strategy:** Batch upload (10MB / 30s)

### 3. Processing
- **Tool:** Lightweight batch jobs
- **Why:** Avoids costly real-time engines (Spark/Flink)
- **Approach:** Async processing from S3

### 4. Storage
- **Tool:** ClickHouse
- **Why:** High compression (ZSTD), efficient for large-scale analytics
- **Optimization:** MergeTree + async inserts + lifecycle to S3

---

##  Cost Strategy
- Avoided Kinesis / Splunk / Datadog (>$3000/month at scale)
- Used:
  - Disk buffering instead of memory
  - S3 (cheap object storage)
  - ClickHouse compression (~90%)

**Estimated total cost: < $800/month**

---

##  Traffic Handling

| Scenario        | Strategy |
|----------------|---------|
| Normal (5K/s)  | Direct ingestion |
| Spike (100K/s) | Disk buffer + batching |
| Failure        | Retry from buffer |
| Long spike     | Sampling / throttling |

---

##  Zero Data Loss
- Disk buffer prevents drops  
- At-least-once delivery  
- Retry on S3 failure  
- Data persisted before processing  

---

##  Reporting
- Daily reports: top domains, query counts, sources  
- Formats: PDF, HTML, JSON  
- Stored in S3 with permanent access  

---

##  Key Highlights
- Handles **20× spike (5K → 100K logs/sec)**  
- Ensures **zero data loss**  
- Fully **cost-optimized under $800**  
- Uses **open-source tools only**  

---

##  Key Trade-off
Chose **disk + S3 buffering over Kafka/Kinesis**:
- Lower cost  
- Simpler operations  
- Slight delay due to batch processing  

---

##  Outcome
A scalable, fault-tolerant DNS log pipeline that prioritizes **reliability, cost-efficiency, and real-world constraints**.
