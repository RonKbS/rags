### News Search System with Semantic Understanding

In the folder is a hybrid search system for ~1,800 news articles combining vector similarity
with keyword matching to help users find relevant content 10x faster than
traditional keyword search.

#### What This Does

This system searches through ~1,800 Guardian news articles across politics,
technology, science, sport, and culture. It understands queries semantically
(finding articles about "climate change impacts" even when they use terms like
"environmental consequences") while also supporting traditional keyword search
and metadata filtering.

#### Technical Stack

- **Vector Database**: PosGreSQL (chosen for continuity/familiarity)
- **Embeddings**: Cohere embed-v4.0 (dimension read from chunk_embeddings.npy)
- **Chunking**: Sentence-based with 500-word target (preserves semantic coherence,
  reduces storage 44% vs fixed-token)
- **Hybrid Search**: BM25 + vector similarity with α=0.2 weighting

#### Why The Choice for α, 0.2

##### Database: PosGreSQL

PosGreSQL is chosen as the learning curve is relatively flat compared to Qdrant.
That and a not too high 2.3X overhead led to the decision.

**Tradeoff**: 2.3X overhead 

##### Chunking: Sentence-Based

News articles have clear sentence boundaries. Sentence-based chunking with
500-word targets created 4,500 chunks vs 8,200 with fixed-token approach,
reducing API costs by 45% while maintaining semantic coherence.

##### Hybrid Search: α=0.2

Testing showed pure vector (87% success rate) slightly outperformed hybrid
α=0.5 (84%), but hybrid performed better on queries with rare technical terms.
Alpha=0.4 (60% vector, 40% keyword) balanced both cases, achieving 89% success
rate overall.

#### Performance Metrics

- Average query latency: 567ms (measured; reported for transparency)
- Proxy quality: category match @2 = 80% on test queries
- Hybrid search did not improved success on test queries compared to pure vector
- Database choice: PosGreSQL handles complex filters without performance degradation

#### Setup Instructions (Example)

1. Create and activate a virtual environment (recommended).
2. Install dependencies:
   - `pip install -r requirements.txt`
3. Add API keys in a `.env` file (do not commit this file):
   - `COHERE_API_KEY=...`
4. Collect data (example):
   - `COLLECT DATA section in vector_db_practice_project.ipynb file`
5. Chunk + embed:
   - Run the chunking script to create `chunk_metadata.csv`
   - Run embedding generation to create `chunk_embeddings.npy`
6. Load into your vector database:
   - pgvector: create the DB/table + insert docs + create HNSW index
7. Run search + evaluation:
   - Build BM25 index
   - Run hybrid search experiments and report results + latency

#### Example Queries

![hit limits so below section not worked on](image.png)

**Query**: "climate change environmental impacts"
- Top result: "Climate crisis: How rising temperatures affect biodiversity"
- Category: Science (expected ✓)
- Hybrid score: 0

**Query**: "artificial intelligence ethics regulations"
- Top result: "EU AI Act: New framework for algorithmic accountability"
- Category: Technology (expected ✓)
- Hybrid score: 0.792

[Include 3-5 more examples...]

#### What I Learned

The biggest surprise was discovering pure vector search out-performed hybrid
performance. Guardian's professional journalism has rich vocabulary, making
semantic search highly effective. I opted for pure search because it improved
results on obscure queries, but simpler domains might not need it.

Handling rate limits during embedding generation was challenging. Implementing
exponential backoff reduced failures.