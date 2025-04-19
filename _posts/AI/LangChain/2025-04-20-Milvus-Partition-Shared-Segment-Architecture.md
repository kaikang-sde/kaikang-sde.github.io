---
title: Milvus Partition-Shard-Segment Architecture
date: 2025-04-19
order: 2
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, Partition, Shard, Segment]
author: kai
---

# 🚀 Milvus Partition-Shard-Segment Architecture
When dealing with billion-scale vector data, Milvus organizes data using a three-layer structure: **Partitions**, **Shards**, and **Segments**. 

## 🧱 The Big Picture: A Giant Library (Milvus Collection)

Imagine managing a **mega library** (representing a **Milvus Collection**) that stores **hundreds of millions of books** (entities/vectors). To manage this efficiently, the library uses 3 types of logical and physical organization:


### 1. Partition = Floor Sections by Topic

#### Analogy:
> The library has different *floors or areas* based on book categories:  
> - 📘 1st Floor: **Science & Tech**  
> - 📙 2nd Floor: **Literature**  
> - 📕 3rd Floor: **Arts**

#### In Milvus:
- A **Partition** is a **logical subset** of a Collection.
- Used to **isolate or filter data by group** (e.g., by category, region, or date).
- Helps **narrow down search scope**, improving retrieval performance.

**Best Practice**:  
Use partitions when you want to filter data upfront (e.g., `WHERE category = 'electronics'`).



### 2. Shard = Multiple Parallel Shelves Within a Partition

#### Analogy:
> Inside each themed floor, there are **10 identical bookshelves**, each storing a portion of the books.  
> Multiple readers can explore **different shelves simultaneously**.

#### In Milvus:
- A **Shard** enables **horizontal scaling**.
- Each partition can be **sharded** across multiple nodes.
- Ensures **write/read throughput scales linearly**.

**Best Practice**:  
Let Milvus auto-shard (default = 2 shards per collection). Don’t over-shard small datasets.


### 3. Segment = Modular Book Boxes on a Shelf

#### Analogy:
> Each bookshelf contains **book boxes**.  
> - New books are first placed in an **"open box"** (growing segment).  
> - Once full, the box is **sealed** (sealed segment) and optimized for storage.

#### In Milvus:
- A **Segment** is a **physical storage unit**.
- Comes in two types:
  - `Growing Segment`: For new inserts.
  - `Sealed Segment`: Read-only, optimized, searchable.

**Best Practice**:  
- Insert data in **batches** to fill segments efficiently.  
- Avoid tiny inserts (may lead to fragmented segments and slower search).


## 📐 Collection, Partition, and Shard

![Partition-Shard-Segment](/assets/img/posts/AI/LangChain/PartitionShardSegment.png)

## 🗞️ Partition, Shard, and Segment

| Dimension     | Partition         | Shard     | Segment     |
|---------------|-------------------|-----------|-------------|
| **Level**     | Logical division  | Physical distribution  | Physical storage unit |
| **Visibility**| Created and managed **by user** | Auto-assigned **by system**  | Fully managed **by system** |
| **Purpose**   | Business data isolation  | Load balancing and scalability  | Storage optimization and query speedup |
| **Analogy**   | Library floors by subject | Identical bookshelves on each floor  | Swappable boxes on each bookshelf  |
| **Operation** | Manually specify in queries | Automatically routes to different nodes | Automatically merged/compressed by system |


## 🛒 Practical Workflow Example: Milvus in E-commerce Data Upload

**Scenario**: A user uploads 100,000 product entries to an e-commerce platform using Milvus as the vector database.

### Step 1: Partitioning by Business Dimension

Partitions allow logical separation of data to improve query precision and reduce scan ranges.

#### Goal:
- Organize data by **domain-specific needs** (e.g., product category, upload date, user type).

#### Example Partitions:
- `partition_2025Q1` – Products uploaded in Q1 2025.
- `vip_users` – Data uploaded by VIP merchants.

#### Code Example: Create Product Category Partitions

To segment data by product type, partitions are created for each category:

```python
# Create partitions for electronics and clothing
collection.create_partition("electronics")
collection.create_partition("clothing")
```

### Step 2: Sharding (Automatically Handled)

**Shards** enable parallelism and scalability by distributing data across multiple compute nodes in the cluster.

#### Behavior:
- Milvus automatically divides inserted data into multiple **shards**.
- For example, if the cluster has **3 nodes**, the data is **evenly distributed across 3 shards**.

#### Benefits:
- Ensures **load balancing** across nodes.
- Enables **parallel ingestion and querying** for high throughput.

![Sharding](/assets/img/posts/AI/LangChain/Sharding.png)

> 💡 Developers don’t need to manually manage shards — Milvus handles this internally based on cluster configuration and data volume.


### Step 3: Segmenting (Automatically Handled)

**Segments** are the physical storage units within each shard.

#### Behavior:
- Within each **shard**, data is automatically split into **segments**.
- Each segment is typically capped at **512MB** in size.
- As data grows, Milvus creates new segments and marks full ones as **sealed** for optimized querying.

#### Benefits:
- Enables efficient storage management.
- Improves query speed by allowing parallel access to smaller data blocks.
- Sealed segments are indexed for fast similarity search.

![Segmenting](/assets/img/posts/AI/LangChain/Segment.png)

> 💡 Like sharding, segment management is fully automated. Developers don't need to configure or control this process.


## 🔄 How Partitions, Shards, and Segments Work Together in Milvus

### Data Write Process

1. **New Data Ingested**
2. ➡️ Routed to a specific **Partition** (e.g., by category or time)
3. ➡️ Automatically assigned to a **Shard** (based on hash of primary key)
4. ➡️ Written into an **Active Segment** within the shard
5. 📦 When the segment reaches its size limit (e.g., 512MB), it becomes **sealed**
6. 🔄 A new active segment is created for continued writes


### Query Process

1. **User Query Received**
2. ➡️ Identify relevant **Partitions** based on filter (e.g., category = "electronics")
3. ➡️ Dispatch query **in parallel** to all matching **Shards**
4. ➡️ Each shard scans all of its **Segments**
5. ➡️ Partial results are gathered and globally **merged & ranked**


### Segment Compaction (Automatic Optimization)

- Small segments are periodically **merged** to improve query efficiency:

```txt
Segment1 (100MB) + Segment2 (100MB) → MergedSegment (200MB)
```

## 🛠️ Development Tips for Using Milvus Effectively
When developing applications with Milvus, especially for large-scale vector search systems, it's essential to follow best practices in data modeling, indexing, and system resource management. 

### Partition Management

- **Use Case-Driven Partitioning**
  - Partition by **time** (e.g., `2024-Q1`) or **business logic** (e.g., `vip_users`, `product_catalog`)
  - Helps in narrowing query scope and improving performance

- **Avoid Over-Partitioning**
  - **Don’t** create partitions per user or document
  - Keep the number of partitions well below **1000**
  - Too many partitions may lead to increased metadata overhead and degraded performance


### Sharding Configuration

- **Optimal Shard Count = Number of Nodes × Number of CPU Cores per Node**
- This formula ensures:
  - Balanced workload distribution
  - Efficient CPU utilization
  - Scalable parallel writes


- **Avoid Over-sharding on Low-Core Machines**
- Example: Setting **128 shards** on an **8-core machine**  
  - 🧨 Leads to frequent context switching
  - 🐢 Results in degraded performance due to overhead


| Shard Count     | Characteristics                         | Performance Impact               |
|-----------------|------------------------------------------|----------------------------------|
| **Too Few**     | Large data per shard                    | ⚠️ Lower write throughput, potential bottleneck |
| **Too Many**    | Small data per shard                    | ⚠️ High resource consumption, increased coordination overhead |
| **Balanced**    | Matches node/core capacity              | ✅ Optimal parallelism, stable performance |


### Example: Create a Collection with Shards and Partitions

```python
from pymilvus import Collection, CollectionSchema

collection = Collection(
    name="product_images",
    shards_num=64,  # Shards = 8 nodes × 8 cores = 64
    partitions=[
        "electronics",       # Electronics category
        "clothing",          # Clothing category
        "home_appliances"    # Home appliances category
    ]
)

client.set_property("dataCoord.segment.maxSize", "1024")       # Max segment size: 1 GB
client.set_property("dataCoord.segment.sealProportion", "0.7") # Seal when 70% full
```

### Segment Optimization Strategie
- **Monitor Segment Status:** 
  - To retrieve current segment information: `collection.get_segment_info()`
    - Current segment utilization
    - Growth rate and sealing patterns
    - Query performance implications

- **Manually Trigger Segment Compaction**
  - When too many small segments degrade query efficiency, trigger a merge operation: `collection.compact()`
    - Merges smaller sealed segments into larger ones
    - Reduces I/O overhead
    - Optimizes index performance

- **Configure Segment Size Threshold**
  - Set segment max size via storage config to control when segments are sealed: `storage.segmentSize = 1024`  # unit: MB
    - Maximum allowed segment size before sealing
    - Balance between write throughput and memory pressure

- **Adaptive Segment Size (Based on Vector Dimensions)**
  - can dynamically configure segment size based on the complexity of your vector data:
    ```python
    if vector_dim > 1024:
        maxSize = 512  # Reduce to relieve memory pressure
    else:
        maxSize = 1024  # Use standard size for moderate dimensions
    ```
    - High-dimensional vectors consume more memory
    - Reducing maxSize can prevent memory overload during ingestion







<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



