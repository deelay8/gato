# Gato: Multi-Version Deterministic Concurrency Control

Gato is a research project that aims to improve the performance of multi-version deterministic concurrency control (MVDCC), building upon Bohm.

## Research Problem

Concurrency control is essential for maintaining consistency while processing database transactions concurrently. However, Bohm's static partitioning can lead to load imbalance, limiting parallelism and transaction throughput.

## Research Objective

Gato aims to improve transaction processing performance and scalability by addressing load imbalance in multi-version deterministic concurrency control.

## Approach

Gato introduces dynamic partitioning to redistribute transaction workloads across threads based on their current load. The system is implemented in C++ by extending Bohm.

The main implementation is located in [`mvdcc/`](./mvdcc).

## Results

Preliminary experiments with 64 worker threads showed that Gato maintained approximately 73,000–77,000 transactions per second during the concurrency control phase, outperforming Bohm, particularly under write-intensive workloads.
