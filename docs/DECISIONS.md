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
