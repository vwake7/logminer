# Log Miner — Project Instructions

## 1. Project Objective

Build an online log-template mining algorithm in Go.

The algorithm should be independently implemented and inspired by the  
Drain/Drain3 family of log parsers.

The eventual purpose is to process legacy Tomcat/JVM application logs  
and produce structured events that can support:

- Component discovery
    
- Dependency discovery
    
- Topology discovery
    
- Anomaly detection
    
- Context DB construction
    
- SLM-based RCA
    

The log miner should eventually be suitable for deployment as a  
lightweight agent on legacy application VMs.

The core parser must be deterministic and must not depend on an LLM.

---

## 2. Technology

Use:

- Go 1.26+
    
- Go modules
    
- Standard library wherever practical
    
- `go test` for testing
    
- `go benchmark` / Go benchmarking facilities for performance testing
    

Avoid unnecessary third-party dependencies.

The final log-miner agent should be capable of being distributed as a  
single statically compiled executable wherever practical.

---

## 3. Algorithm Scope

The initial implementation should focus on:

1. Log tokenization
    
2. Parsing tree
    
3. Candidate cluster selection
    
4. Token similarity
    
5. Cluster matching
    
6. Template generalization
    
7. Parameter extraction
    
8. Online/streaming processing
    

Do not implement semantic extraction, topology discovery or SLM/RCA  
until the core log miner is working and benchmarked.

---

## 4. Important Constraint

Do NOT copy the Drain3 implementation or reproduce its source code.

Use the Drain/Drain3 algorithmic concepts as inspiration and implement  
the algorithm independently.

The implementation should be understandable and modular rather than  
being a direct port or clone.

Document important deviations from the original Drain algorithm.

---

## 5. Production Design Principles

The implementation should be designed as a production observability  
component rather than only as an academic prototype.

Prioritize:

- Low CPU usage
    
- Low memory usage
    
- Predictable memory growth
    
- High throughput
    
- Low per-message latency
    
- Concurrent processing where appropriate
    
- Safe handling of malformed input
    
- Deterministic behavior
    
- Graceful shutdown
    
- Configurability
    
- Testability
    
- Observability of the log miner itself
    

Do not introduce concurrency prematurely into the core algorithm.

First establish correctness with a deterministic single-threaded  
implementation.

Then evaluate where concurrency provides measurable benefit.

---

# 6. Development Principles

Work incrementally.

For each significant change:

1. Explain the proposed approach.
    
2. Identify the files that will change.
    
3. Implement the smallest useful increment.
    
4. Write or update tests.
    
5. Run the test suite.
    
6. Report the results.
    
7. Do not make unrelated changes.
    

Do not implement the entire system in one step.

Prefer simple, readable Go over premature optimization.

Use idiomatic Go.

Use explicit types and clear interfaces where they improve  
maintainability.

Avoid unnecessary abstractions.

Keep algorithmic logic independent from:

- File readers
    
- Network receivers
    
- Storage
    
- Metrics
    
- Logging
    
- Configuration
    
- Future LLM/SLM integration
    

---

# 7. Initial Task — Algorithm Design

We are currently at the architecture/design stage.

Do NOT modify any files yet.

First study the Drain algorithm conceptually and explain how we  
should independently implement it in Go.

Cover:

### 7.1 Tokenization

Explain how a raw log message should be converted into tokens.

Consider:

- timestamps
    
- log levels
    
- class names
    
- IP addresses
    
- hostnames
    
- UUIDs
    
- numbers
    
- URLs
    
- identifiers
    
- punctuation
    
- exception messages
    

The tokenizer should preserve enough information to allow later  
parameter extraction.

---

### 7.2 Parsing Tree

Explain:

- root node
    
- message-length branching
    
- token-position branching
    
- tree depth
    
- maximum children
    
- wildcard handling
    
- candidate lookup
    

Show the tree structure with an example.

Propose appropriate Go data structures.

Consider whether slices, maps, structs, pointers or interfaces are  
appropriate for each part of the tree.

---

### 7.3 Candidate Cluster Selection

Explain how an incoming message should locate potential existing  
clusters without comparing against every cluster.

The design should minimize unnecessary allocations and avoid scanning  
all clusters whenever possible.

---

### 7.4 Similarity Calculation

Explain how similarity between an incoming message and an existing  
cluster/template should be calculated.

Include:

- matching tokens
    
- wildcard tokens
    
- similarity threshold
    
- tie-breaking
    
- configurable thresholds
    

Discuss the CPU complexity of the similarity calculation.

---

### 7.5 Cluster Creation

Explain when a new cluster should be created.

Define the information stored in a cluster.

Consider:

- cluster ID
    
- template
    
- token count
    
- message count
    
- first seen timestamp
    
- last seen timestamp
    
- parameter information
    

---

### 7.6 Template Generalization

Example:

Message 1:

ERROR OrderService failed for customer 12345

Message 2:

ERROR OrderService failed for customer 67891

Expected template:

ERROR OrderService failed for customer <*>

Explain how the template should be updated incrementally.

The algorithm should preserve constant tokens and replace variable  
tokens with a wildcard representation.

---

### 7.7 Parameter Extraction

For:

ERROR OrderService failed for customer 12345

the parser should be able to return:

Template:

ERROR OrderService failed for customer <*>

Parameters:

12345

Explain an appropriate Go representation for:

- Template
    
- Parameters
    
- Structured log event
    
- Cluster
    

---

### 7.8 Online Processing

Explain how the algorithm behaves when millions of messages arrive  
incrementally.

Discuss:

- cluster growth
    
- cluster statistics
    
- memory management
    
- persistence
    
- restart/recovery
    
- pruning strategies
    
- maximum cluster limits
    

The core API should support processing one log message at a time.

For example, conceptually:

```
event := miner.Process(message)
```

Do not assume this exact API; propose the most appropriate design.

---

### 7.9 Concurrency

Analyze whether the core parser should be:

- single-threaded
    
- protected by a mutex
    
- sharded
    
- worker-based
    
- or otherwise concurrent
    

Consider the trade-off between:

- throughput
    
- ordering
    
- memory
    
- lock contention
    
- deterministic behavior
    

Do not implement concurrency yet.

First propose the design.

---

### 7.10 Complexity

Explain:

- insertion complexity
    
- lookup complexity
    
- similarity complexity
    
- memory complexity
    

Compare this with naive comparison against every existing template.

---

# 8. Proposed Go Architecture

After explaining the algorithm, propose a clean Go module structure.

Potential areas include:

- tokenizer
    
- parser
    
- parsing tree
    
- cluster
    
- template
    
- similarity
    
- configuration
    
- persistence
    

A possible structure is:

```
log-miner/
├── cmd/
│   └── logminer/
│       └── main.go
├── internal/
│   ├── tokenizer/
│   ├── parser/
│   ├── tree/
│   ├── cluster/
│   ├── template/
│   └── similarity/
├── testdata/
├── benchmarks/
├── datasets/
├── go.mod
└── AGENTS.md
```

Do not assume this structure is mandatory.

Recommend the architecture that best fits the algorithm.

Explain the responsibility of each package.

Avoid creating packages merely for organizational purposes.

---

# 9. API Design

Propose the public API for the core log miner.

The API should eventually support:

- Creating a miner
    
- Processing a log message
    
- Retrieving a template
    
- Retrieving cluster statistics
    
- Exporting discovered templates
    
- Resetting/pruning state
    
- Persisting state
    
- Loading state
    

Keep the core API small.

Avoid exposing internal parsing-tree implementation details.

---

# 10. Testing Strategy

Use Go's standard testing framework.

Design tests for:

- empty messages
    
- single messages
    
- identical messages
    
- messages with numeric parameters
    
- messages with UUIDs
    
- messages with IP addresses
    
- messages with hostnames
    
- messages with different token lengths
    
- partially matching messages
    
- completely different messages
    
- cluster creation
    
- cluster updates
    
- template generalization
    
- parameter extraction
    
- similarity thresholds
    
- high-cardinality parameters
    
- streaming behavior
    
- malformed input
    
- very long log messages
    

Include regression tests for every discovered bug.

Use table-driven tests where appropriate.

---

# 11. Benchmark Strategy

Build Go benchmarks using the standard benchmarking framework.

Eventually benchmark:

- 10K messages
    
- 100K messages
    
- 1M messages
    
- 10M messages
    

Measure:

- messages/second
    
- processing latency
    
- allocations per message
    
- bytes allocated per message
    
- memory usage
    
- number of clusters
    
- template accuracy
    
- parameter extraction accuracy
    

Where useful, use CPU and memory profiling.

The benchmark methodology must be reproducible.

Later compare the implementation against:

- Drain3
    
- LenMa
    

The comparison should clearly document:

- hardware
    
- dataset
    
- configuration
    
- workload
    
- methodology
    
- accuracy metrics
    
- throughput
    
- memory usage
    

---

# 12. Synthetic Log Dataset

Later create a deterministic synthetic log generator representing  
legacy Tomcat/JVM applications.

Include:

- HTTP requests
    
- JDBC calls
    
- SQL Server connection failures
    
- connection pool exhaustion
    
- application exceptions
    
- authentication failures
    
- timeouts
    
- thread pool messages
    
- garbage collection
    
- Tomcat lifecycle events
    

The generator should support:

- configurable message volume
    
- configurable template count
    
- parameter cardinality
    
- noise
    
- template frequency
    
- anomaly injection
    

Use fixed random seeds for reproducibility.

The generator may be implemented in Go or another language if there  
is a strong reason to do so, but the generated dataset must be  
language-independent.

---

# 13. Real Log Evaluation

Once the core miner works, evaluate it against representative  
Tomcat/JVM application logs.

Identify:

1. Log formats
    
2. Variable fields
    
3. Repeated patterns
    
4. Exception patterns
    
5. Database-related patterns
    
6. HTTP-related patterns
    
7. Component identifiers
    
8. Potential dependency identifiers
    

Produce a dataset specification that can be converted into regression  
tests.

Do not modify the parser simply to fit one dataset.

---

# 14. Semantic Extraction

Only after the deterministic log parser is working should we build a  
semantic extraction layer.

The parser should produce structured events containing information  
such as:

- Timestamp
    
- Template ID
    
- Template
    
- Parameters
    
- Source host
    
- Log level
    
- Original message where appropriate
    

The semantic layer should identify:

- Component
    
- Dependency
    
- Dependency type
    
- Protocol
    
- Host
    
- Port
    
- Operation
    
- Error type
    

Keep semantic extraction separate from the core log parser.

Do not introduce an LLM into the core parser.

---

# 15. Context Discovery

Using the structured log events, investigate how we can infer:

- Components
    
- Dependencies
    
- Relationships
    
- Application topology
    
- Anomalies
    

The objective is to produce a Context DB containing three initial  
entities:

1. Components
    
2. Topology
    
3. Anomalies
    

Do not implement this during the initial parser phase.

First propose the inference model and explain its limitations.

---

# 16. Future Architecture

Once the deterministic log miner is working:

```
Raw Logs
    ↓
Go Log Miner
    ↓
Structured Events
    ↓
Semantic Extraction
    ↓
Components + Dependencies
    ↓
Topology + Anomalies
    ↓
Causeora Context DB
    ↓
SLM
    ↓
RCA
```

The SLM should consume structured context rather than raw logs  
wherever possible.

---

# 17. Future Deployment Model

The eventual production deployment may look like:

```
Legacy Application VM
┌──────────────────────────────┐
│                              │
│  Tomcat / JVM                │
│      │                       │
│      ├── catalina logs       │
│      └── application logs    │
│                              │
│      ↓                       │
│                              │
│  Causeora Log Miner          │
│      │                       │
└──────┼───────────────────────┘
       │
       ↓
  Structured Events
       │
       ↓
  Causeora Context DB
```

The agent should eventually be capable of:

- Reading log files
    
- Tracking file offsets
    
- Handling log rotation
    
- Recovering after restart
    
- Processing multiple log sources
    
- Reporting its own health