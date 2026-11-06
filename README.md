# Gato: Optimizing Multi-Version Deterministic Concurrency Control

Gato is a research prototype that extends Bohm, a multi-version deterministic concurrency control (MVDCC) protocol, to investigate performance bottlenecks in transaction processing.

Gato focuses on two challenges: workload imbalance caused by static record partitioning during concurrency control, and execution delays caused by read dependencies under high contention.

The design combines dynamic partitioning for workload redistribution with the split-on-demand technique introduced by Caracal.

## 1. Background and Motivation

### Multi-Version Deterministic Concurrency Control

Multi-version deterministic concurrency control establishes transaction ordering before execution, allowing transactions to operate on predetermined record versions.

Bohm separates transaction processing into two phases:

1. **Concurrency Control (CC):** Establishes the ordering of transactions and prepares record versions.
2. **Execution (EX):** Executes transactions according to the predetermined ordering and accesses the corresponding record versions.

This separation reduces the need for conventional lock-based conflict resolution and transaction aborts.

However, Bohm's design introduces performance limitations under skewed and highly contended workloads.

### Challenge 1: Static Partitioning

During concurrency control, Bohm assigns records to fixed partitions, each managed by a designated CC thread.

Although this reduces synchronization requirements, it can create uneven workloads when transactions frequently access records belonging to particular partitions.

Consequently, some CC threads may become overloaded while others remain underutilized, limiting overall throughput.

### Challenge 2: Read Spinning

During execution, a transaction may need to read a record version whose corresponding write has not yet completed.

The transaction must wait for the required version to become available.

Under high contention, these dependencies can cause substantial waiting time and delay subsequent transaction processing.

## 2. Gato System Design

Gato addresses these challenges through two complementary mechanisms applied at different stages of transaction processing.

```text
            Transaction Workload
                    |
                    v
       +-------------------------+
       |  Concurrency Control    |
       |                         |
       |  Dynamic Partitioning   |
       |  - Track workload       |
       |  - Redistribute tasks   |
       |  - Balance CC threads   |
       +-------------------------+
                    |
                    v
       +-------------------------+
       |   Transaction Execution |
       |                         |
       |   Split-on-Demand       |
       |   - Manage dependencies |
       |   - Reduce read waiting |
       +-------------------------+
                    |
                    v
          Completed Transactions
```

### 2.1 Dynamic Partitioning

Gato introduces dynamic partitioning to mitigate load imbalance during the concurrency control phase.

Instead of relying exclusively on a fixed assignment of records to CC threads, the mechanism maintains workload information and adjusts task distribution according to thread load.

The design uses mapping and workload-tracking structures to identify overloaded threads and redistribute work to less heavily loaded threads.

The intended effect is to improve CC-thread utilization and reduce bottlenecks caused by skewed record access patterns.

### 2.2 Split-on-Demand

Gato also incorporates the split-on-demand approach from Caracal to address read-related waiting during transaction execution.

In Bohm, a read operation may be delayed when the required record version has not yet been produced by its corresponding write.

Split-on-demand seeks to reduce such delays by separating operations according to their dependencies and allowing eligible work to proceed without unnecessary waiting.

This mechanism complements dynamic partitioning: while dynamic partitioning targets workload imbalance in the CC phase, split-on-demand targets dependency-related delays in the EX phase.

## 3. Implementation

Gato is implemented in C++ as an extension of Bohm.

The repository contains the following top-level directories:

```text
gato/
├── mvdcc/
├── protocol/
└── README.md
```

The primary implementation is located in `mvdcc/`.

The implementation investigates dynamic workload distribution within the concurrency control stage of a multi-version deterministic transaction processing system.

## 4. Evaluation

Gato is evaluated against Bohm to investigate the effects of workload redistribution on transaction processing performance.

The current repository reports preliminary concurrency-control measurements using 64 worker threads, with approximately 73,000–77,000 transactions per second.

These measurements indicate improvements over Bohm in the evaluated configurations, particularly under write-intensive workloads.

The results should be interpreted within the scope of the corresponding benchmark configurations rather than as general performance guarantees.

## 5. Research Scope

Gato investigates how workload-aware task distribution and dependency management can improve multi-version deterministic transaction processing.

Its principal design objectives are:

- Reducing workload imbalance among concurrency control threads.
- Improving resource utilization under skewed workloads.
- Mitigating execution delays caused by read dependencies.
- Evaluating the scalability of deterministic transaction processing under contention.

## 6. References

1. J. M. Faleiro and D. J. Abadi. *Rethinking Serializable Multiversion Concurrency Control*. PVLDB, 2015.
2. D. Qin, A. D. Brown, and A. Goel. *Caracal: Contention Management with Deterministic Concurrency Control*. SOSP, 2021.
