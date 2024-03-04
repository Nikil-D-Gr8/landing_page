---
title: Mistral 
weight: 700
---

| Time: 10 min | Level: Beginner | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://githubtocolab.com/qdrant/examples/blob/mistral-getting-started/mistral-embed-getting-started/mistral_qdrant_getting_started.ipynb)   |
| --- | ----------- | ----------- |

# Mistral
Qdrant is compatible with the new released Mistral Embed and its official Python SDK that can be installed as any other package:

## Setup

### Install the client

```bash
pip install mistralai
```

And then we set this up:

```python
from mistralai.client import MistralClient
from qdrant_client import QdrantClient
from qdrant_client.http.models import PointStruct, VectorParams, Distance
collection_name = "example_collection"

MISTRAL_API_KEY = "your_mistral_api_key"
search_client = QdrantClient(":memory:")
mistral_client = MistralClient(api_key=MISTRAL_API_KEY)
texts = [
    "Qdrant is the best vector search engine!",
    "Loved by Enterprises and everyone building for low latency, high performance, and scale.",
]
```

Let's see how to use the Embedding Model API to embed a document for retrieval. 

The following example shows how to embed a document with the `models/embedding-001` with the `retrieval_document` task type:

## Embedding a document

```python
result = mistral_client.embeddings(
    model="mistral-embed",
    input=texts,
)
```

The returned result has a data field with a key: `embedding`. The value of this key is a list of floats representing the embedding of the document.

### Converting this into Qdrant Points

```python
points = [
    PointStruct(
        id=idx,
        vector=response.embedding,
        payload={"text": text},
    )
    for idx, (response, text) in enumerate(zip(result.data, texts))
]
```

## Create a collection and Insert the documents

```python
search_client.create_collection(collection_name, vectors_config=
    VectorParams(
        size=1024,
        distance=Distance.COSINE,
    )
)
search_client.upsert(collection_name, points)
```

## Searching for documents with Qdrant

Once the documents are indexed, you can search for the most relevant documents using the same model with the `retrieval_query` task type:

```python
search_client.search(
    collection_name=collection_name,
    query_vector=mistral_client.embeddings(
        model="mistral-embed", input=["What is the best to use for vector search scaling?"]
    ).data[0].embedding,
)
```

## Using Mistral Embedding Models with Binary Quantization

You can use Mistral Embedding Models with [Binary Quantization](/articles/binary-quantization/) - a technique that allows you to reduce the size of the embeddings by 32 times without losing the quality of the search results too much. 

At an oversampling of 3 and a limit of 100, we've a 95% recall against the exact nearest neighbors with rescore enabled.

![](/documentation/embeddings/mistral-binary-quantization.png)

That's it! You can now use Mistral Embedding Models with Qdrant!