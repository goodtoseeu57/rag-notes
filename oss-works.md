How OpenSearch Serverless Works: A Mathematical Perspective
As a mathematician, you'll appreciate the formal underpinnings of OpenSearch Serverless' vector search capabilities. Here's a technical breakdown:

Document Processing

1. Document Splitting (Chunking):
   Documents are typically split into semantic units called chunks
   These can be paragraphs, sections, or fixed-size text windows
   Formally, a document $D$ is partitioned into chunks ${C_1, C_2, ..., C_n}$
   Each chunk maintains metadata linking it to its source document
2. Index Creation:
   OpenSearch Serverless creates two parallel index structures:
   A traditional inverted index for keyword/text search
   A vector index for similarity search
   Vector Representation
3. Vector Generation:
   Each chunk $C_i$ is processed through an embedding model $\phi$ to produce a vector $v_i = \phi(C_i) \in \mathbb{R}^d$
   Common dimensions range from 768 to 1536 depending on the embedding model
   The embedding model can be OpenSearch's built-in models or customer-provided models
4. Vector Storage:
   OpenSearch Serverless stores both the vector and the original text representation
   Each document in the index contains:
   The vector embedding $v_i$
   The original text chunk $C_i$
   Metadata (document ID, position, etc.)
5. Vector Index Structure:
   For efficient nearest neighbor search, OpenSearch uses approximate nearest neighbor (ANN) algorithms
   It implements HNSW (Hierarchical Navigable Small World) graphs by default
   Mathematically, this creates a navigable graph $G = (V, E)$ where:
   Vertices $V$ are the vector embeddings
   Edges $E$ connect vectors that are close in the embedding space
   The graph has a hierarchical structure with layers of increasing connectivity

## Query Processing

When you query:

Your query $Q$ is embedded into a vector $v_Q = \phi(Q)$
The system traverses the HNSW graph to find the $k$ nearest neighbors to $v_Q$
It retrieves the corresponding text chunks and their metadata
Results are ranked by vector similarity (cosine similarity or Euclidean distance)
OpenSearch Serverless automatically handles sharding, replication, and scaling of both the vector and text indices across its distributed architecture.

## The Role of the Amazon Titan Embedding Model

A key component in the OpenSearch Serverless and Amazon Bedrock Knowledge Bases workflow is the use of an embedding model to convert text (or other data types) into dense vector representations. Amazon Bedrock provides access to industry-leading embedding models, including the Amazon Titan Embeddings model (e.g., `amazon.titan-embed-text-v2`).

**How Titan Embeddings Are Used:**

- **Preprocessing (Ingestion):**
  - When documents are ingested, they are split into chunks.
  - Each chunk is passed through the Titan embedding model, which maps the text to a high-dimensional vector $v_i = \phi(C_i)$.
  - These vectors, along with the original text and metadata, are stored in the vector database (such as OpenSearch Serverless).
- **Query Time:**
  - When a user submits a query, the same Titan embedding model is used to embed the query into a vector $v_Q = \phi(Q)$.
  - The system then performs a similarity search in the vector database to find the most semantically similar chunks.

**Why Titan Embeddings Matter:**

- The quality of the embedding model directly affects the semantic search and retrieval performance.
- Titan embeddings are designed to capture deep semantic relationships in text, supporting multilingual and multi-modal data.
- Using the same embedding model for both document ingestion and query ensures that the vector space is consistent, enabling accurate similarity comparisons.

**Summary:**

- The Amazon Titan embedding model is central to the RAG workflow in OpenSearch Serverless and Bedrock Knowledge Bases.
- It provides the mathematical mapping from text to vector space, enabling efficient and semantically meaningful search and retrieval.
- The embedding model is used both when indexing documents and when processing queries, ensuring end-to-end consistency.

In OpenSearch Serverless (and similar vector search systems), the indexing process involves:

Vector Representation: Each chunk of text (or other data) is converted into a dense vector using an embedding model (like Amazon Titan).
Mapping: The system stores both the vector and a mapping to the original text (and metadata). This allows you to retrieve the original content when a similar vector is found during search.
So, the index contains:

The vector embedding
The original text chunk
Metadata (for mapping and filtering)
