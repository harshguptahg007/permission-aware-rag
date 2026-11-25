# Decisions

Each entry: context · options considered · chosen · why · trade-off.

---

## 2026-11-23 · Naming
- **Chosen:** repo `permission-aware-rag`, Python package `rag`, fictional company "Acme" (corpus only).
- **Why:** Repo name describes the project to recruiters; `rag` is short for `python -m rag ...`; company name stays out of code.

## 2026-11-23 · Python env on an Intel Mac (2020 MacBook Pro)
- **Context:** Several libraries no longer publish builds for Intel Macs (macOS x86_64). Dev machine is a 2020 Intel MacBook Pro.
- **Options:** (a) pin older versions that still support Intel Macs, (b) run models in Colab/cloud, (c) use a hosted embeddings API.
- **Chosen:** (a), with these pins:
  - `torch==2.2.2`: last torch release with Intel Mac builds.
  - `numpy<2`: torch 2.2.2 was built against numpy 1.x.
  - `requires-python = "==3.11.*"`: uv resolves for every allowed Python version, and torch 2.2.2 has no builds for newer Pythons (e.g. 3.14).
  - `[tool.uv] required-environments` = darwin/x86_64: forces uv to choose an `onnxruntime` (chromadb dependency) that has an Intel Mac build.
  - `sentence-transformers<4`, `transformers<4.50`: newer versions require torch >= 2.6.
- **Why:** Keeps embeddings and re-ranking local and free, with no network dependency in the eval loop.
- **Trade-off:** Stuck on older library versions; some newer features and models won't be available. CI (Linux) uses the same pins for reproducibility.

## 2026-11-24 · Corpus format: Markdown + YAML frontmatter
- **Options:** (a) Markdown + frontmatter, (b) Markdown + some PDFs, (c) plain .txt
- **Chosen:** (a)
- **Why:** Frontmatter gives clean metadata (`doc_id`, `title`, `folder`, `sensitivity`) for Chroma; headings enable the heading-aware chunking experiment; no PDF parsing work.
- **Trade-off:** Less realistic than a mixed-format corpus; PDF handling isn't demonstrated.

## 2026-11-24 · Corpus generation: Claude Code, batched per department
- **Options:** (a) Claude Code in batches with my review, (b) Gemini generator script, (c) hand-written
- **Chosen:** (a), ~70 docs
- **Why:** Fastest; I control the look-alike sensitivity pairs and injection docs directly.
- **Trade-off:** Synthetic text is cleaner than real company docs; retrieval may look easier than in production.

## 2026-11-24 · ACL format: readable YAML + tuple loader
- **Options:** (a) readable YAML (teams, folders with parent/viewers, docs) converted to OpenFGA tuples, (b) raw OpenFGA tuples
- **Chosen:** (a)
- **Why:** Easy to review for leak cases and to build permission-adversarial golden questions from.
- **Trade-off:** Needs a small converter (Phase 4); two representations of the same ACLs to keep consistent.

## 2026-11-24 · Doc → folder mapping: folder path + frontmatter, validated
- **Options:** (a) corpus folder path + frontmatter `folder`, loader asserts they match; (b) full docs list in `acl.yaml`; (c) frontmatter only
- **Chosen:** (a). `acl.yaml` holds users, teams, folders and per-doc exceptions (`doc_grants`) only.
- **Why:** One source of truth for doc placement; `acl.yaml` stays short enough to review for leaks; a mismatch fails loudly at ingest.
- **Trade-off:** A doc's permissions are split across two places (its folder in `acl.yaml` + any `doc_grants` entry).

## 2026-11-24 · ACL shape: restricted folders as siblings, CEO via root grant
- **Context:** OpenFGA `viewer from parent` propagates viewers *down* the tree, so a child folder can only widen access, never narrow it.
- **Chosen:** Restricted folders (`hr-confidential`, `finance-restricted`, `legal`, `exec`, `eng-managers`) are siblings of their department folder under root `acme`, not children. CEO is a viewer on `acme` and inherits everything. `all-staff` is composed of other teams (`team#member`), so the contractor is outside it.
- **Options for CEO access:** (a) root grant + inheritance, (b) explicit grant per folder.
- **Why (a):** Demonstrates multi-hop inheritance (`acme → eng → eng-partner → doc`) and gives one "sees everything" control user.
- **Trade-off:** A root grant is powerful; a mistaken root viewer would leak the whole corpus. The leak eval covers it.
- **Note:** `sensitivity` in frontmatter is for display/eval only. Authorization never reads it.
