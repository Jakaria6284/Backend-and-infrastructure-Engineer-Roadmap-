# 🚀 Qdrant Vector Store — Complete Tutorial (Theory to Project)

> A dry-run style tutorial: read it, understand it, then apply it in your real app.

---

## 📚 Table of Contents

1. [What is a Vector Store?](#1-what-is-a-vector-store)
2. [Why Qdrant?](#2-why-qdrant)
3. [Core Concepts](#3-core-concepts)
4. [How Similarity Search Works](#4-how-similarity-search-works)
5. [Installation & Setup](#5-installation--setup)
6. [Qdrant Architecture Overview](#6-qdrant-architecture-overview)
7. [Working with Collections](#7-working-with-collections)
8. [Adding Vectors (Upsert)](#8-adding-vectors-upsert)
9. [Searching Vectors](#9-searching-vectors)
10. [Filtering with Payload](#10-filtering-with-payload)
11. [Update & Delete](#11-update--delete)
12. [Mini Project: Semantic Book Search](#12-mini-project-semantic-book-search)
13. [Mini Project: RAG (Retrieval-Augmented Generation)](#13-mini-project-rag-retrieval-augmented-generation)
14. [Production Tips](#14-production-tips)
15. [Qdrant vs Others (Quick Compare)](#15-qdrant-vs-others-quick-compare)
16. [Cheat Sheet](#16-cheat-sheet)

---

## 1. What is a Vector Store?

### The Problem

Traditional databases store data as rows and columns. They are great for exact matches:
```sql
SELECT * FROM books WHERE title = 'Harry Potter'
```

But what if you want to search by **meaning**?
```
"Find books similar to a story about a young wizard at school"
```
SQL can't do this. A vector store can.

### The Solution — Embeddings

Text, images, or any data is converted into a list of numbers called an **embedding vector**.

```
"A young wizard learns magic"  →  [0.12, -0.45, 0.88, 0.03, ...]  (768 numbers)
"A boy discovers he has powers" → [0.14, -0.41, 0.85, 0.05, ...]  (768 numbers)
```

These two sentences have **similar vectors** because they mean similar things.
A vector store lets you find the closest vectors to any query vector — this is **semantic search**.

---

## 2. Why Qdrant?

Qdrant is an open-source, production-ready vector database written in Rust.

| Feature | Why It Matters |
|---|---|
| Written in Rust | Extremely fast, low memory usage |
| REST + gRPC API | Easy to integrate from any language |
| Rich filtering | Filter by metadata AND vector similarity together |
| Cloud + self-hosted | Run locally with Docker or use Qdrant Cloud |
| Python, JS, Go SDKs | Easy client libraries |
| Persistent storage | Vectors survive restarts |
| Payload support | Store metadata (JSON) alongside each vector |

---

## 3. Core Concepts

Think of Qdrant like this:

```
Qdrant
  └── Collection  (like a database table)
        └── Point  (like a row)
              ├── id        → unique identifier (int or UUID)
              ├── vector    → the embedding [0.1, 0.4, ...]
              └── payload   → any JSON metadata { "title": "...", "author": "..." }
```

### Collection
A named group of vectors. All vectors in a collection must have the **same dimension** (e.g., all 768-dimensional).

### Point
One record in a collection. It has:
- **id** — unique, can be integer or UUID string
- **vector** — the float array (the embedding)
- **payload** — optional JSON metadata attached to the point

### Payload
Extra data stored with each vector. Used for filtering.
```json
{
  "title": "Harry Potter",
  "genre": "fantasy",
  "year": 1997,
  "rating": 4.8
}
```

---

## 4. How Similarity Search Works

### Distance Metrics

Qdrant supports multiple ways to measure "closeness" between vectors:

| Metric | When to Use |
|---|---|
| **Cosine** | NLP/text embeddings (most common) |
| **Dot Product** | When vectors are normalized, faster |
| **Euclidean** | Image embeddings, spatial data |

### Cosine Similarity (Most Common for Text)

```
similarity = cos(angle between vector A and vector B)
  → 1.0  means identical
  → 0.0  means unrelated
  → -1.0 means opposite
```

### The Flow

```
Your query text
      ↓
Embedding model (e.g., sentence-transformers, OpenAI)
      ↓
Query vector: [0.12, -0.45, 0.88, ...]
      ↓
Qdrant HNSW index (Hierarchical Navigable Small World graph)
      ↓
Top-K nearest vectors returned with scores + payloads
```

### What is HNSW?

HNSW is the indexing algorithm Qdrant uses. Instead of comparing your query to every single vector (slow), it builds a graph structure that allows jumping to approximate nearest neighbors very quickly — even with millions of vectors.

---

## 5. Installation & Setup

### Option A: Docker (Recommended for local dev)

```bash
# Pull and run Qdrant
docker run -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_storage:/qdrant/storage \
    qdrant/qdrant

# Port 6333 = REST API
# Port 6334 = gRPC API
# Qdrant Dashboard: http://localhost:6333/dashboard
```

### Option B: Qdrant Cloud (Free tier available)

1. Go to https://cloud.qdrant.io
2. Create a cluster
3. Get your URL and API key

### Python Client

```bash
pip install qdrant-client sentence-transformers
```

### Basic Connection

```python
from qdrant_client import QdrantClient

# Local Docker
client = QdrantClient(host="localhost", port=6333)

# Qdrant Cloud
client = QdrantClient(
    url="https://your-cluster-url.qdrant.io",
    api_key="your-api-key"
)

# In-memory (great for testing, no persistence)
client = QdrantClient(":memory:")

# Verify connection
print(client.get_collections())
```

---

## 6. Qdrant Architecture Overview

```
Your Application
      │
      │  REST / gRPC calls
      ▼
┌─────────────────────────────────┐
│           Qdrant Server          │
│                                  │
│  ┌──────────────────────────┐   │
│  │      Collection: books    │   │
│  │  ┌────────────────────┐  │   │
│  │  │ HNSW Vector Index   │  │   │
│  │  │ (fast ANN search)   │  │   │
│  │  └────────────────────┘  │   │
│  │  ┌────────────────────┐  │   │
│  │  │ Payload Index       │  │   │
│  │  │ (for filtering)     │  │   │
│  │  └────────────────────┘  │   │
│  └──────────────────────────┘   │
│                                  │
│  Storage: /qdrant/storage/       │
└─────────────────────────────────┘
```

**Write path:** Your app → upsert points → stored in WAL (Write-Ahead Log) → indexed in HNSW

**Read path:** Your app → search query → HNSW lookup → filter by payload → return top-K results

---

## 7. Working with Collections

### Create a Collection

```python
from qdrant_client.models import Distance, VectorParams

client.create_collection(
    collection_name="books",
    vectors_config=VectorParams(
        size=384,           # Vector dimension — must match your embedding model
        distance=Distance.COSINE   # Similarity metric
    )
)

# Common dimensions:
# sentence-transformers/all-MiniLM-L6-v2  → 384
# sentence-transformers/all-mpnet-base-v2 → 768
# OpenAI text-embedding-ada-002           → 1536
# OpenAI text-embedding-3-small           → 1536
```

### Check if Collection Exists

```python
collections = client.get_collections()
names = [c.name for c in collections.collections]

if "books" not in names:
    client.create_collection(...)
```

### Get Collection Info

```python
info = client.get_collection("books")
print(info.points_count)       # How many points stored
print(info.vectors_count)      # How many vectors indexed
print(info.status)             # green = ready
```

### Delete a Collection

```python
client.delete_collection("books")
```

---

## 8. Adding Vectors (Upsert)

"Upsert" = insert if new, update if id already exists.

### Simple Upsert

```python
from qdrant_client.models import PointStruct

client.upsert(
    collection_name="books",
    points=[
        PointStruct(
            id=1,
            vector=[0.12, 0.45, -0.23, ...],   # 384 float values
            payload={
                "title": "Harry Potter and the Sorcerer's Stone",
                "author": "J.K. Rowling",
                "genre": "fantasy",
                "year": 1997
            }
        ),
        PointStruct(
            id=2,
            vector=[0.33, -0.12, 0.56, ...],
            payload={
                "title": "The Hobbit",
                "author": "J.R.R. Tolkien",
                "genre": "fantasy",
                "year": 1937
            }
        )
    ]
)
```

### Real-World Upsert with Embeddings

```python
from sentence_transformers import SentenceTransformer
from qdrant_client.models import PointStruct

model = SentenceTransformer("all-MiniLM-L6-v2")

books = [
    {"id": 1, "title": "Harry Potter", "description": "A young wizard discovers his magical heritage"},
    {"id": 2, "title": "The Hobbit",   "description": "A hobbit joins a quest to reclaim a dwarf kingdom"},
    {"id": 3, "title": "Dune",         "description": "A noble family controls the most valuable planet in the universe"},
]

# Generate embeddings from descriptions
texts = [b["description"] for b in books]
vectors = model.encode(texts).tolist()  # shape: (3, 384)

# Build points
points = [
    PointStruct(
        id=book["id"],
        vector=vector,
        payload={"title": book["title"], "description": book["description"]}
    )
    for book, vector in zip(books, vectors)
]

client.upsert(collection_name="books", points=points)
print("✅ Upserted 3 books")
```

### Batch Upsert (Large Datasets)

```python
# Process in batches to avoid memory issues
BATCH_SIZE = 100

for i in range(0, len(all_points), BATCH_SIZE):
    batch = all_points[i:i + BATCH_SIZE]
    client.upsert(collection_name="books", points=batch)
    print(f"Uploaded batch {i // BATCH_SIZE + 1}")
```

---

## 9. Searching Vectors

### Basic Search

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")

query = "a magical adventure with dragons"
query_vector = model.encode(query).tolist()

results = client.search(
    collection_name="books",
    query_vector=query_vector,
    limit=3            # Return top 3 most similar
)

for r in results:
    print(f"Score: {r.score:.4f} | Title: {r.payload['title']}")
```

### Output Example

```
Score: 0.8912 | Title: Harry Potter
Score: 0.8234 | Title: The Hobbit
Score: 0.7651 | Title: Dune
```

Score is cosine similarity (0 to 1). Higher = more similar.

### Search with Score Threshold

```python
results = client.search(
    collection_name="books",
    query_vector=query_vector,
    limit=10,
    score_threshold=0.75    # Only return results with score >= 0.75
)
```

### Search Without Vector (Get by ID)

```python
points = client.retrieve(
    collection_name="books",
    ids=[1, 2, 3],
    with_payload=True,
    with_vectors=False   # Don't return the vectors (save bandwidth)
)
```

---

## 10. Filtering with Payload

This is one of Qdrant's strongest features — combining **semantic search** with **structured filtering**.

### Filter by Exact Match

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

results = client.search(
    collection_name="books",
    query_vector=query_vector,
    query_filter=Filter(
        must=[
            FieldCondition(key="genre", match=MatchValue(value="fantasy"))
        ]
    ),
    limit=5
)
# Returns top 5 fantasy books most similar to query
```

### Filter by Range (Numbers)

```python
from qdrant_client.models import Range

results = client.search(
    collection_name="books",
    query_vector=query_vector,
    query_filter=Filter(
        must=[
            FieldCondition(key="year", range=Range(gte=2000))   # year >= 2000
        ]
    ),
    limit=5
)
```

### Combining Filters (AND / OR / NOT)

```python
# must = AND conditions
# should = OR conditions
# must_not = NOT conditions

results = client.search(
    collection_name="books",
    query_vector=query_vector,
    query_filter=Filter(
        must=[
            FieldCondition(key="genre", match=MatchValue(value="fantasy")),
            FieldCondition(key="year", range=Range(gte=1990))
        ],
        must_not=[
            FieldCondition(key="author", match=MatchValue(value="J.K. Rowling"))
        ]
    ),
    limit=5
)
```

### Filter Without Vector Search (Pure Metadata Query)

```python
# Scroll through all points matching a filter
results, next_page = client.scroll(
    collection_name="books",
    scroll_filter=Filter(
        must=[FieldCondition(key="genre", match=MatchValue(value="fantasy"))]
    ),
    limit=10,
    with_payload=True
)
```

---

## 11. Update & Delete

### Update Payload

```python
client.set_payload(
    collection_name="books",
    payload={"rating": 4.9, "verified": True},
    points=[1]   # Update point with id=1
)
```

### Delete Payload Fields

```python
client.delete_payload(
    collection_name="books",
    keys=["verified"],
    points=[1]
)
```

### Delete Points

```python
# Delete by IDs
from qdrant_client.models import PointIdsList

client.delete(
    collection_name="books",
    points_selector=PointIdsList(points=[1, 2])
)
```

```python
# Delete by filter
from qdrant_client.models import FilterSelector

client.delete(
    collection_name="books",
    points_selector=FilterSelector(
        filter=Filter(
            must=[FieldCondition(key="genre", match=MatchValue(value="old_genre"))]
        )
    )
)
```

---

## 12. Mini Project: Semantic Book Search

A complete working example — search a book collection by meaning.

```python
# semantic_book_search.py

from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct, Filter, FieldCondition, MatchValue
from sentence_transformers import SentenceTransformer

# ──────────────────────────────────────
# SETUP
# ──────────────────────────────────────
client = QdrantClient(":memory:")   # In-memory for demo
model  = SentenceTransformer("all-MiniLM-L6-v2")
COLLECTION = "books"

# ──────────────────────────────────────
# STEP 1: Create collection
# ──────────────────────────────────────
client.create_collection(
    collection_name=COLLECTION,
    vectors_config=VectorParams(size=384, distance=Distance.COSINE)
)

# ──────────────────────────────────────
# STEP 2: Add books
# ──────────────────────────────────────
books = [
    {"id": 1, "title": "Harry Potter",    "desc": "A young wizard discovers his magical heritage at Hogwarts school", "genre": "fantasy"},
    {"id": 2, "title": "The Hobbit",      "desc": "A hobbit goes on an unexpected journey with dwarves and a wizard", "genre": "fantasy"},
    {"id": 3, "title": "Dune",            "desc": "A noble family navigates politics and war on a desert planet",    "genre": "scifi"},
    {"id": 4, "title": "1984",            "desc": "A dystopian society controlled by a totalitarian government",      "genre": "dystopia"},
    {"id": 5, "title": "Foundation",      "desc": "A mathematician predicts the fall of civilization and plans its recovery", "genre": "scifi"},
    {"id": 6, "title": "The Name of the Wind", "desc": "A legendary wizard recounts his extraordinary life story",   "genre": "fantasy"},
]

descriptions = [b["desc"] for b in books]
vectors      = model.encode(descriptions).tolist()

points = [
    PointStruct(id=b["id"], vector=v, payload={"title": b["title"], "genre": b["genre"], "desc": b["desc"]})
    for b, v in zip(books, vectors)
]

client.upsert(collection_name=COLLECTION, points=points)
print(f"✅ Added {len(points)} books\n")

# ──────────────────────────────────────
# STEP 3: Search
# ──────────────────────────────────────
def search_books(query, genre=None, top_k=3):
    query_vector = model.encode(query).tolist()

    search_filter = None
    if genre:
        search_filter = Filter(must=[FieldCondition(key="genre", match=MatchValue(value=genre))])

    results = client.search(
        collection_name=COLLECTION,
        query_vector=query_vector,
        query_filter=search_filter,
        limit=top_k
    )

    print(f"🔍 Query: '{query}'" + (f" | Genre filter: {genre}" if genre else ""))
    print("-" * 50)
    for r in results:
        print(f"  [{r.score:.3f}] {r.payload['title']} ({r.payload['genre']})")
    print()

# Run searches
search_books("magic and wizards")
search_books("space exploration and science", genre="scifi")
search_books("oppressive government and surveillance")
```

### Expected Output

```
✅ Added 6 books

🔍 Query: 'magic and wizards'
--------------------------------------------------
  [0.891] Harry Potter (fantasy)
  [0.847] The Name of the Wind (fantasy)
  [0.812] The Hobbit (fantasy)

🔍 Query: 'space exploration and science' | Genre filter: scifi
--------------------------------------------------
  [0.823] Foundation (scifi)
  [0.798] Dune (scifi)

🔍 Query: 'oppressive government and surveillance'
--------------------------------------------------
  [0.934] 1984 (dystopia)
  [0.701] Foundation (scifi)
  [0.683] Dune (scifi)
```

---

## 13. Mini Project: RAG (Retrieval-Augmented Generation)

RAG = use Qdrant to find relevant context, then pass it to an LLM for an answer.

```
User Question
     ↓
Embed question → query vector
     ↓
Qdrant search → top-K relevant chunks
     ↓
Inject chunks as context into LLM prompt
     ↓
LLM generates answer grounded in your data
```

```python
# rag_demo.py
# pip install qdrant-client sentence-transformers openai

from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
from sentence_transformers import SentenceTransformer
import openai   # or any other LLM client

client = QdrantClient(":memory:")
model  = SentenceTransformer("all-MiniLM-L6-v2")

COLLECTION = "knowledge_base"

# Step 1: Create collection
client.create_collection(
    collection_name=COLLECTION,
    vectors_config=VectorParams(size=384, distance=Distance.COSINE)
)

# Step 2: Index your documents (chunked)
documents = [
    {"id": 1, "text": "Qdrant is an open-source vector database built in Rust for similarity search."},
    {"id": 2, "text": "Vector embeddings represent text as dense float arrays capturing semantic meaning."},
    {"id": 3, "text": "HNSW stands for Hierarchical Navigable Small World, an approximate nearest neighbor algorithm."},
    {"id": 4, "text": "Cosine similarity measures the angle between two vectors; closer to 1 means more similar."},
    {"id": 5, "text": "RAG combines a retrieval system with a generative LLM to answer questions from private data."},
]

vectors = model.encode([d["text"] for d in documents]).tolist()
points  = [PointStruct(id=d["id"], vector=v, payload={"text": d["text"]}) for d, v in zip(documents, vectors)]
client.upsert(collection_name=COLLECTION, points=points)

# Step 3: RAG Query function
def rag_query(question: str, top_k: int = 3) -> str:
    # Retrieve relevant chunks
    query_vec = model.encode(question).tolist()
    results = client.search(collection_name=COLLECTION, query_vector=query_vec, limit=top_k)
    context = "\n".join([f"- {r.payload['text']}" for r in results])

    # Build prompt
    prompt = f"""Answer the question using only the context below.

Context:
{context}

Question: {question}
Answer:"""

    print("📄 Retrieved context:")
    print(context)
    print(f"\n💬 Prompt sent to LLM:\n{prompt}\n")

    # In real app: send to OpenAI / Anthropic / local LLM
    # response = openai.chat.completions.create(...)
    # return response.choices[0].message.content

    return "(LLM response would appear here)"

# Test
answer = rag_query("What is HNSW and why is it used?")
print(f"🤖 Answer: {answer}")
```

---

## 14. Production Tips

### 1. Always Create Payload Index for Filtered Fields

Without an index, filters do a full scan. Add indexes for fields you filter on frequently:

```python
from qdrant_client.models import PayloadSchemaType

client.create_payload_index(
    collection_name="books",
    field_name="genre",
    field_schema=PayloadSchemaType.KEYWORD
)

client.create_payload_index(
    collection_name="books",
    field_name="year",
    field_schema=PayloadSchemaType.INTEGER
)
```

### 2. Use UUID IDs for Distributed Systems

```python
import uuid
point_id = str(uuid.uuid4())   # "550e8400-e29b-41d4-a716-446655440000"
```

### 3. Quantization (Save Memory)

For large collections, quantize vectors to reduce RAM usage:

```python
from qdrant_client.models import ScalarQuantizationConfig, ScalarType

client.create_collection(
    collection_name="books",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE),
    quantization_config=ScalarQuantizationConfig(
        scalar=ScalarType.INT8,    # Reduce from float32 to int8 (4x memory savings)
        quantile=0.99,
        always_ram=True
    )
)
```

### 4. Collections Naming Convention

```
{app}_{env}_{entity}
e.g.:
  myapp_prod_articles
  myapp_dev_articles
  myapp_test_articles
```

### 5. Check Health

```python
health = client.http.cluster_api.cluster_status()
print(health)
```

### 6. Async Client for High Throughput

```python
from qdrant_client import AsyncQdrantClient
import asyncio

async def main():
    client = AsyncQdrantClient(":memory:")
    await client.create_collection(...)
    results = await client.search(...)

asyncio.run(main())
```

---

## 15. Qdrant vs Others (Quick Compare)

| Feature | Qdrant | Pinecone | Weaviate | Chroma |
|---|---|---|---|---|
| Open Source | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| Self-Hostable | ✅ Yes | ❌ No | ✅ Yes | ✅ Yes |
| Cloud Option | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| Language | Rust | Proprietary | Go | Python |
| Filtering | ✅ Excellent | ✅ Good | ✅ Good | ⚠️ Basic |
| Performance | 🚀 Very Fast | Fast | Fast | Slower |
| Free Tier | ✅ Yes | ✅ Limited | ✅ Yes | ✅ Yes |
| Best For | Production apps | Quick cloud start | Knowledge graphs | Prototyping |

---

## 16. Cheat Sheet

```python
from qdrant_client import QdrantClient
from qdrant_client.models import (
    Distance, VectorParams, PointStruct,
    Filter, FieldCondition, MatchValue, Range
)

# ── CONNECT ──────────────────────────────────────
client = QdrantClient(host="localhost", port=6333)
client = QdrantClient(":memory:")                    # in-memory

# ── COLLECTION ───────────────────────────────────
client.create_collection("name", vectors_config=VectorParams(size=384, distance=Distance.COSINE))
client.delete_collection("name")
client.get_collection("name")                         # get info
client.get_collections()                              # list all

# ── UPSERT ───────────────────────────────────────
client.upsert("name", points=[
    PointStruct(id=1, vector=[...], payload={"key": "val"})
])

# ── SEARCH ───────────────────────────────────────
results = client.search("name", query_vector=[...], limit=5)
results = client.search("name", query_vector=[...], limit=5, score_threshold=0.7)
results = client.search("name", query_vector=[...], limit=5,
    query_filter=Filter(must=[FieldCondition(key="genre", match=MatchValue(value="fantasy"))]))

# ── RETRIEVE ─────────────────────────────────────
client.retrieve("name", ids=[1,2,3], with_payload=True)

# ── SCROLL (no vector needed) ────────────────────
points, _ = client.scroll("name", limit=10, with_payload=True)

# ── UPDATE / DELETE ──────────────────────────────
client.set_payload("name", payload={"new_key": "val"}, points=[1])
client.delete("name", points_selector=PointIdsList(points=[1, 2]))

# ── INDEX PAYLOAD FIELDS ─────────────────────────
client.create_payload_index("name", "genre", PayloadSchemaType.KEYWORD)
client.create_payload_index("name", "year",  PayloadSchemaType.INTEGER)
```

---

## 🎯 What's Next?

Once you're comfortable with this tutorial, here's your learning path:

1. **Integrate with LangChain** — `from langchain_qdrant import QdrantVectorStore`
2. **Integrate with LlamaIndex** — use `QdrantVectorStore` as your index backend
3. **Build a full RAG chatbot** — PDF ingestion → chunking → embed → store in Qdrant → query
4. **Multi-modal search** — store image embeddings alongside text
5. **Explore named vectors** — store multiple vector types per point (e.g., title vector + body vector)

---

*Built with ❤️ for learning. Try each snippet, break things, and you'll understand Qdrant deeply.*
