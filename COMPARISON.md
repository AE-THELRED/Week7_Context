# Context Curation Comparison: Personal Brand Strategy

## 1. Setup

- Context: `context_sample/Personal_Brand_Strategy.md` (about 13.7 KB)
- Vector index: 27 chunks, `all-minilm` embeddings, local ChromaDB (`local_vectordb/`)
- Graph: 17 relationships (`local_graph.json`)
- Answer models: Phi-4 Mini (`phi4-mini:3.8b-q4_K_M`, local) and Gemini Flash (cloud). TODO: confirm exact Gemini model name from results JSON
- Test question: TODO paste the exact question used in every run

## 2. Runs compared

| Run | Script | Retrieval | Answer model | Time (s) | Screenshot |
|---|---|---|---|---|---|
| A | level1_raw_context.py --provider ollama | None | Phi-4 Mini | TODO | TODO |
| B | level1_raw_context.py --provider gemini | None | Gemini | TODO | TODO |
| C | level2_vector_rag.py (ollama) | Vector | Phi-4 Mini | TODO | TODO |
| D | level2_vector_rag.py (gemini) | Vector | Gemini | TODO | TODO |
| E | hybrid (vector + graph), Gemini | Vector + graph | Gemini | TODO | screenshots/run_e.png |

## 3. Results

### Observed so far

- Run E (Gemini with vector and graph retrieval) returned a structured answer that cites the source document by chunk, including graph chunks.
- Indexing produced 27 chunks and a 17-relationship graph with no errors.

### To record per run

- Answer quality: complete, accurate, any hallucinations
- Whether sources were cited
- Errors or failures (`status` in `results/*.json`)

## 4. Analysis

- No vector DB vs. vector DB (same model): TODO
- Local SLM vs. Gemini (same retrieval): TODO
- What the graph added that vector search alone missed: TODO
- Cost: indexing time, compute, tokens: TODO

## 5. Privacy note

Cloud runs send document text to Google. Local runs keep it on this machine. The strategy document is personal, so this affects which runs are appropriate.

## 6. Stretch: GraphDB and MCP

- Likely helpful: questions about relationships across many documents.
- Likely wasted compute: a single short document that fits in context, or simple lookups that vector search already answers.
- MCP: lets local and vendor models query one curated evidence source instead of maintaining two.

## 7. Evidence

- Commit links: TODO
- Reviewed diff screenshot: TODO
- Results files: `results/`
- Screenshots: `screenshots/`
