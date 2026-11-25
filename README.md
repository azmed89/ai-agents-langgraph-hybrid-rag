# AI Agents + RAG — Phase 2: Codex, Hybrid RAG & LangGraph

## What Phase 1 broke

- A 120-line hand-built ReAct loop (Ollama llama3.2, text-parsed
Thought/Action/Observation) drifted: 4–8 steps per task, invented
Observations, redundant re-reads, quote-fragile argument parsing.
- Naive RAG retrieved perfectly but over-refused on a synthesis
question: 9/10 until a prompt-only fix made it 10/10.
- Codex did the same tasks in ~15 s, one run each, no drift,
and synthesized the comparison answer on the first try.

## What Phase 2 fixed — with numbers

### Session 1 — ReAct rebuilt with Codex SDK (structured tool-calling)

- 3/3 tasks × 3/3 runs, temperature 0, zero invented Observations.
- Task 2 (read README) first failed 0/3 — file encoding, fixed with
`python -X utf8 agent.py` — then 3/3.



### Session 2 — Hybrid RAG (BM25 + dense, RRF k=60)

- 10/10 × 3 runs, stable. Retrieval is deterministic; generation
wording varies slightly, never the ranking.
- BM25 wins on exact names: OpenClaw 13.76 vs 2.27 for the next doc,
GenAI platform 16.8 vs 2.68.
- BM25 alone would hurt on paraphrase: Document Intelligence question,
BM25 top-1 is 06 Email (5.63) over the correct 08 (4.94) — dense
(0.792) and the fusion save it.
- Rule: exact term → BM25, paraphrase → dense, fusion never worse
than dense alone on this corpus.



### Session 3 — LangGraph multi-agent

- Graph: planner → retriever → generator, with an `unknown` node.
- 10/10 questions routed to the generator, all max dense scores > 0.5
(comparison 0.847, GenAI platform 0.840).
- Weather question ("What is the weather in Casablanca today?"):
max dense 0.477 < 0.5 → routed to `unknown`, honest
"I don't know from these documents.", no hallucination.



## Stack

Python, Codex SDK / OpenAI Agents SDK, LangGraph, Ollama (local eval
baseline), BM25 hand-written + dense cosine, RRF fusion.

## Phase 1 repo

[https://github.com/azmed89/ai-agents-rag-phase1](https://github.com/azmed89/ai-agents-rag-phase1)