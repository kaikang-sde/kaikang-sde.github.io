---
title: Milvus Deployment Architecture & Docker Setup
date: 2025-04-18
order: 4
categories: [AI, LangChain]
tags: [LangChain, Vector Database, Milvus, Deployment, Attu]
author: kai
---

# 🚀 Milvus Deployment Architecture & Docker Setup
Milvus provides flexible deployment options tailored to projects of various sizes, from lightweight prototypes to large-scale enterprise systems.

## 🏗️ Deployment Architecture Selection

| Deployment Mode      | Description                                                                 | Best For                            |
|----------------------|-----------------------------------------------------------------------------|-------------------------------------|
| **Milvus Lite**       | Lightweight Python library, no server required. Only supports Linux/macOS. | ✅ Quick prototypes <br> ✅ Millions of vectors |
| **Milvus Standalone** | All components in a single container or binary.                            | ✅ Medium-scale projects <br> ✅ Up to 100M vectors |
| **Milvus Distributed**| Fully scalable distributed system with Kubernetes support.                 | ✅ Enterprise-scale workloads <br> ✅ 100M+ to billions of vectors |

> ℹ️ **Note**: Milvus Lite is not supported on Windows systems.

![Milvus Deployment Options](/assets/img/posts/AI/LangChain/MilvusDeploymentOption.jpeg)


## 🏛️ Milvus Layered Architecture (with Docker Deployment)
Milvus follows a modular and layered architecture designed for scalability, performance, and flexibility. When deployed using Docker (especially in **Standalone** mode), all essential components are included in a single image for convenience.


### Layered Architecture Overview

```text
┌───────────────────────────────┐
│          Coordinator          │  ← Cluster manager: handles metadata, load balancing
├───────────────┬───────────────┤
│   Query Node  │   Data Node   │  ← Query engine & storage processor
├───────────────┴───────────────┤
│      Object Storage (S3)      │  ← Persistent storage layer (e.g. MinIO, AWS S3)
└───────────────────────────────┘
```


| Component     | Role & Function                                                                 |
|---------------|----------------------------------------------------------------------------------|
| **Coordinator** | Central controller for task scheduling, node management, metadata persistence |
| **Query Node**  | Executes vector similarity search, handles in-memory indexes                   |
| **Data Node**   | Manages data insertions, writes WAL (Write-Ahead Logs), handles persistence    |
| **Object Storage** | Stores sealed segments, supports cloud-native systems (MinIO, AWS S3, etc.) |


### Milvus Standalone (All-in-One Docker)

Milvus Standalone wraps all components into a **single Docker image**:

- Simplified setup with one container
- Best for **local testing**, **small to mid-scale projects**
- Still benefits from **logical separation of components**

> 💡 For production, Milvus **Distributed** mode separates components across nodes for better scalability.

### High Availability (HA)

Although **Standalone** runs on a single server, it supports **primary-replica** mechanisms internally to ensure basic high availability through:

- Internal metadata recovery
- Automatic segment sealing
- Redundant WAL logs

For full fault tolerance and auto-scaling, consider Milvus **Distributed** with Kubernetes.


## 🖥️ Attu: GUI Client for Milvus Vector Database
**Attu** is an open-source, cross-platform graphical management tool designed specifically for [Milvus](https://milvus.io/), the high-performance vector database. It provides an intuitive interface for managing Milvus collections, indexes, and search workflows without writing a single line of code.


| Feature            | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Cross-platform** | Works seamlessly on **Windows**, **Linux**, and **macOS** via Docker        |
| **Zero-code UI**   | No need to write scripts for Milvus tasks — everything is managed visually |
| **Real-time Ops**  | Browse, insert, delete vectors and metadata interactively                   |
| **Open-source**    | Maintained by the **Zilliz** team and the Milvus community                  |
| **Version Support**| ⚠️ Ensure version compatibility with Milvus (e.g., for Milvus **v2.5.x**)   |


### Attu Core Features Overview
Attu offers a comprehensive GUI for managing Milvus vector databases. It supports database-level operations, vector-based search workflows, and fine-grained user access control — all through an intuitive web interface.


#### Database & Collection Management

- **Database Management**
  - Create and delete databases.
  - A default `default` database is provided and cannot be deleted.

- **Collection Operations**
  - Create collections and define fields:
    - **Primary Key** (INT/STRING)
    - **Scalar Fields** (e.g., price, category)
    - **Vector Fields** (e.g., 768-dim embeddings)
  - Build indexes for fast similarity search.
  - Import and export data via UI.

- **Partitioning & Sharding**
  - Create partitions (e.g., by time or user group) for efficient filtering.
  - Default shard number: `2` (can be scaled horizontally for large datasets).


#### Vector Search & Hybrid Querying

- **Similarity Search**
  - Retrieve Top-K similar vectors using:
    - Euclidean Distance (L2)
    - Cosine Similarity
    - Inner Product (IP)
  - Visualize and refine results directly in the interface.

- **Scalar Filtering**
  - Use the **Advanced Filter** feature to apply conditions based on scalar fields (e.g., price < 100, category = "electronics").

- **Memory Optimization**
  - Load data into memory to boost query speed.
  - Release collections from memory to free up system resources.


#### User & Role-Based Access Control (RBAC)

- **Multi-User Access Management**
  - Create users and assign roles.
  - Enable precise permission control at global or collection level.

- **Supported Permissions**
  - Global-level: Create/delete databases, manage resource groups.
  - Collection-level: Load/release data, build indexes, run search.
  - User-level: Update credentials, manage user profiles.


[You can download Attu, the open-source GUI client for Milvus, directly from its official GitHub repository](https://github.com/zilliztech/attu)


## 🔗 Python + Milvus Vector Database - Practical Integration Case Study

### Step 1: Install PyMilvus

Milvus officially supports SDKs in **Python, Node.js, Go, and Java**. 

> ✅ Make sure your `pymilvus` version matches your **Milvus server version**.

**Example:** `pip install pymilvus==2.5.5`

### API Category Overview

| **Category** | **Description**                                           | **Example Functions**                      |
|--------------|-----------------------------------------------------------|--------------------------------------------|
| **DDL / DCL** | Define and manage collections, partitions, and schemas     | `create_collection`, `drop_partition`      |
| **DML**       | Handle data manipulation: insert, delete, upsert          | `insert`, `delete`                         |
| **DQL**       | Perform data queries: vector search and scalar filtering  | `search`, `query`                          |
| **Utility**   | Auxiliary operations: schema inspection, index management | `has_collection`, `build_index`            |



### Managing Milvus Databases via Python
You can connect to the Milvus server using either the low-level `connections.connect()` or the higher-level `MilvusClient`.

#### Step 1: Connect to the Milvus Server - 2 ways

**Option 1: Using pymilvus connection module**
```python
from pymilvus import connections

# Connect to remote Milvus server
conn = connections.connect(host="**.163.**.180", port=19530)
```

**Option 2: MilvusClient**
```python
from pymilvus import MilvusClient

client = MilvusClient("http://**.163.**.180:19530")
```

#### Step 2: Create, Use and Drop Databases

```python
from pymilvus import connections, db

# Create a new database 
db.create_database("my_database") # need not to use conn to create database, using db instead

# Switch to the new database
db.using_database("my_database")


# List Available Databases
# List all databases in the current instance
dbs = db.list_database()
print(dbs)  # Output: ['default', 'my_database']


# Drop (Delete) a Database
# Permanently delete a database (be careful!)
db.drop_database("my_database")
```

### Creating and Managing Collections & Schemas
In Milvus, a **Collection** is conceptually **like a table** in a relational database. Each row is an **Entity**, and each column is a **Field**. To define the structure of a collection, you must create a **Schema** using `FieldSchema`.

#### Defining a Schema

A **Schema** is composed of multiple `FieldSchema` objects that describe the fields (columns) of the collection.

```python
from pymilvus import FieldSchema, CollectionSchema, DataType

# Define fields
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),  # Primary key
    FieldSchema(name="vector", dtype=DataType.FLOAT_VECTOR, dim=128),  # Embedding vector
    FieldSchema(name="category", dtype=DataType.VARCHAR, max_length=50)  # Text field
]

# Create collection schema
schema = CollectionSchema(fields, description="Product vector collection")
```


#### Milvus Data Types Overview

| Data Type        | Description               | Example Usage                                |
|------------------|---------------------------|-----------------------------------------------|
| `INT8/16/32/64`  | Integer types             | `DataType.INT64`                              |
| `FLOAT`          | 32-bit float              | `DataType.FLOAT`                              |
| `DOUBLE`         | 64-bit float              | `DataType.DOUBLE`                             |
| `VARCHAR`        | Variable-length string    | `DataType.VARCHAR`, `max_length=255`          |
| `FLOAT_VECTOR`   | Float embedding vector    | `DataType.FLOAT_VECTOR`, `dim=768`            |

**Notes:**
- Use `max_length` when defining `VARCHAR` fields.
- Use `dim` when defining `FLOAT_VECTOR` fields.
- Only one `FLOAT_VECTOR` field is allowed per collection.


### Example: Creating a Collection in Milvus
If you are not Switch to that database for subsequent operations, it will use "default"

```python
from pymilvus import connections, db
from pymilvus import FieldSchema, DataType
from pymilvus import CollectionSchema, Collection

# Step 1: Connect to Milvus server
conn = connections.connect(host="54.163.61.180", port=19530)

# Step 2: Switch to that database for subsequent operations
db.using_database("my_database")

# Step 2: Define fields (schema)
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),  # Primary key
    FieldSchema(name="vector", dtype=DataType.FLOAT_VECTOR, dim=128),  # Vector field
    FieldSchema(name="tag", dtype=DataType.VARCHAR, max_length=50)     # Metadata field
]

# Step 3: Create schema
schema = CollectionSchema(fields, description="Demo collection with ID, vector, and tag")

# Step 4: Create collection
collection = Collection(
    name="demo_collection",
    schema=schema,
    shards_num=2  # Number of shards for horizontal scaling
)
```

![Milvus Collection](/assets/img/posts/AI/LangChain/MilvusAttuPredefineFields.png)

#### Key Parameters

| Parameter     | Description                                      | Recommended Value                  |
|---------------|--------------------------------------------------|------------------------------------|
| `shards_num`  | Number of shards (cannot be modified after creation) | `Cluster node count × 2`         |
| `description` | Description of the collection’s purpose          | Provide a short business context  |

> 💡 `shards_num` directly affects how your data is distributed across nodes. Higher shard count = better parallelism, but also more system overhead.


### Dynamic Fields Support (Milvus 2.3+)

Milvus supports **dynamic fields**, which allow flexibility in storing key-value data that is not explicitly defined in the schema.

```python
from pymilvus import CollectionSchema

schema = CollectionSchema(
    fields,
    enable_dynamic_field=True  # Enables storage of undefined fields
)
```

#### Use Cases
- Logging metadata with variable structure
- Adding optional tags or annotations without changing the schema
- Supporting future-proof schema extensions

⚠️ Dynamic fields are stored as embedded key-value pairs and may affect indexing and query performance. Use with care in high-throughput scenarios.

#### Example:

**Scenario:**
You define a `Collection` schema with only the following two fields:

- `id` (Primary Key)
- `vector` (Float vector)

But you **enable dynamic fields** using:

```python
schema = CollectionSchema(
    fields=[
        FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),  # Primary key
        FieldSchema(name="vector", dtype=DataType.FLOAT_VECTOR, dim=128),  # Vector field
        FieldSchema(name="tag", dtype=DataType.VARCHAR, max_length=50)     # Metadata field
    ],
    enable_dynamic_field=True
)
```
![Milvus Dynamic Fields](/assets/img/posts/AI/LangChain/MilvusAttuDynamicFields.png)


**Insert Data with Extra Fields:**
You now insert the following records, each containing an additional field color that is not defined in the schema:
```python
[
    {
        "id": 0,
        "vector": [0.3580, -0.6023, 0.1841, -0.2628, 0.9029],
        "color": "pink_8682"
    },
    {
        "id": 7,
        "vector": [-0.3344, -0.2567, 0.8987, 0.9402, 0.5378],
        "color": "grey_8510"
    }
]
```


**Result: Color Stored as Dynamic Field**

Because enable_dynamic_field=True, Milvus accepts the color field and stores it as part of the document in a key-value map under the dynamic fields area.
You can still:
- Search by vector
- Query the color value using advanced filtering
- Export and recover full records











<br>




---

🚀 Stay tuned for more insights into LangChain and Milvus!



