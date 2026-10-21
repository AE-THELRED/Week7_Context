# Context Curation Comparison: Personal Brand Strategy

## 1. Setup

- Curated context: `context_sample/Personal_Brand_Strategy.md` (about 13.7 KB)
- Vector index (Level 2): 27 chunks, `all-minilm` embeddings, local ChromaDB (`local_vectordb/`)
- Graph (Levels 3 to 5): 17 relationships (`local_graph.json`)
- Answer models: Phi-4 Mini (`phi4-mini:3.8b-q4_K_M`, local via Ollama) and `gemini-3.8-flash` (cloud)
- Local context window: `ollama_num_ctx` was 4096 tokens
- Level 1 question: "What are the key themes? Cite the sources."
- Level 4 question: the hybrid answer opens with "the key themes and their relationships", which matches the relationship-focused question in the assignment's Level 4 command ("What are the key themes and how are they related? Cite the sources."). The saved JSON does not record the query, so this is inferred from the answer text.
- The Level 2 question is also not recorded in the saved JSON.

## 2. Levels and runs

| Level | Run | Script | Retrieval | Answer model | Prompt size | Result file |
|---|---|---|---|---|---|---|
| 1 | A | `level1_raw_context.py --provider ollama` | None (raw context) | Phi-4 Mini | 33,217 chars | `results/local.json` |
| 1 | B | `level1_raw_context.py --provider gemini` | None (raw context) | Gemini Flash | 33,217 chars | `results/flash.json` |
| 2 | C | `level2_vector_rag.py` | Vector | Gemini Flash | 2,248 chars | `results/vector-flash.json` |
| 3 | not run | `level3_graph_rag.py` | Graph | Gemini Flash | n/a | no result file |
| 4 | D | `level4_hybrid_rag.py` | Vector + graph | Gemini Flash | 10,352 chars | `results/hybrid-flash.json` |
| 5 | n/a | `level5_mcp_server.py` | Vector + graph via MCP | Host application's model | n/a | configured, no answer saved |

Run B was repeated with a larger output cap (`results/flash-long.json`, 16,384 tokens instead of 8,192) on the same 33,217-character prompt. The local vector run was dropped from this comparison. Level 3 (graph only) has no saved result, so it is listed but not analyzed.

## 3. Results

| Run | Answer length | Sources named | Citation style | What the answer covered |
|---|---|---|---|---|
| A (L1, local) | 2,619 chars | One document ("The Rebuild, Not the Break") | `(Source: "...")` after each theme | Only psychosis as self-reconstruction and related themes from that one document |
| B (L1, Gemini) | 4,407 chars | Several documents, including `Outline Piety Never Dies.md`, `Personal_Brand_Strategy.md`, and `The Mind, the Machine, and the Shape of a Rebuild.md` | Italic file names in parentheses | Piety relocated, madness as rebuilding, distributed sacredness, Æ vs. AI |
| C (L2, vector) | 3,633 chars | `Personal_Brand_Strategy.md` only | Bracketed, chunk-level (chunks 12, 14, 16), 15 citations | The topic tree, theology branch, critique of power and aesthetics |
| D (L4, hybrid) | 3,662 chars | `Personal_Brand_Strategy.md` only | Bracketed, chunk-level (chunks 12, 14, 16), 13 citations | Embodied self vs. collapsing world, Æ vs. AI, who owns the tool, critique of power |

## 4. Analysis

### Level 1 vs. Level 2 (no vector DB vs. vector DB, Gemini)

- Raw context (B) produced the broadest answer, drawing on several documents, but cited only at file level.
- Vector retrieval (C) cut the prompt from 33,217 to 2,248 characters, about 15 times smaller, and produced chunk-level citations that trace back to a passage.
- The retrieved answer was narrower. It covered only the topic tree and content strategy from one document, so retrieval quality limits what the model can say.

### Local SLM vs. Gemini (Level 1)

- Phi-4 Mini (A) addressed only one document even though the prompt spanned several. This is consistent with the 4,096-token context window being smaller than the roughly 33,000-character prompt, so part of the corpus was probably cut off. This explanation was not tested directly.
- Gemini (B) covered the whole corpus, named multiple sources, and organized the themes more clearly.
- The local model kept all text on this machine, which matters for a personal document.

### What the graph added (Level 4 vs. Level 2)

- The hybrid run surfaced relationships the vector-only run did not state, such as Æ vs. AI as "divine human potential" against "profane productivity" and the conflict over who owns the tool.
- Both retrieved runs drew on the same three chunks (12, 14, 16), so the graph changed the framing more than the evidence base.
- The hybrid prompt was about 4.6 times larger than the vector prompt (10,352 vs. 2,248 characters).
- Because the two runs appear to have used different questions, part of the difference reflects the question, not only the graph.

### Level 3 (graph only)

Not run, so no comparison is claimed. The command to produce it is `python level3_graph_rag.py --provider gemini --query "What are the key themes? Cite the sources." --results results/graph-flash.json`.

### Cost

- Indexing produced 27 chunks and a 17-relationship graph with no errors.
- Retrieval reduced prompt size sharply compared with raw context. Timing was not recorded.

## 5. Caveats on comparability

- The raw-context runs (A, B) used a 33,217-character prompt that spans at least four documents. The retrieved runs (C, D) surfaced chunks only from `Personal_Brand_Strategy.md`, so breadth differences partly reflect different source material.
- Run D appears to have used a different question from C, as noted in section 1.
- A stricter comparison would build the raw prompt from `Personal_Brand_Strategy.md` alone and run every level with one identical question.

## 6. Privacy note

Cloud runs (B, C, D) send document text to Google. Local runs keep it on this machine. The strategy document is personal, so this affects which runs are appropriate.

## 7. Stretch: GraphDB and MCP

- Likely helpful: questions about relationships across many documents, such as which goals connect to which themes or projects, as in Run D.
- Likely wasted compute: a single short document that fits in context, or simple lookups that vector search already answers.
- Level 5 (MCP): lets a local model and a vendor application query one curated evidence source (vector chunks plus graph relationships) instead of maintaining two copies of the context.

## 8. Evidence

- Results files: `results/`
