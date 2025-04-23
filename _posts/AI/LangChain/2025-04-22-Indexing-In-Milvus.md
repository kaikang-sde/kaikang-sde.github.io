---
title: Indexing in Milvus - Best Practices and Strategy Guide
date: 2025-04-22
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, Indexing]
author: kai
---

# 🚀 Indexing in Milvus - Best Practices and Strategy Guide
Efficient vector search at scale requires well-optimized indexing. In Milvus, choosing the right index is crucial for balancing **query speed**, **recall accuracy**, and **resource efficiency**.


## 🔍 Why Do You Need Indexes?

- **Accelerated Search**: Avoid brute-force comparisons, enabling fast and scalable approximate nearest neighbor (ANN) search.
- **Reduced Resource Usage**: Lowers memory and CPU overhead during query execution.
- **Performance Optimization**: Especially beneficial for large collections or frequently accessed fields.


## 📚 Common Index Types in Milvus

| Index Type | Best Use Case                       | Memory Usage | Accuracy    | Build Speed |
|------------|-------------------------------------|--------------|-------------|-------------|
| `FLAT`     | Small datasets with exact search (< 1M vectors) | 🔴 High       | ✅ 100%      | ⚡ Fast     |
| `IVF_FLAT` | Balanced use case for millions of vectors       | 🟡 Medium     | ✅ 95–98%    | ⚡ Fast     |
| `HNSW`     | High-recall scenarios, real-time relevance      | 🔴 High       | ✅ 98–99%    | 🐢 Slow     |
| `DISKANN`  | Massive-scale datasets (>1B vectors)            | 🟢 Low        | ✅ 90–95%    | 🐢 Slowest  |


### Best Practices

- For **exact search** on small datasets: use `FLAT`.
- For **balanced performance**: `IVF_FLAT` is a popular choice.
- For **high recall quality** (e.g., semantic search): choose `HNSW`.
- For **web-scale deployments** with limited memory: go with `DISKANN`.


### Tip: When to Build Index?

- After inserting large batches of data.
- Before loading collections for high-performance search.
- When updating frequently accessed fields.


## 🧠 Creating Indexes in Milvus with PyMilvusClient
Define a collection and apply a vector index using the `MilvusClient` API.

```python
# Import MilvusClient, DataType to establish Milvus connection and operate data types
from pymilvus import MilvusClient, DataType

# Connect to Milvus server via MilvusClient and select database
client = MilvusClient(
    uri="http://54.163.61.180:19530",
    db_name="my_database")

# Step 1: Define schema
schema = MilvusClient.create_schema(
    auto_id=False,                 # IDs are manually assigned
    enable_dynamic_field=True      # Allows flexible key-value fields
)

# Step 2: Add fields into schema
schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True)
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=5)

# Step 3: Create collection with defined schema
client.create_collection(
    collection_name="customized_setup",
    schema=schema
)

# Step 4: Prepare index parameters
index_params = MilvusClient.prepare_index_params()

# Add index configuration
index_params.add_index(
    field_name="vector",            # Create index on vector field
    metric_type="COSINE",           # Similarity Calculation Options: L2, IP, COSINE
    index_type="IVF_FLAT",          # Index Types: FLAT, IVF_FLAT, HNSW, etc.
    index_name="vector_index",      # Index Name
    params={"nlist": 128}           # Recommended: sqrt(total_vectors)
)

# Step 5: Create index (asynchronous mode)
client.create_index(
    collection_name="customized_setup",
    index_params=index_params,
    sync=False                      # Set to True to wait for completion
)
```

![Indexing Creation Example](/assets/img/posts/AI/LangChain/MilvusIndexType.png)

## 🧾 Index Parameter Descriptions

| Parameter       | Description |
|-----------------|-------------|
| `field_name`    | Name of the field to index (must be a vector field). |
| `metric_type`   | Algorithm used to measure similarity between vectors.<br> Valid values include: `IP` (Inner Product), `L2` (Euclidean), `COSINE`, `JACCARD`, `HAMMING`.<br> Applicable only for vector fields. |
| `index_type`    | Type of index to be built (e.g., `FLAT`, `IVF_FLAT`, `HNSW`, `DISKANN`). |
| `index_name`    | Custom name for the index. |
| `params`        | Fine-tuning parameters for the selected index type (e.g., `{"nlist": 128}`). |
| `collection_name` | Name of the Milvus collection on which the index is built. |
| `sync`          | Controls whether the client waits for index creation to complete. <br/>- `True` (default): Blocking mode. Returns only after index is fully created.<br/>- `False`: Non-blocking mode. Returns immediately, index is built in the background. |


## 🔍 View Index Information
To inspect existing index configurations for a given collection, you can use the following functions with `MilvusClient`:

### List All Indexes in a Collection

```python
from pymilvus import MilvusClient

# Connect to Milvus server via MilvusClient and select database
client = MilvusClient(
    uri="http://54.163.61.180:19530",
    db_name="my_database")

index_info = client.list_indexes(
    collection_name="customized_setup"
)
print(index_info) # ['vector_index']
```

### Describe Detailed Index Configuration

```python
from pymilvus import MilvusClient

# Connect to Milvus server via MilvusClient and select database
client = MilvusClient(
    uri="http://54.163.61.180:19530",
    db_name="my_database")

idx_config = client.describe_index(
    collection_name="customized_setup",
    index_name="vector_index"
)
print(idx_config)

# Output: {'nlist': '128', 'index_type': 'IVF_FLAT', 'metric_type': 'COSINE', 'field_name': 'vector', 'index_name': 'vector_index', 'total_rows': 0, 'indexed_rows': 0, 'pending_index_rows': 0, 'state': 'Finished'}
```


## 🗑️ Delete Index

- Before deleting an index, ensure no active queries are using it.  
- After deletion, you must **rebuild the index** to enable efficient vector searches again.
- Collection must be released. Otherwise, will get MilvusException
  - <MilvusException: (code=65535, message=index cannot be dropped, collection is loaded, please release it first)>

### Example: Drop an Index

```python
client = MilvusClient(
    uri="http://54.163.61.180:19530",
    db_name="my_database")
    
# Drop the specified index from the collection
client.drop_index(
    collection_name="customized_setup",
    index_name="vector_index"
)
print("Index has been deleted.")
```


## ✅ Best Practices & Pitfall Guide for Milvus Schema and Index Management
Designing a robust schema and managing indexes properly is crucial for high-performance vector search.


### Schema Design Principles

| Aspect         | Recommendation |
|----------------|----------------|
| 🔑 Primary Key | Use auto-increment integer IDs to avoid collision. Do **not** use vector fields as primary keys. |
| 🧱 Field Limit | Keep total number of fields under **32** per collection. |
| 🔢 Vector Dim  | Must be pre-defined and **immutable** after creation. Plan ahead based on embedding model (e.g. 128, 768, 1536). |


### Index Selection Strategy

| Data Scale        | Recommended Index     |
|-------------------|------------------------|
| < 1 Million        | `FLAT` (Exact Search)  |
| 1M – 1B            | `IVF_FLAT` or `HNSW`   |
| > 1B               | `DISKANN`              |

- Always build indexes **after** data insertion is completed.
- Rebuild indexes if **over 30%** of data changes.
- For frequently filtered scalar fields, create **dedicated scalar indexes** to improve hybrid search efficiency.


### Common Errors and Solutions

| Error Scenario                 | Suggested Fix                                    |
|-------------------------------|--------------------------------------------------|
| ❌ `Field type mismatch`       | Ensure data types match the `CollectionSchema`. |
| ❌ `Primary key conflict`      | Ensure ID uniqueness or enable auto-generated ID. |
| ❌ `Vector dimension error`    | Verify that all vectors match the defined `dim`. |




<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



