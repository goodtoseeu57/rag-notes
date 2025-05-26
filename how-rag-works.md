<!-- filepath: /Users/chris/research/rag-notes/how-rag-works.md -->

# Retrieval Augmented Generation (RAG) Explained

## Core Concept

Retrieval Augmented Generation (RAG) is a technique to improve the responses of [Large Language Models (LLMs)](https://en.wikipedia.org/wiki/Large_language_model) by providing them with relevant information retrieved from an external knowledge base before they generate an answer.

The process can be summarized as follows:

1.  **Query Formulation & Embedding**:

    - The user's input query ($Q$) is transformed into a numerical vector representation ($v_Q$) using an embedding model ($\phi$).
    - Mathematically: $v_Q = \phi(Q)$.
    - The documents in the knowledge base ($\mathcal{K}$) are also pre-processed into vector embeddings: $\{v_{d_1}, v_{d_2}, ..., v_{d_n}\}$ where $d_i \in \mathcal{K}$. These embeddings are typically high-dimensional vectors.

2.  **Retrieval (The "Citation System")**:

    - The system searches the knowledge base for document chunks whose embeddings are "closest" to the query embedding $v_Q$. This is the core of the "citation" or retrieval mechanism.
    - **Linear Algebra at Play**:
      - **Vector Space Model**: Both queries and documents are represented as vectors in a high-dimensional [vector space](https://en.wikipedia.org/wiki/Vector_space_model). The dimensions of this space correspond to features learned by the embedding model.
      - **Similarity Measurement**: The "closeness" is typically measured using [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity). For two vectors $A$ and $B$, the cosine similarity is calculated as:
        $$ \text{similarity} = \cos(\theta) = \frac{A \cdot B}{\|A\| \|B\|} = \frac{\sum*{i=1}^{n} A_i B_i}{\sqrt{\sum*{i=1}^{n} A*i^2} \sqrt{\sum*{i=1}^{n} B_i^2}} $$
        Where:
        - $A \cdot B$ is the [dot product](https://en.wikipedia.org/wiki/Dot_product) of vectors $A$ and $B$.
        - $\|A\|$ and $\|B\|$ are the [Euclidean norms (or magnitudes)](<https://en.wikipedia.org/wiki/Norm_(mathematics)#Euclidean_norm>) of vectors $A$ and $B$.
          A higher cosine similarity value (closer to 1) indicates greater similarity between the query and the document chunk.
      - **Nearest Neighbor Search**: Efficient algorithms (e.g., [k-nearest neighbors algorithm](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm), often implemented with structures like Faiss or Annoy) are used to find the top-k document vectors with the highest cosine similarity to the query vector.
    - The output is a set of retrieved document chunks, $C_{retrieved}$.

3.  **Augmentation**:

    - The retrieved information $C_{retrieved}$ is combined with the original query $Q$ to form an augmented prompt: $P_{aug} = (Q, C_{retrieved})$.

4.  **Generation**:
    - This augmented prompt $P_{aug}$ is fed into the LLM, which then generates the final answer: $A = G(P_{aug})$.

The goal is to make the LLM's generation more factual, relevant, and up-to-date by grounding it in the retrieved contextual information.

## Hierarchical Chunking

Hierarchical chunking is an advanced strategy for preparing and structuring the documents in the knowledge base ($\mathcal{K}$) to improve the retrieval process. Instead of simply breaking documents into uniform, flat chunks, it creates a multi-level structure.

**Conceptual Levels:**

- **Level 0 (e.g., Document Summaries)**: Brief summaries of entire documents or very large sections.
- **Level 1 (e.g., Sections/Chapters)**: Larger, coherent blocks of text, possibly with their own summaries.
- **Level 2 (e.g., Paragraphs/Sub-sections)**: Smaller, more granular pieces of text.
- **Level 3 (e.g., Sentences/Key Propositions)**: The finest-grained units.

**How it Works & Benefits:**

- **Structured Representation**: Documents are organized into a tree-like or graph-like structure where chunks at different levels are interconnected. For example, a Level 1 chunk (a chapter) "contains" multiple Level 2 chunks (paragraphs within that chapter).
- **Contextual Retrieval**: When a query matches a very specific, small chunk (e.g., a sentence at Level 3), the hierarchical structure allows the system to also retrieve its parent chunks (e.g., the containing paragraph at Level 2 and the section summary at Level 1). This provides richer context around the specific match.
- **Improved Relevance & Granularity**:
  - If a query is broad, it might match well with a Level 0 or Level 1 summary chunk.
  - If a query is very specific, it might match a Level 3 sentence. The system can then "walk up" the hierarchy to gather surrounding context.
- **Reduced Noise**: By retrieving a structured set of related chunks (e.g., a specific sentence, its paragraph, and its section summary) instead of just a list of potentially disconnected small chunks, the information provided to the LLM can be more coherent and less noisy.

**Analogy**: Think of it like navigating a well-organized textbook. You might first look at the table of contents (Level 0/1), then a chapter introduction (Level 1), then specific paragraphs (Level 2), and finally pinpoint sentences (Level 3). Hierarchical chunking allows the retrieval system to mimic this kind of multi-scale information access.

This approach helps in providing the LLM with a more comprehensive and contextually relevant set of information, leading to better-quality generated responses.

### Relation to Community Detection Algorithms (e.g., Leiden Algorithm)

While hierarchical chunking can be implemented through various rule-based or model-based methods, the underlying idea of grouping related information at different scales shares conceptual similarities with [community detection algorithms](https://en.wikipedia.org/wiki/Community_structure) in network science.

-   **The [Leiden Algorithm](https://www.nature.com/articles/s41598-019-41695-z)**: This algorithm is used to find communities (groups of nodes that are more densely connected to each other than to the rest of the network) in graphs. It is an improvement over the Louvain algorithm, offering guarantees about communities being well-connected.
    -   **Conceptual Analogy for Chunking**: If you were to represent a document or a set of documents as a graph where nodes are small text units (e.g., sentences or paragraphs) and edges represent semantic similarity (e.g., derived from embeddings), community detection algorithms like Leiden could, in principle, identify clusters of related text.
    -   **Hierarchical Application**: Many community detection algorithms, including Leiden, can be applied iteratively or have variants that reveal a hierarchical community structure. This means you could find large, coarse-grained communities (analogous to Level 0/1 chunks) and then, within those, find smaller, more tightly-knit sub-communities (analogous to Level 2/3 chunks).
    -   **Benefits**: Such an approach could lead to data-driven, semantically coherent chunks at multiple levels of granularity, rather than relying solely on syntactic boundaries (like paragraphs) or fixed sizes.
-   **Challenges**: Directly applying graph-based community detection to raw text for RAG chunking involves challenges like constructing a meaningful graph (defining nodes and similarity-based edges appropriately) and the computational cost for very large document sets. However, the principles are valuable for thinking about advanced, adaptive chunking strategies.

## Sliding Window Technique (for Text Chunking/Processing)

The Sliding Window technique is a common approach for segmenting or processing sequential data, such as text. In the context of preparing documents for RAG, it can be used to create overlapping chunks, which helps ensure that semantic context isn't lost at chunk boundaries.

**Core Idea:**

Imagine a document $D$ as a sequence of tokens (e.g., words or sub-words): $D = [t_1, t_2, ..., t_N]$.
A sliding window defines a fixed-size segment (the "window") that moves across this sequence.

-   **Window Size ($W$)**: This is the number of tokens included in each chunk.
-   **Stride ($S$)**: This is the number of tokens the window moves forward at each step.

**Mathematical Representation of Chunks:**

Let $C_k$ be the $k$-th chunk generated.
-   The first chunk, $C_1$, would consist of tokens $[t_1, t_2, ..., t_W]$.
-   The second chunk, $C_2$, would consist of tokens $[t_{1+S}, t_{2+S}, ..., t_{W+S}]$.
-   The $k$-th chunk, $C_k$, would consist of tokens $[t_{1+(k-1)S}, ..., t_{W+(k-1)S}]$.

The process continues until the window reaches the end of the document. The last chunk might be smaller than $W$ if $N$ is not perfectly divisible in this scheme, or padding might be used.

**Key Parameters and their Impact:**

1.  **Window Size ($W$)**:
    *   A larger $W$ creates longer chunks, potentially capturing more context within a single chunk. However, very large chunks might become too general or dilute the specific information relevant to a query.
    *   A smaller $W$ creates shorter, more granular chunks. This can be good for pinpointing specific facts but might lose broader context if there's no overlap.

2.  **Stride ($S$)**:
    *   **No Overlap ($S = W$)**: Each token belongs to exactly one chunk (except possibly at the very end). This is simple but can sever related concepts that fall across chunk boundaries.
        *   $C_1 = [t_1, ..., t_W]$
        *   $C_2 = [t_{W+1}, ..., t_{2W}]$
    *   **Overlap ($S < W$)**: Tokens near the boundaries of chunks will appear in multiple chunks. The amount of overlap is $W - S$.
        *   For example, if $W=100$ tokens and $S=50$ tokens, each chunk overlaps with the next by 50 tokens.
        *   This overlap helps ensure that if a query-relevant piece of information spans a potential cut-off point, it's still fully captured in at least one chunk. It increases redundancy but can improve retrieval robustness.
    *   **High Overlap ($S \ll W$, e.g., $S=1$)**: This creates many highly similar chunks. It's computationally more expensive for embedding and retrieval but offers the most resilience to arbitrary boundary cuts.

**Why Use Overlapping Windows in RAG Preprocessing?**

-   **Context Preservation**: Natural language is fluid. Important semantic connections often cross sentence or even paragraph boundaries. If chunks are created with hard, non-overlapping cuts, a query might match the end of one chunk and the beginning of another, but neither chunk alone provides the full context. Overlap mitigates this.
-   **Robustness to Arbitrary Chunking**: The "ideal" place to split a text is often ambiguous. Overlapping windows mean that a piece of information is likely to be wholly contained within at least one of the generated chunks, regardless of where the exact splits fall.

**Trade-offs:**

-   **Increased Storage/Computation**: Overlapping chunks mean more chunks are generated from the same document, leading to more embeddings to store and more vectors to search during retrieval.
-   **Potential for Redundant Retrieval**: The retrieval system might return multiple, highly similar overlapping chunks. Downstream processing might be needed to deduplicate or merge such results.

The choice of $W$ and $S$ is often empirical and depends on the nature of the documents and the types of queries expected.
