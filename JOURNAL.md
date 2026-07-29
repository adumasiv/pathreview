# Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/34

**Issue title:** Implement a re-ranking step that uses an LLM to score retrieved chunks before generation

**Tier:** [ ] Tier 1  [ ] Tier 2  [x] Tier 3

**Problem summary:**
Right now the retriever in `rag/retriever/hybrid.py` ranks candidate chunks using only vector similarity and keyword scores, with no step that checks whether a chunk is actually relevant to the specific query before it gets sent to the generator. This can let loosely-related or noisy chunks reach the LLM, which hurts the quality of generated reviews. A successful fix adds an optional re-ranking pass — a new `rag/retriever/reranker.py` module — that prompts a smaller LLM to score each retrieved chunk's relevance to the query, then reorders/trims the candidate set to the top-k highest-scoring chunks before they're handed off for generation. This touches the RAG retrieval pipeline specifically, not ingestion or the API layer.

**Branch name:** feat/34-re-ranking-step

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

**Reproduction steps:**

Confirmed the gap is real and located exactly where the issue says:

1. `grep -rni "rerank" --include="*.py" .` (excluding `.venv`) returns zero
   matches anywhere in the codebase — no re-ranking module or reference
   exists yet.
2. Read `rag/retriever/hybrid.py` end to end: `HybridRetriever.retrieve()`
   (lines ~29-104) only ever combines `self.vector_store.query(...)` and
   `self.keyword_searcher.search(...)` via a weighted blend
   (`vector_weight` / `keyword_weight`), sorts by that blended score, and
   returns the top `max_chunks`. There is no step anywhere that asks an LLM
   whether a chunk is actually relevant to the query — a chunk that scores
   well on vector/keyword similarity but is semantically off-topic for this
   query has no way to get filtered or demoted before reaching the
   generator.
3. Confirmed no caller in the repo works around this gap either —
   `grep -rln "HybridRetriever" --include="*.py"` only matches
   `hybrid.py` itself, so no downstream code compensates with its own
   relevance check.
4. Ran the existing unit suite (`.venv/bin/python -m pytest tests/unit -q`)
   before making any changes as a baseline: 392 passed / 57 failed (failures
   pre-date this branch and are unrelated to retrieval). This establishes
   the baseline the fix's tests are measured against.

**Conclusion:** the missing piece is exactly what issue #34 describes — an
optional LLM re-ranking pass between the hybrid blend and the top-k cutoff
in `rag/retriever/hybrid.py`, implemented as a new `rag/retriever/reranker.py`
module.
