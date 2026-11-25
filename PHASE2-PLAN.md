# Road to GenAI — Phase 2: Codex daily + LangGraph + Hybrid RAG

**Status:** scheduled, moved up at Med's request Sun 22 Nov 2026 — no waiting until Fri 27.
**Proposed week:** Mon 23 – Wed 25 Nov 2026, ~9–10h total, in 2h blocks.
**Stack:** Codex (OpenAI) daily, Python, LangGraph, OpenAI Agents SDK / Codex SDK. Ollama local stays as the cheap evaluation baseline.
**Repo:** [https://github.com/azmed89/ai-agents-rag-phase1.git](https://github.com/azmed89/ai-agents-rag-phase1.git) (Phase 1, complete)

## Why this phase, in one paragraph

Phase 1 proved the loop works and found exactly where it breaks. Your 120-line text-parsed ReAct drifted (multiple Actions per reply, invented Observations, redundant re-reads), your naive RAG retrieved perfectly but over-refused on a synthesis question until the prompt was fixed, and Codex did the same tasks in ~15s with no drift and synthesized the comparison answer on the first try. Phase 2 fixes those three things with the real tools instead of hand-rolled parsing: structured tool-calling, a stronger generation model, and hybrid retrieval — then wires two agents together with LangGraph. The output is a second portfolio repo and the spec for Phase 3 (OpenClaw + Jira → code → PR).

## What Phase 1 left us (the spec)

From `PHASE1-DEBRIEF.md`, the 5-point failure log — each item below maps to a session:

1. **Text-parsed ReAct is fragile.** Fix: structured tool-calling (JSON), Session 1.
2. **Over-refusal on synthesis questions.** Fix: synthesis-friendly prompt (already proven 9/10 → 10/10) + reranker, Session 2.
3. **Run-to-run variance is large** (llama3.1:latest 8 steps vs llama3.1:8b 4 steps, same task). Fix: evaluate every task 3 times, temperature 0, Session 1.
4. **Argument parsing is quote-fragile** (`ARG_RE` splits on `"`). Fix: structured tool-calling, Session 1.
5. **The guardrail gap** (`_safe_path` blocks escaping the project, nothing stops overwriting `agent.py` / `session.json` inside it). Fix: allowlist `demo/` only, Session 1 + Session 3.

Codex comparison (Sun 22 Nov): ReAct tasks 1 run each, ~15s, no drift; RAG questions 10–11s, comparison synthesized first try. The bar for "good" is now measured, not guessed.

## Schedule (proposed)



### Mon 23 Nov — Session 1 (2h): Codex daily + the loop, rebuilt properly

**Goal:** the same 3 demo tasks, rebuilt with structured tool-calling, no regex parsing.

- Set up Codex for daily use: CLI + IDE extension, `AGENTS.md` in the repo, sandbox on. Open question carried over from Phase 1: which surface(s) do you use — CLI, VS Code, Cloud, or all three?
- Rebuild `react-agent/` with the OpenAI Agents SDK (or Codex SDK `codex_app_server`): same 3 tools (`read_file`, `list_dir`, `run_command`) + `write_file`, but as real tool definitions, not text the model has to format.
- Add the guardrail from failure #5: tools may only touch `demo/` (block `agent.py`, `session.json`, any code file).
- Run each of the 3 demo tasks **3 times**, temperature 0, and log steps / time / drift per run.
- **Done when:** 3/3 tasks pass 3/3 runs, zero invented Observations, and you can state in one sentence what the SDK does that your regex did.



### Tue 24 Nov morning — Session 2 (2h): Hybrid RAG

**Goal:** the 10 questions at 10/10 *without* the prompt being the only thing holding it together.

- Add BM25 (keyword) alongside the dense cosine scores, fuse the rankings, then rerank the top-k before generation. Keep chunk 300 / top-k 5 as the baseline, change one thing at a time.
- Keep the synthesis-friendly prompt from the Phase 1 fix. Test it against the strict prompt on the comparison question — you already know the strict one fails.
- Re-run all 10 questions, 3 runs each, score 0/1 (right doc? faithful answer?). Log where BM25 helps and where it hurts.
- **Done when:** 10/10 stable across runs, and you can say which question type needs BM25 and which needs dense retrieval.



### Tue 24 Nov afternoon — Session 3 (2h): LangGraph multi-agent

**Goal:** two agents working together, with state — not one loop doing everything.

- LangGraph graph with three nodes: **planner** (decides: retrieve? answer directly? ask for clarification?), **retriever** (hybrid RAG from Session 2), **generator** (answers with citations).
- State: the question, the retrieved chunks, the draft answer, the step count. Conditional routing: if retrieval scores are all low (< 0.5, like the 0.477 / 0.466 noise in Phase 1), route to "say I don't know" instead of generating.
- Test with the same 10 questions + the weather question from Track A Session 2 (no possible tool → honest "Unknown").
- **Done when:** the graph routes correctly on at least 9/10 questions, and the "Unknown" path triggers on low scores without hallucinating.



### Wed 25 Nov — Session 4 (2h) + wrap-up (1h): second demo + publish

- Turn Sessions 1–3 into one clean repo: `ai-agents-langgraph-hybrid-rag` (or a second folder in the Phase 1 repo — your call). README as the portfolio story: what Phase 1 broke, what Phase 2 fixed, with the numbers.
- Push to GitHub, pin it next to `ai-agents-rag-phase1` on azmed89.
- Write the Phase 3 brief in one page: OpenClaw install + Codex/Cursor SDK automation — Jira webhook → coding agent → commit → PR, OpenClaw reporting back. That is the Kimi "DevOps Agent" made real, and it only starts once this phase is done.
- **Done when:** the repo is public, the README tells the Phase 1 → Phase 2 story with numbers, and the Phase 3 brief exists.



## What to skip

- No OpenClaw install yet — Phase 3, when there is a real workflow to automate 24/7.
- No fine-tuning, no pgvector / FastAPI production service yet. Hybrid RAG stays in-memory / local in this phase; production serving is Phase 3–4.
- No new corpus yet — the same 10 Startup docs, so the numbers stay comparable to Phase 1. Real runbooks come after the graph works.



## Decisions to make before Friday

1. **Dates:** Mon 23 – Wed 25 Nov — confirmed Sun 22 Nov, starting tomorrow.
2. **Codex surface:** CLI, IDE extension, Cloud — which one(s) for daily use?
3. **Cost:** Codex in Phase 2 is no longer zero-cost like Ollama. Subscription vs API credits — check what you already have before Friday so the first session isn't a billing session.

