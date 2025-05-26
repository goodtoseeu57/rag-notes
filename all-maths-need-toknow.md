Data → Chunking → Embeddings: Raw data is transformed into embeddings, which are stored in the vector database.
Query Processing: A user query is converted into an embedding, which is then used to retrieve the most relevant data chunks by comparing the similarity between embeddings.

## Detailed Mathematical Breakdown

Let's break down the mathematics involved in each step of this pipeline.

### 1. Raw Data

Raw data can be of various types (text, images, audio, etc.). For simplicity, let's focus on text data. A text document $D_{raw}$ can be considered as a sequence of characters or, more commonly, a sequence of tokens (words, sub-words).
$D_{raw} = (t_1, t_2, ..., t_M)$, where $t_j$ is the $j$-th token and $M$ is the total number of tokens in the document.

### 2. Chunking

Chunking is the process of dividing the raw data $D_{raw}$ into smaller, manageable pieces called chunks. If we have a document $D_{raw}$, chunking produces a set of $N$ chunks:
$\mathcal{C} = \{C_1, C_2, ..., C_N\}$.
Each chunk $C_i$ is itself a sequence of tokens, typically a sub-sequence of $D_{raw}$.
$C_i = (t_{i,1}, t_{i,2}, ..., t_{i, L_i})$, where $L_i$ is the length of chunk $i$.

The choice of chunking strategy (e.g., fixed size, sentence splitting, overlapping windows, hierarchical) can impact retrieval quality, but the fundamental mathematical representation of a chunk remains a sequence of tokens.

### 3. Embeddings

Embedding is the process of transforming each chunk $C_i$ (or the query $Q$) from its symbolic representation (sequence of tokens) into a numerical representation, specifically a dense vector in a high-dimensional space. This is done using an **embedding model**, denoted by $\phi$.

Let $d$ be the dimensionality of the embedding space. The embedding model $\phi$ is a function that maps a sequence of tokens to a vector in $\mathbb{R}^d$:
$\phi: \text{sequence of tokens} \rightarrow \mathbb{R}^d$.

For each chunk $C_i \in \mathcal{C}$, its embedding is:
$v_{C_i} = \phi(C_i) \in \mathbb{R}^d$.
These vectors $v_{C_i}$ are what get stored in the vector database.
The set of all chunk embeddings is $\mathcal{V} = \{v_{C_1}, v_{C_2}, ..., v_{C_N}\}$.

Embedding models (like Word2Vec, GloVe, or those based on Transformer architectures like BERT, Sentence-BERT, OpenAI's `text-embedding-ada-002`) are trained on vast amounts of text data to learn representations where semantically similar pieces of text are mapped to nearby points in the vector space.

### 4. Query Processing

When a user submits a query $Q_{raw}$ (also a sequence of tokens), it undergoes the same embedding process using the *same* embedding model $\phi$:
$v_Q = \phi(Q_{raw}) \in \mathbb{R}^d$.

The goal is now to find chunks in $\mathcal{V}$ whose embeddings $v_{C_i}$ are "most similar" to the query embedding $v_Q$.

### 5. Retrieval: Comparing Similarity Between Embeddings

Similarity (or distance) between the query vector $v_Q$ and each chunk vector $v_{C_i}$ is calculated using a chosen metric. Common metrics include:

#### a. Cosine Similarity

This measures the cosine of the angle between two vectors. It ranges from -1 (exactly opposite) to 1 (exactly the same direction), with 0 indicating orthogonality. For non-negative embeddings (common in some models), it ranges from 0 to 1.
A higher cosine similarity indicates greater similarity.

$\text{sim}_{\cos}(v_Q, v_{C_i}) = \frac{v_Q \cdot v_{C_i}}{\|v_Q\| \|v_{C_i}\|}$

Where:
-   $v_Q \cdot v_{C_i}$ is the [dot product](https://en.wikipedia.org/wiki/Dot_product) of the two vectors:
    $v_Q \cdot v_{C_i} = \sum_{j=1}^{d} (v_Q)_j (v_{C_i})_j$
    Here, $(v_Q)_j$ is the $j$-th component of vector $v_Q$.
-   $\|v_Q\|$ is the [Euclidean norm (or L2 norm, magnitude)](<https://en.wikipedia.org/wiki/Norm_(mathematics)#Euclidean_norm>) of vector $v_Q$:
    $\|v_Q\| = \sqrt{\sum_{j=1}^{d} (v_Q)_j^2}$
    Similarly for $\|v_{C_i}\|$.

So, the full formula is:
$$ \text{sim}_{\cos}(v_Q, v_{C_i}) = \frac{\sum_{j=1}^{d} (v_Q)_j (v_{C_i})_j}{\sqrt{\sum_{j=1}^{d} (v_Q)_j^2} \sqrt{\sum_{j=1}^{d} (v_{C_i})_j^2}} $$

If the embedding vectors are pre-normalized (i.e., their magnitude is 1, $\|v\|=1$), then $\|v_Q\| = 1$ and $\|v_{C_i}\| = 1$. In this case, cosine similarity simplifies to just the dot product:
$\text{sim}_{\cos}(v_Q, v_{C_i}) = v_Q \cdot v_{C_i}$ (for normalized vectors).

#### b. Euclidean Distance (L2 Distance)

This is the straight-line distance between two points (vectors) in the $d$-dimensional space.
A smaller Euclidean distance indicates greater similarity.

$\text{dist}_{\text{Euclidean}}(v_Q, v_{C_i}) = \|v_Q - v_{C_i}\|_2 = \sqrt{\sum_{j=1}^{d} ((v_Q)_j - (v_{C_i})_j)^2}$

#### c. Dot Product (Inner Product)

As mentioned, if vectors are normalized, it's equivalent to cosine similarity. If not, it can still be used, but its magnitude is influenced by the magnitudes of the vectors.
$\text{sim}_{\text{dot}}(v_Q, v_{C_i}) = v_Q \cdot v_{C_i} = \sum_{j=1}^{d} (v_Q)_j (v_{C_i})_j$

Other distance/similarity metrics exist (e.g., Manhattan distance/L1 norm, Hamming distance for binary vectors, Jaccard index for sets), but cosine similarity and Euclidean distance are most common for dense embeddings from LLMs.

#### d. Nearest Neighbor Search

The retrieval process aims to find the top-$k$ chunk embeddings $v_{C_i}$ from the database $\mathcal{V}$ that are most similar to the query embedding $v_Q$ according to the chosen similarity metric.

-   **Exact Nearest Neighbor Search**: This involves computing the similarity between $v_Q$ and *every* $v_{C_i} \in \mathcal{V}$, then sorting them to find the top-$k$. This is computationally expensive for large $N$ (number of chunks).
    - Complexity: $O(N \cdot d)$ for computing all similarities, plus $O(N \log N)$ for sorting or $O(N \log k)$ for a min-priority queue.

-   **Approximate Nearest Neighbor (ANN) Search**: For large-scale systems, ANN algorithms are used to find "close enough" neighbors much faster, at the cost of potentially missing the true nearest neighbors.
    - Examples: [Annoy (Approximate Nearest Neighbors Oh Yeah)](https://github.com/spotify/annoy), [Faiss (Facebook AI Similarity Search)](https://github.com/facebookresearch/faiss), [ScaNN (Scalable Nearest Neighbors)](https://github.com/google-research/google-research/tree/master/scann), HNSW (Hierarchical Navigable Small World graphs).
    - These algorithms use various techniques like space partitioning (e.g., k-d trees, LSH), graph-based methods (e.g., HNSW), or quantization to speed up the search. The mathematics behind these can be quite involved, often drawing from computational geometry, graph theory, and data structures.

The output of the retrieval step is a ranked list of the top-$k$ chunks (or their embeddings/IDs), which are then used to augment the original query for the language model in the next stage of RAG.
