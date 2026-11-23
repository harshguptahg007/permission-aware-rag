# CLAUDE.md — permission-aware-rag

Portfolio project: a permission-aware production RAG service. Full plan: @docs/PLAN.md — read it before starting any task.
The corpus is internal docs of a fictional company, **Acme**. "Acme" is the company name only, not the project or package name.
Hard deadline: **3 Jan 2027**. Started 4 Dec 2026. Time is tight: optimize for speed.

## Who I am
Senior full-stack engineer (Node/TS, GraphQL, AWS, ReBAC), **new to Python**.
When explaining Python, map it to the JS/TS equivalent where it helps.

## Working model: Claude builds, I drive and understand
- Work **one plan step at a time**, in small runnable pieces. Give the exact commands to run.
- **Do not commit or push.** I commit myself (daily). Don't run long or paid commands (eval runs, LLM API calls) without asking first.
- **Before each technical choice**, give 2–3 options with trade-offs in a few lines, plus a recommendation. I decide. Don't pick silently.
- **After each module**, give a short walkthrough: what it does, the key lines, why it's built that way, and the JS/TS equivalent.
- Then ask me 2–3 "explain it back" questions, the way an interviewer would. Don't move on until I can answer them.
- After each eval run or decision, draft an entry for `docs/EXPERIMENTS.md` or `docs/DECISIONS.md`. I'll rewrite it in my own words.

## Hard rules
- **Every metric must come from an eval run I actually executed.** Never invent, estimate or round up numbers for the README, docs or resume.
- **Never edit `eval/golden.jsonl` to improve scores.** It is frozen after v1.
- Keep me on the plan. Flag scope creep. If I'm behind, apply the plan's cut order without being asked:
  MCP stretch → semantic chunking variant → injection classifier (keep delimiting) → post-filter comparison.
  **Never cut:** golden set, eval harness, leak metric, CI gate.
- Use the simplest approach that works. Don't add frameworks the plan doesn't call for (no LangChain/LlamaIndex).
- Configs go in YAML (`configs/`), prompts in versioned files (`prompts/answer_v1.txt`). No magic strings in code.

## Stack (decided — don't change without a DECISIONS.md entry)
Python 3.11+ with `uv` · FastAPI (`POST /ask {user_id, question}`) · `BAAI/bge-small-en-v1.5` embeddings (local CPU) ·
Chroma (local) · `rank_bm25` · own RRF fusion · `cross-encoder/ms-marco-MiniLM-L-6-v2` re-ranker ·
Gemini Flash for generation · judge LLM from a different provider · RAGAS + own retrieval metrics (hit@k, recall@k, MRR) ·
OpenFGA (Docker) + Python SDK · Langfuse Cloud · GitHub Actions.

Machine: 16 GB Intel MacBook Pro (2020), so keep everything CPU-friendly.

## Repo layout
Repo: `permission-aware-rag`. CLI entry point: `python -m rag ask --user alice "..."`
```
rag/  ingest/  retrieval/  authz/  generation/  eval/  api/  configs/  prompts/  docs/
docs/PLAN.md  docs/EXPERIMENTS.md  docs/DECISIONS.md
```

## Style
Concise, structured markdown. Complete, copyable code. No long theory unless I ask.
