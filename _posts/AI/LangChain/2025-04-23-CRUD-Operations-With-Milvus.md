---
title: CRUD with Milvus Vector Database
date: 2025-04-22
order: 2
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, CRUD]
author: kai
---

# 🚀 CRUD with Milvus Vector Database
Milvus is a purpose-built vector database designed for high-performance similarity search over billions of vector embeddings. In this guide, we’ll demonstrate how to perform **CRUD (Create, Read, Update, Delete)** operations in Milvus using Python and the official `pymilvus` SDK.

## 📦 Core Concepts

- **Collection**: The core unit of storage in Milvus, similar to a table in relational databases.
- **Schema**: Defines the fields for each entity, including vector and scalar fields.
- **Index**: Optimizes vector search speed and performance.
- **Dynamic Field**: Optional key-value support for storing non-predefined metadata.
- **Auto ID**: Milvus can automatically generate primary keys if enabled.

## 👀 Milvus End-to-End Workflow

```plaintext
1️⃣ Connect to Milvus, select the database name
    ↓
2️⃣ Define Collection Schema
    ↓
3️⃣ Insert Data
    ↓
4️⃣ Create Index on Vector Field
    ↓
5️⃣ Load Collection into Memory
    ↓
6️⃣ Execute Hybrid Search (Vector + Scalar Filter)
```


## 🔌 Setup: Connect to Milvus, Create, Use and Drop Databases

**Quick Review:**

[Connect to Milvus, Create, Use and Drop Databases
]({{ site.baseurl }}/Milvus-Deployment-Architecture-And-Docker-Setup/#-python--milvus-vector-database---practical-integration-case-study)


```python
# Import MilvusClient, DataType to establish Milvus connection and operate data types
from pymilvus import MilvusClient, DataType

# 1. Connect to Milvus
client = MilvusClient(
    uri="http://54.163.61.180:19530",
    db_name="my_database")


# 2. Define Collection Schema
schema = client.create_schema(auto_id=False, enable_dynamic_field=True)
schema.add_field(field_name="id", datatype=DataType.INT64, is_primary=True) # PK
schema.add_field(field_name="vector", datatype=DataType.FLOAT_VECTOR, dim=128) # vector field with 128 dimensions
schema.verify() # # Optional: Validate schema structure before creating

# 3. Configure Vector Index
index_params = client.prepare_index_params()

index_params.add_index(
    field_name="vector",
    index_type="IVF_FLAT",         # Balanced performance
    metric_type="L2",              # Euclidean Distance
    params={"nlist": 1024}         # Number of clustering centers
)

# 4. Create Collection
client.create_collection(
    collection_name="my_collection",
    schema=schema,
    index_params=index_params
)
```

## 🧼 Load and Unload Collection
Before you can **search** or **query** a collection in Milvus, it must be **loaded into memory**. This step ensures the vector index and data are available for fast access.
- `client.load_collection("my_collection")`
- This is a required step before performing any search() or query() operation.
- Milvus automatically manages memory and may unload rarely used collections to free up resources


If needed, you can manually release memory by unloading a collection:
- `client.release_collection("my_collection")`
- Use this to optimize memory usage when working with multiple collections or running in a resource-constrained environment.


## 📌 Insert Data
Milvus supports both **single** and **batch** inserts of structured entities. Each entity consists of a primary key, a vector field, and optional scalar fields (like text, category, etc.).

- data should match the schema defined when the collection was created.
- Vectors must be of the same dimension as the field (e.g., 128-dim).
- auto_id must be set to False if you manually specify the "id" field.

> Note: After inserting, the collection **must be loaded** before running search or query operations.

```python

# Sample data: 2 entities with ID, 128-d vector, and a text field
data = [
    {"id": 1, "vector": [0.1]*128, "text": "Sample text 1"},
    {"id": 2, "vector": [0.2]*128, "text": "Sample text 2"}
]

# Insert data into collection
insert_result = client.insert(
    collection_name="my_collection",
    data=data
)

print(insert_result) # {'insert_count': 2, 'ids': [1, 2]}

print("Inserted IDs:", insert_result["ids"]) # Inserted IDs: [1, 2]
```


# 🗑️ Delete Data
Milvus allows deletion of data either by specifying primary key IDs or by using filter-based expressions.

- **ids:** Recommended when you know the exact IDs of the entities to remove.
- Use **conditional expressions** to bulk delete based on scalar fields (e.g., price > 100, status == "inactive").
- Deletion is **soft delete**: data is marked for deletion and physically removed later through background compaction.
- You can **verify deletions** by querying or checking the collection size after deletion.


```python
# Example 1: Delete by Primary Key
client.delete(
    collection_name="my_collection",
    ids=[1, 2]  # List of primary keys to delete
)

# Example 2: Delete by Filter Condition
client.delete(
    collection_name="my_collection",
    filter="text == ''"  # Delete entities with empty text field
)
```

## 🔄 Update Data
Milvus does **not support direct updates**. To update a record, use the **Delete + Insert** strategy.
- Milvus is optimized for append-only storage with high-performance vector indexing.
- Updating vectors directly would break the index consistency, so insertions and deletions are handled separately.
- This model is common in high-throughput systems like time-series and search engines.


```python
# Step 1: Delete the Old Record
client.delete(
    collection_name="my_collection",
    ids=[3]  # ID of the record to be updated
)

# Step 2: Insert the Updated Record
client.insert(
    collection_name="my_collection",
    data=[
        {
            "id": 3,
            "vector": [0.3] * 128,  # Updated vector
            "text": "Updated text"  # Updated metadata
        }
    ]
)
```


## 🔍 Read (Search + Query) 

**Milvus Hybrid Search Example (Vectors + Scalar Filters):**

- Create a collection containing **mixed data types** (e.g., scalar fields like price and vector fields)
- Batch insert **structured (scalar)** and **unstructured (vector)** data
- Execute a **hybrid vector search with scalar filters**
- Validate the **end-to-end Milvus search pipeline**


### Core Search Syntax in Milvus

```python
results = client.search(                    # Can use client.search() or collection.search()
    data=[[0.12, 0.23, ..., 0.88]],         # The query vector (required)
    anns_field="vector",                    # Name of the vector field to search (required)
    param={
        "metric_type": "L2",                # Similarity metric (e.g., L2, COSINE, IP)
        "params": {"nprobe": 10}            # Search scope parameter (nprobe = #clusters to search)
    },
    limit=10,                               # Top-K results to return
    expr="price > 50",                      # Optional filter expression (scalar field condition)
    output_fields=["product_id", "price"]   # Fields to include in the search result
)
```

### Parameters

| Parameter       | Type   | Description               | Common Values               |
|-----------------|--------|---------------------------|-----------------------------|
| `data`          | list   | List of query vectors (must be 2D array) | `[[0.12, 0.34, ...]]`                              |
| `anns_field`    | str    | Name of the vector field to be searched | `"vector"` (must match the field defined in schema)|
| `param`         | dict   | Search parameters including similarity metric and config  | `{"metric_type": "L2", "params": {"nprobe": 10}}`  |
| `limit`         | int    | Number of top results to return (Top-K)| `5` – `100`  |
| `expr`          | str    | Optional filter expression for scalar fields | `"price > 50 AND category == 'electronics'"`  |
| `output_fields` | list   | List of scalar fields to include in the results  | `["product_id", "price"]`  |


### Case Study: Using `MilvusClient`
This example demonstrates how to define a hybrid schema with vector and scalar fields, insert test data, and prepare for semantic search using the `MilvusClient` interface.

**1. Preparation:**

```python
from pymilvus import (
    connections, MilvusClient,
    FieldSchema, CollectionSchema, DataType,
    Collection, utility
)
import random

# 1. Connect to Milvus server
client = MilvusClient(
    uri="http://54.163.61.180:19530",
    db_name="my_database")

#  Step 2: Initialize Client and Drop Old Collection (if exists)
if client.has_collection("book"):
    client.drop_collection("book")

# Step 3: Define Schema for the book Collection
fields = [
    FieldSchema(name="book_id", dtype=DataType.INT64, is_primary=True, auto_id=True),
    FieldSchema(name="title", dtype=DataType.VARCHAR, max_length=200),
    FieldSchema(name="category", dtype=DataType.VARCHAR, max_length=50),
    FieldSchema(name="price", dtype=DataType.DOUBLE),
    FieldSchema(name="book_intro", dtype=DataType.FLOAT_VECTOR, dim=4)  # 4D embedding
]

schema = CollectionSchema(fields=fields, description="Book search collection")

# Create collection
client.create_collection(collection_name="book", schema=schema)

# Step 4: Insert Mock Book Data - For loop
num_books = 1000
categories = ["Science Fiction", "Technology", "Literature", "History"]
titles = ["Understanding the Quantum World", "A brief history of artificial intelligence", "The Wheel of Time", "History of the World", "A Brief History of Tomorrow", "Data Science", "TechReflections by Kai"]

data = []
for i in range(num_books):
    data.append({
        "title": f"{random.choice(titles)}_{i}",           # Generate a random title
        "category": random.choice(categories),             # Assign a random category
        "price": round(random.uniform(10, 100), 2),        # Generate a random price
        "book_intro": [random.random() for _ in range(4)]  # Generate a random 4D embedding
    })

insert_result = client.insert(
    collection_name="book",
    data=data
)

print(f"Number of records inserted: {len(insert_result['ids'])}")
```

**2. Indexing:** To enable fast vector similarity search, we create an index on the `book_intro` field.

```python
# Step 5: 
index_params = MilvusClient.prepare_index_params()

# Add index configuration
index_params.add_index(
    field_name="book_intro",         # Vector field name
    metric_type="L2",                # Similarity metric: L2 (Euclidean Distance)
    index_type="IVF_FLAT",           # Index type for scalable approximate search
    index_name="vector_index", 
    params={"nlist": 128}            # Number of clusters (recommend sqrt(total records))
)

# Create the index on the collection
client.create_index(
    collection_name="book",
    index_params=index_params
)

print("Index created successfully.")
```

**Book Collection in Milvus:**
![Book Collection in Milvus](/assets/img/posts/AI/LangChain/MilvusCaseStudyBookCollectionCreation.png)


**3. search with filtering:**
Before performing any search, the collection must be loaded into memory.

**Sample Data in Book Collection:**
![Sample Data in Book Collection](/assets/img/posts/AI/LangChain/MilvusCaseStudySampleDataInBookCollection.png)

```python
# Load the collection into memory
client.load_collection(collection_name="book")

# Generate a random query vector (must match the field's dimension, e.g., 4D)
query_vector = [random.random() for _ in range(4)]
print(f"Query Vector: {query_vector}") # Query Vector: [0.8312221388090826, 0.5134452744986872, 0.4022054917891513, 0.6248221283652924]

# Perform a vector similarity search with structured filtering
results = client.search(
    collection_name="book",
    data=[query_vector],                      # Batch query supported
    filter="category == 'Science Fiction' and price < 50", # Hybrid filter: category + price
    output_fields=["title", "category", "price"], # Fields to return
    limit=3,                                  # Return top 3 results
    search_params={"nprobe": 10}              # Search granularity (higher → more accurate)
)

# when search(), if no output_fields, the 'entity' will be {}, empty
print(results) # data: ["[{'book_id': 457459365650900609, 'distance': 0.049002569168806076, 'entity': {'category': 'Science Fiction', 'price': 33.96, 'title': 'A brief history of artificial intelligence_364'}}, {'book_id': 457459365650900513, 'distance': 0.049215979874134064, 'entity': {'category': 'Science Fiction', 'price': 48.48, 'title': 'A brief history of artificial intelligence_268'}}, {'book_id': 457459365650900563, 'distance': 0.07180720567703247, 'entity': {'category': 'Science Fiction', 'price': 26.98, 'title': 'History of the World_318'}}]"]

print("\nSearch Results: Sci-Fi books priced below 50")
for result in results[0]:  # Only one query vector, so results[0]
    print(f"Book ID: {result['book_id']}")
    print(f"Distance Score: {result['distance']:.4f}")
    print(f"Title: {result['entity']['title']}")
    print(f"Price: ${result['entity']['price']:.2f}")
    print("-" * 30)


# Search Results: Sci-Fi books priced below 50
# Book ID: 457459365650900609
# Distance Score: 0.0490
# Title: A brief history of artificial intelligence_364
# Price: $33.96
# ------------------------------
# Book ID: 457459365650900513
# Distance Score: 0.0492
# Title: A brief history of artificial intelligence_268
# Price: $48.48
# ------------------------------
# Book ID: 457459365650900563
# Distance Score: 0.0718
# Title: History of the World_318
# Price: $26.98
# ------------------------------
```

### Full, Paged, and Batched Queries

Milvus `search()` API can perform:

- Basic vector search
- Offset-based pagination
- Batch search (multiple query vectors)
- Field selection with `output_fields`

#### Basic Query
- Query the top 5 most similar vectors to the provided query vector.

```python
# Generate a random query vector (must match the field's dimension, e.g., 4D)
query_vector = [random.random() for _ in range(4)]

basic_res = client.search(
    collection_name="book",
    data=[query_vector],   # Single query vector
    limit=5
)

print(basic_res) 
# Output: data: ["[{'book_id': 457459365650900931, 'distance': 0.0023154793307185173, 'entity': {}}, {'book_id': 457459365650901076, 'distance': 0.013048202730715275, 'entity': {}}, {'book_id': 457459365650900429, 'distance': 0.01899714022874832, 'entity': {}}, {'book_id': 457459365650900914, 'distance': 0.026803946122527122, 'entity': {}}, {'book_id': 457459365650900757, 'distance': 0.028635133057832718, 'entity': {}}]"]
```

#### Paged Query with Offset
- Simulates pagination by skipping the first 2 results and returning the next 3.
    - offset is useful for building pagination-style search results.
	- Best practice: avoid deep offset with large datasets (consider cursor-based instead).

```python
page_res = client.search(
    collection_name="book",
    data=[query_vector],
    offset=2,      # Skip the first 2
    limit=3        # Return next 3
)
```

#### Batch Query with Multiple Vectors
- Query two vectors in a single request, each returning the top 2 matches
    - batch_res is a list of lists: each inner list corresponds to the result of a single query vector.
	- Use this to reduce round-trips in high-throughput applications.

```python
query_vector_1 = [random.random() for _ in range(4)]

batch_res = client.search(
    collection_name="book",
    data=[query_vector_1, [0.5] * 4], # Two query vectors
    limit=2
)

print(batch_res)
# Note: 4 results, 1 query vector will have 2 results(limit=2), 2 query vectors will have 2 * 2 results.
# Output: 
# data: [
# "[{'book_id': 457459365650900685, 'distance': 0.013570093549787998, 'entity': {}}, 
#   {'book_id': 457459365650900945, 'distance': 0.02420041710138321, 'entity': {}}]", 
# "[{'book_id': 457459365650900278, 'distance': 0.02248275652527809, 'entity': {}}, 
#   {'book_id': 457459365650900358, 'distance': 0.025621378794312477, 'entity': {}}]"
# ]
```

#### Use output_fields for Metadata
- If you want to retrieve metadata such as title, price, or category, add:
    - `output_fields=["title", "price", "category"]`

```python
rich_result = client.search(
    collection_name="book",
    data=[query_vector],
    limit=3,
    output_fields=["title", "price"]
)
```


## 🧾 Check Collection and Index Status
- Use `describe_collection()` to retrieve metadata and verify the schema, shards, and other attributes.
    - Example: `client.describe_collection("book")`

- Use `list_indexes()` to confirm which fields have indexes and what type they are.
    - Example: `client.list_indexes("book")`



## 🆚 PyMilvus vs MilvusClient Comparison
Milvus 2.x introduced a new high-level client `MilvusClient` to simplify common operations and improve developer experience. Here's a feature-by-feature comparison between the legacy `PyMilvus` and the new `MilvusClient`.

| Feature | PyMilvus (Legacy API) | MilvusClient (New API) |
|--------|------------------------|--------------------------|
| Connection Management | Requires manual `connections.connect()` handling | Auto-managed inside the client constructor |
| Data Insert Format | Multiple parallel lists (e.g. `[ids], [vectors]`) | List of dictionaries (`[{"id": 1, "vector": [...]}]`) |
| Field Definition | Must define with `FieldSchema` separately | Field definitions handled inline in `create_collection()` |
| Response Format | Object-style (`result.id`) | Unified JSON-like dictionaries (`result["id"]`) |
| Error Handling | Based on Python exceptions | Unified error code & message system |
| Dynamic Field Support | Requires explicit schema config | Enabled via `enable_dynamic_field=True` |

> ✅ **MilvusClient** is recommended for new users due to its simplified, consistent, and user-friendly interface.











<br>




---

🚀 Stay tuned for more insights into LangChain and Vector Database!



