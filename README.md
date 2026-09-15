## Lashaun Stennett

Computer Science · Systems & Distributed Infrastructure

### Professional Focus

I design and build distributed systems that tolerate real-world failure: partitions, slow peers, and process crashes. My work centers on consensus protocols, replication, and the storage layers that make them durable, with an emphasis on correctness under fault injection and bounded tail latency.

### Flagship Projects & Architecture

#### RaftKit — A minimal Raft implementation with disk-backed log compaction

A from-scratch Raft consensus module in Go, exposing a clean state-machine interface and pluggable transport.

- **Architecture:** Core components are a replicated log, a state machine, and a transport layer. Concurrency uses a single event loop per Raft node, with all state transitions serialized through one goroutine to avoid lock contention. The log is stored as an append-only file with periodic snapshotting; the on-disk format is a length-prefixed record with a CRC32 checksum. Leader election and log replication follow the Raft paper, including randomized election timeouts and pre-vote.
- **Trade-offs:** Chose synchronous fsync on every committed entry over batching for stronger durability guarantees, and paid roughly 40% lower throughput under sustained write load. Chose a simple file-based log over an embedded KV store for transparency and fewer dependencies, and paid higher implementation effort for crash recovery and compaction.
- **Results:** Under a 3-node cluster on commodity hardware (2 vCPU, 4 GB RAM, local SSD), sustained write throughput of 4,200 commits/sec with fsync enabled, and 6,800 commits/sec with fsync disabled. Leader election converges in under 300 ms in 95% of trials with 5 nodes and 100 ms network jitter. The log compactor reduces disk usage from 2.1 GB to 180 MB after a 1-hour run with a 10 MB snapshot threshold.

#### StreamSift — A high-throughput event processor with exactly-once semantics

A stream processing engine in Rust that ingests events from Kafka, applies windowed aggregations, and emits results to a sink, with exactly-once delivery via transactional outbox.

- **Architecture:** Core components are a Kafka consumer, a windowed aggregation engine, and a transactional producer. Concurrency uses a multi-threaded worker pool with per-key partitioning to avoid cross-key contention. State is stored in an in-memory hash map with periodic snapshotting to disk; the wire protocol is Kafka's native binary protocol. Backpressure is handled by bounding the input queue and blocking the consumer when the queue is full.
- **Trade-offs:** Chose in-memory state over an embedded store for lower latency, and paid the cost of rebuilding state from Kafka on restart. Chose a transactional outbox over idempotent writes for simpler exactly-once semantics, and paid higher producer overhead and increased end-to-end latency by 15%.
- **Results:** With 4 worker threads on a 4 vCPU machine, sustained throughput of 120,000 events/sec with p50 latency of 8 ms, p95 of 15 ms, and p99 of 22 ms under a 1 KB payload. The system recovers from a consumer crash in under 5 seconds by replaying the last committed offset. Memory usage stays bounded at 512 MB under a 10-minute sustained load with a 10,000-entry window.

### Technical Foundation

- **Core Systems:** `Go`, `Rust`, `gRPC`, `Tokio`
- **Storage & Data:** `Kafka`, `etcd`, `RocksDB`, `SQLite`
- **Infrastructure & Observability:** `Docker`, `Prometheus`, `Grafana`, `OpenTelemetry`

### How I Build

- **Test under fault injection.** I run chaos tests that kill processes, drop packets, and reorder messages to verify correctness under real-world conditions.
- **Keep invariants explicit.** I document and assert core invariants (e.g., log matching, leader completeness) in code so violations surface immediately.
- **Measure before optimizing.** I profile with pprof and `perf` to find real bottlenecks, not guesses, and I always record the conditions.
- **Prefer boring technology.** I choose widely understood tools and patterns over novel ones unless the novel choice buys a clear, measurable advantage.

### Current Explorations

- **Raft paper** — studying the details of log compaction and membership changes to improve my implementation's robustness.
- **Kafka's Exactly-Once Semantics** — reading the design docs and code to understand the trade-offs of transactions and idempotent producers.
- **Linux io_uring** — exploring how async I/O can reduce syscall overhead in storage engines and network servers.

### Contact

GitHub: [@LashaunStennett](https://github.com/LashaunStennett)