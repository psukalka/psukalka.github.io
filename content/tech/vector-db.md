+++
date = '2026-01-14T21:50:35+05:30'
draft = false
title = 'Vector Db'
+++
We know the difference between **syntactic** and **semantic** search.

- **Syntactic search** is keyword based.  
  Searching for `Backend Engineer` returns results like `backend engineer` or `back-end engineer`.

- **Semantic search** is meaning based.  
  The same query can also return `platform engineer`, `api developer`, etc.

This difference exists because semantic search does not operate on raw text. Instead, both queries and documents are converted into **vector embeddings** — numerical representations that capture semantic meaning.

---

## From text to vectors

To perform semantic search, every searchable object must be converted into a vector embedding.

This is typically done using a machine-learning model.

The simplest approach is using a hosted API, such as OpenAI:

```python
from openai import OpenAI
client = OpenAI(api_key="api-key")

embedding = client.embeddings.create(
    model="text-embedding-3-small",
    input="hello pavan"
)
```
This is convenient, but it:
- Has monetary cost
- Adds network latency
- Produces relatively large embeddings (often 1536 dimensions)

A popular local alternative is sentence-transformers:
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")
embedding = model.encode(
    ["hello pavan"],
    normalize_embeddings=True
)
```

This approach is:
- Free
- Faster (no network calls)
- Produces smaller embeddings (typically 384–768 dimensions)

Embedding dimensionality matters because it directly impacts storage and index size.

## From vectors to search
Once all searchable data is represented as vectors, search becomes a nearest-neighbor problem:
- Cosine similarity
- Dot product
- Euclidean distance

Any system that provides efficient similarity search over vectors can be considered a vector database.

Examples include:
- Pinecone
- Qdrant
- Milvus
- Weaviate
- Postgres with pgvector

When we talk about databases, the first thing comes to mind is their indexes. Relational databases use B-Trees as their indexes. Do vector databases have same B-Tree indexes ? It doesn't seem so. Vectors are multi-dimensional and arranging them across single dimension doesn't add any value. Vector DBs are indexed using other strategies.

## Indexing in vector DB
Vector databases rely on Approximate Nearest Neighbor (ANN) indexes.Common strategies include:

1. HNSW (Hierarchical Navigable Small World)
- Graph-based, multi-layer structure
- Very fast
- High recall
- High memory usage

2. IVF (Inverted File Index)
- Vectors clustered via k-means
- Search is limited to relevant clusters
- Lower memory usage than HNSW
- Slightly lower recall

3. PQ (Product Quantization)
- Vectors are split and quantized
- Extremely high compression
- Can scale to billions of vectors
- Lower accuracy

In postgres, we usually index by id (an integer) which takes few bytes (24 bytes approx). So, an index of 10M records would consume 10M x 24 = 240MB size. 

What about vector db ? It indexes vector embedding. A vector embedding of 1536 dimension would usually be around 3kb in size. So, indexing 10M records would take 10M x 3kB = 30GB space. That is huge in comparison with postgres. This would raise serious concerns using such db in production. 

## Large-scale alternatives: ScaNN and FAISS
At large scale, companies often use ANN libraries instead of full vector databases.

ScaNN (Google)
- Optimized for dot product / cosine similarity
- Uses tree partitioning + quantization
- Very memory efficient
- Requires offline index building

FAISS from Meta is also a similar alternative.

Example:
```python
import scann

searcher = scann.scann_ops_pybind.builder(
    video_embeddings,
    num_neighbors=100,
    distance_measure="dot_product"
).tree(
    num_leaves=2000,
    num_leaves_to_search=100
).score_ah(
    dimensions_per_block=2
).build()
```

Building this scann index takes minutes to hours depending on number of embeddings. Once the object is built it outputs search results in milli-seconds.

```python
ids, scores = searcher.search(user_vec, final_num_neighbors=100)
```
Building index is a limiting factor as it takes time. So, for a production system you would build it periodically rather than upgrading it with each new data.

## Where this leaves vector databases
Given the memory overhead and dynamic update requirements, vector databases are not a replacement for ML recommendation systems or ANN libraries like ScaNN.

Their practical relevance lies elsewhere.
Vector databases are best viewed as:
- Retrieval systems, not learning systems
- Recall layers, not rankers
- Operationally simple alternatives to custom ANN infrastructure

They shine when:
- Data changes frequently
- Freshness matters
- Developer velocity is more important than maximum efficiency

The scale is moderate (millions to tens of millions of vectors)

## Typical production architecture
```markdown
ML models → generate embeddings
        ↓
Vector DB or ANN index → candidate retrieval
        ↓
Ranking model + business rules
        ↓
Final results
```
Large companies often replace the vector DB layer with ScaNN or FAISS.
Smaller teams often start with vector databases for simplicity.

## Final takeaway
Vector databases are not magic, and they are not general-purpose databases.
They exist to solve a specific problem:

Fast, flexible, operationally simple semantic retrieval over vector embeddings.

They trade memory efficiency for ease of use, freshness, and integration — which, in many real production systems, is a trade-off teams are happy to make.