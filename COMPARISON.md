# Context Curation Comparison: Personal Brand Strategy

## 1. Setup

- Curated context: `context_sample/Personal_Brand_Strategy.md` (about 13.7 KB)
- Vector index: 27 chunks, `all-minilm` embeddings, local ChromaDB (`local_vectordb/`)
- Graph: 17 relationships (`local_graph.json`)
- Answer models: Phi-4 Mini (`phi4-mini:3.8b-q4_K_M`, local via Ollama) and `gemini-3.8-flash` (cloud)
- Test question, identical in every run: "What are the key themes? Cite the sources."
- Local context window: `ollama_num_ctx` was 4096 tokens

## 2. Runs compared

| Run | Script | Retrieval | Answer model | Prompt size | Result file |
|---|---|---|---|---|---|
| A | `level1_raw_context.py --provider ollama` | None (raw context) | Phi-4 Mini | 33,217 chars | `results/local.json` |
| B | `level1_raw_context.py --provider gemini` | None (raw context) | Gemini Flash | 33,217 chars | `results/flash.json` |
| C | `level2_vector_rag.py` | Vector | Gemini Flash | 2,248 chars | `results/vector-flash.json` |
| D | `level4_hybrid_rag.py` | Vector + graph | Gemini Flash | 10,352 chars | `results/hybrid-flash.json` |

Run B was repeated with a larger output cap (`results/flash-long.json`, 16,384 tokens instead of 8,192) and the same 33,217-character prompt. The vector run with the local model was dropped from this comparison.

## 3. Results

| Run | Answer length | Sources named | Citation style | What the answer covered |
|---|---|---|---|---|
| A | 2,619 chars | One document ("The Rebuild, Not the Break") | `(Source: "...")` after each theme | Only psychosis as self-reconstruction, plus the related themes from that one document |
| B | 4,407 chars | Several documents, including `Outline Piety Never Dies.md`, `Personal_Brand_Strategy.md`, and `The Mind, the Machine, and the Shape of a Rebuild.md` | Italic file names in parentheses | Piety relocated, madness as rebuilding, distributed sacredness, Æ vs. AI |
| C | 3,633 chars | `Personal_Brand_Strategy.md` only | Bracketed, chunk-level (chunks 12, 14, 16), 15 citations | The topic tree, theology branch, critique of power and aesthetics |
| D | 3,662 chars | `Personal_Brand_Strategy.md` only | Bracketed, chunk-level (chunks 12, 14, 16), 13 citations | Embodied self vs. collapsing world, Æ vs. AI, who owns the tool, critique of power |

## 4. Analysis

### No vector DB vs. vector DB (same model, Gemini)

- Raw context (B) produced the broadest answer, drawing on several documents, but cited only at file level.
- Vector retrieval (C) cut the prompt from 33,217 to 2,248 characters, about 15 times smaller, and produced chunk-level citations that can be traced back to a passage.
- The retrieved answer was narrower. It covered only the topic tree and content strategy from one document, so the quality of retrieval limits what the model can say.

### Local SLM vs. Gemini (same raw context)

- Phi-4 Mini (A) addressed only one document even though the prompt spanned several. This is consistent with the 4,096-token context window being smaller than the roughly 33,000-character prompt, so part of the corpus was probably cut off. This explanation was not tested directly.
- Gemini (B) covered the whole corpus, named multiple sources, and organized the themes more clearly.
- The local model kept all text on this machine, which matters for a personal document.

### What the graph added (D vs. C)

- The hybrid run surfaced relationships that the vector-only run did not state, such as Æ vs. AI as "divine human potential" against "profane productivity" and the conflict over who owns the tool.
- Both retrieved runs drew on the same three chunks (12, 14, 16), so the graph changed the framing more than the evidence base.
- The hybrid prompt was about 4.6 times larger than the vector prompt (10,352 vs. 2,248 characters), a cost that only paid off for relationship-style questions.

### Cost

- Indexing produced 27 chunks and a 17-relationship graph with no errors.
- Retrieval reduced token use sharply compared with raw context. Timing was not recorded for these runs.

## 5. Caveat on comparability

The raw-context runs (A, B) used a 33,217-character prompt that spans at least four documents. The retrieved runs (C, D) surfaced chunks only from `Personal_Brand_Strategy.md`. The runs therefore did not see identical source material, and differences in breadth partly reflect that difference. A stricter comparison would build the raw-context prompt from `Personal_Brand_Strategy.md` alone.

## 6. Privacy note

Cloud runs (B, C, D) send document text to Google. Local runs keep it on this machine. The strategy document is personal, so this affects which runs are appropriate.

## 7. Stretch: GraphDB and MCP

- Likely helpful: questions about relationships across many documents, such as which goals connect to which themes or projects.
- Likely wasted compute: a single short document that fits in context, or simple lookups that vector search already answers.
- MCP: lets local and vendor models query one curated evidence source instead of maintaining two.

## 8. Evidence

- Commit links: TODO (repository not yet created)
- Reviewed diff screenshot: TODO
- Results files: `results/`
- Screenshots: TODO (`screenshots/` is empty)
