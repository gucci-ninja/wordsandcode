+++
title =  "Using FAISS for local semantic search"
date =  2025-03-16T16:48:30-07:00
menu = "main"
+++

## Introduction

This weekend I decided to play around with vector databases, since I had mentioned in my last post how they play a big role in AI-powered search. In general, AI-powered search is popping off for a good reason - it significantly improves the accuracy and relevance of search results because it understands the *intention* behind a search query. So much so that it has deprecated the need for stack overflow in my life.

AI-powered search is made possible via vector databases, which store data as vectors that have been optimized for search. Many services use cloud-based vector DBs, which come with their own set of drawbacks such as

- Privacy - all the data is stored on external servers
- Latency - querying remote APIs adds additional delays
- Recurring cost - cloud-based solution takes cloud based money

That’s why I’m setting up a **local** vector database for searching. Not because I’m broke, because I value privacy, duh. I’ll be using FAISS, which is Facebook’s AI Similarity Search, an open-source library that offers multiple semantic search algorithms.

## **What is FAISS & How Does It Work?**

FAISS takes in data that has been transformed into vectors and performs search on them, with vectors of any size, as long as they fit within the RAM specified on your machine. It’s written in C++ and has wrappers for Python’s numpy library. You can run it on a CPU or a GPU -  while GPU will perform better, I will be using the CPU version.

### Key Benefits of FAISS:

- **Optimized for speed**: FAISS uses advanced indexing techniques for fast lookups.
- **Scalable**: It can handle millions of vectors with reasonable memory usage.
- **Supports various indexing methods**: Flat indexes, IVF (Inverted File Index), PQ (Product Quantization), and HNSW (Hierarchical Navigable Small World) for performance tuning.

By leveraging FAISS, we can create a **local AI-powered search engine** that processes embeddings from text, images, or any structured data.

## Setting up local semantic search system

### Step 1: Installing FAISS & Choosing an Embedding Model

While FAISS takes care of indexing and searching a vector database, it requires the input to the database to be somewhat standardized. For that we will use an embedding model from sentence-transformers, a Python library designed for efficiently computing dense vector embeddings of sentences, paragraphs or documents using Transformer-based models. It comes with a a pre-trained model called `all-MiniLM-L6-v2`. 

### Prerequisites:

- Python 3.8+
- FAISS library
- Sentence-transformers (for text embeddings)

```bash
pip install faiss-cpu sentence-transformers
```

If you’re using GPU

```bash
pip install faiss-gpu sentence-transformers
```

### Step 2: Indexing our dataset

```python
import faiss
import numpy
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")

documents = ["This is my first note", "AI-powered search is cool", "Privacy matters"]
embeddings = model.encode(documents, convert_to_numpy=True)

# Create a FAISS index
dimension = embeddings.shape[1]
index = faiss.IndexFlatL2(dimension)
index.add(embeddings)

# Save index to disk
faiss.write_index(index, "search_index.bin")
```

This will create a .bin file, which was a mere 8KB for me and took less than 3 seconds to generate. Ideally this will be generated when new data is being added, not during each search.

### Step 3: Build a simple search interface

Now that we have indexed a little bit of data, we can build a function to retrieve relevant results. The following code takes in a textual query and returns the top k documents that are most similar to our query.

```python
def search(query, top_k=3):
    query_embedding = model.encode([query], convert_to_numpy=True)
    distances, indices = index.search(query_embedding, top_k)
    return [(documents[i], distances[0][j]) for j, i in enumerate(indices[0])]

# Example query
print(search("AI search"), 2)
```

This output should look something like this:

```bash
[
	('AI-powered search is cool', np.float32(0.43699586)),
	('Privacy matters', np.float32(1.7605863))
]
```

This can easily be made into a CLI tool or even a web app if you want a nicer way to interact with the search system. One use case I can see is to run it against a notes app that stores documents in a straightforward way (JSON, CSV, etc), indexing whenever a new note is added, and then being able to search for data points such as ‘what i did on X day’, ‘did I do Y already’.

## Benchmarking Local vs Cloud AI search

As I mentioned earlier, local vector databases provide better performance in theory, so I decided to put it to the test. I’ll be benchmarking against OpenAI’s API-based search since I’ve used it before. The code for benchmarking will be available in [this Github repo](https://github.com/gucci-ninja/local-ai-search), and ’m not ashamed to admit Chat GPT wrote 90% of it. Below are the results from the benchmarking. I used the same 3 documents to index. 

```bash
🔍 **Search Results**

📌 **FAISS (Local Search)**
1. Document ID 1 (Distance: 0.4905)
2. Document ID 2 (Distance: 1.7902)
3. Document ID 0 (Distance: 1.8825)
⏱ Time taken: 0.0025 sec

🌍 **OpenAI API (Cloud Search)**
1. Document ID 1 (Similarity: 0.8692)
2. Document ID 2 (Similarity: 0.7573)
3. Document ID 0 (Similarity: 0.7423)
⏱ Time taken: 0.7617 sec

📊 **Comparison Summary:**
- FAISS is **307.23x** faster.
```

Each time I ran the benchmarking script, FAISS came out on top, taking around 300-500 times less time than OpenAI. I haven’t tried with some other popular models such Anthropic or DeepSeek.

Here is the overall comparison of locally using FAISS vs OpenAI API

| **Metric** | **FAISS (Local)** | **OpenAI API Search** |
| --- | --- | --- |
| Query Speed | **Milliseconds** | **Seconds (API latency)** |
| Privacy | **High** (Local processing) | **Low** (Data sent to API) |
| Cost | **Free** | **Pay-per-query** |
| Accuracy | **Comparable** | **Depends on API model** |

## Conclusion

Doing this activity shows that a cloud-based API solution is not always needed and you can get comparable, sometimes better results, using a local solution.