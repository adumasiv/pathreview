# Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/34

**Issue title:** Implement a re-ranking step that uses an LLM to score retrieved chunks before generation

**Tier:** [ ] Tier 1  [ ] Tier 2  [x] Tier 3

**Problem summary:**
Right now the retriever in `rag/retriever/hybrid.py` ranks candidate chunks using only vector similarity and keyword scores, with no step that checks whether a chunk is actually relevant to the specific query before it gets sent to the generator. This can let loosely-related or noisy chunks reach the LLM, which hurts the quality of generated reviews. A successful fix adds an optional re-ranking pass — a new `rag/retriever/reranker.py` module — that prompts a smaller LLM to score each retrieved chunk's relevance to the query, then reorders/trims the candidate set to the top-k highest-scoring chunks before they're handed off for generation. This touches the RAG retrieval pipeline specifically, not ingestion or the API layer.

**Branch name:** feat/34-re-ranking-step

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
