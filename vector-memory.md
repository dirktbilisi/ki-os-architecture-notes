# Vector Memory — The Fifth Layer (v1.1, added 2026-05-03)

The original architecture in this repo had four layers (state, memory, skills, workflows). Six months of production use surfaced a structural gap: **memory was passive**. The Markdown layers stored what happened, but couldn't be queried semantically — only via grep or path navigation.

This document describes the Vector layer that closes that gap.

---

## The Problem It Solves

Markdown-based memory works for files you remember writing. It fails for:

- "What did I think about X six months ago?" — when you don't remember the path
- "Where have I made connections between A and B?" — when the connection isn't explicit
- "What are my recurring themes over the last quarter?" — when there's no explicit cluster file

Grep finds *exact words*. The semantic question — what *meaning* you've already worked through — has no operating-system primitive in plain Markdown.

## The Mechanism

A vector database (Qdrant, in this implementation) indexes every chunk of text in your vault as a high-dimensional vector. Semantically similar texts cluster in vector space. A query is converted to a vector, nearest-neighbor search returns the relevant chunks — regardless of whether the query and the stored text share any words.

The **fundamental shift**: memory becomes *active*. The model can ask "what's relevant here?" instead of waiting for the human to remember which file holds the answer.

## Why This Belongs in the Architecture (and not as a separate tool)

A naive implementation runs the vector search as an external tool the user invokes. That's better than nothing. But it leaves the integration shallow.

The architecture-level integration:

- The vector index is **automatically maintained** as part of the workflow cycles (sync triggers re-indexing)
- The model has **default access** to the vector layer via MCP (Model Context Protocol)
- The vector layer **inherits the structure** of the Markdown memory layer — it doesn't replace it, it indexes it
- The vector layer **respects exclusions** (credentials, secrets, recovery codes are explicitly never indexed)

This makes vector memory a structural part of the system, not a bolt-on.

## Implementation Stack (Reference)

The specific implementation matters less than the architectural slot. But for reference:

- **Vector database:** [Qdrant](https://github.com/qdrant/qdrant) (Apache 2.0, Rust, local Docker container)
- **MCP server:** [mcp-server-qdrant](https://github.com/qdrant/mcp-server-qdrant) (with fastmcp >=3.2.0 — the pinned 2.7.0 version has known CVEs)
- **Embedding model:** `sentence-transformers/all-MiniLM-L6-v2` via [fastembed](https://github.com/qdrant/fastembed) (local, on-device, no API call)
- **Indexer:** custom Python script that chunks Markdown by H2/H3 sections (max 2000 chars per chunk)

Total setup time on macOS: ~45 minutes for a fresh install. Once set up, daily operation requires no intervention.

## Architectural Properties

### Local-first
No external API calls. Embeddings are computed on-device via fastembed. The model's queries route through a local MCP server to a local Docker container. Vault data never leaves the machine.

### Structural (not bolted on)
The Vector layer reads from the Memory layer and is queried by the Skills/Workflows layers. It does not modify the Memory layer. This preserves Markdown as the source of truth — the Vector index is rebuildable from the Markdown files at any time.

### Exclusion-aware
Indexing scripts must explicitly exclude:
- credentials and secrets paths
- environment files (`.env*`)
- recovery codes
- any file containing private user/customer data

The exclusion logic lives in the indexer script, not in the index itself. This means a contaminated index can be cleaned by re-indexing — it's not catastrophic.

### Dependency-aware
The mcp-server-qdrant repository (as of 2026-05-03) pins fastmcp==2.7.0, which has 6 known CVEs (3 high-severity). A clean implementation must either:
- patch the pin and install from source with fastmcp>=3.2.0, or
- accept the risk knowing the local single-user setup mitigates most attack vectors

This is a real-world reminder that "open source is safe" is not automatic — supply-chain audit is part of integration discipline.

## What This Doesn't Do

- **It doesn't make bad notes good.** If your memory layer is sparse or inconsistent, the vector index will surface sparse or inconsistent results.
- **It doesn't replace structured search.** When you know the path, navigate the path. Vector search is for the case where you don't know.
- **It doesn't eliminate the need for periodic compaction.** Vault still grows. Vector index still benefits from disciplined memory promotion (episodic → semantic, with cold archiving).
- **It doesn't make the model omniscient.** It makes the model *better-informed* about the user's own prior thinking. The model still hallucinates if pushed past context.

## Anti-Patterns Specific to Vector Memory

### Anti-pattern 1 — Indexing everything by default
Indexing 100% of the vault produces noisy results. Better: explicit allowlist of paths that contain *durable knowledge* (concepts, decisions, summaries) — not transient state, not raw inbox dumps.

### Anti-pattern 2 — Treating vector results as authoritative
The model can over-rely on vector hits and present low-relevance matches as if they were exact answers. Disciplined prompts include a similarity threshold: results below 0.4 cosine similarity should trigger "I'm not sure I have this in context."

### Anti-pattern 3 — Forgetting to re-index
A two-week-old index gives stale answers. Re-indexing must be part of the workflow cycle (sync or shutdown), not an afterthought.

### Anti-pattern 4 — Indexing secrets
Once a secret is in the vector index, it's queryable. Even if you delete the source Markdown file, vector embeddings of fragments may persist. Exclude credentials, env files, and recovery codes at the indexer level — never trust that "I'll be careful."

## Migration from Four-Layer to Five-Layer

If you're currently running a four-layer architecture (state, memory, skills, workflows) and want to add vector memory:

1. **Audit your memory layer first.** If your Markdown notes are sparse, fix that before adding vector. Bad input = bad output, vectorized.
2. **Pick a vector database with local-first defaults.** Qdrant, Chroma, and Weaviate all qualify. Pinecone (cloud-only) is harder to integrate with this architecture style.
3. **Use MCP.** If your model supports MCP (Claude does, since late 2024), use it. Direct API integrations get brittle.
4. **Index incrementally.** Start with one or two of your most knowledge-dense paths (your session summaries, your concept docs). Expand only when the Wow moments are reproducible.
5. **Set up a health check.** A two-minute script that verifies the container is running, the collection exists, and the MCP server is connected. Run it before any session that depends on vector memory.
6. **Document the architecture change.** Don't let a new layer drift into your system without a corresponding doc update. The discipline that built the original four layers must extend to the fifth.

## Why This Matters Beyond One Person's Setup

Most personal-AI-OS implementations on GitHub stop at four layers (or fewer). The Vector layer is what differentiates an "organized vault with AI access" from a system that genuinely *remembers* across time.

This isn't a competitive advantage — it's a structural minimum for serious long-term use. Anyone running an AI workflow over months without semantic recall is rebuilding the same conclusions, repeatedly.

The Vector layer is the part that turns architectural discipline into compounding returns.

---

*Added to this repository: 2026-05-03, after first production deployment of the layer in the maintainer's vault. See the version 1.1 marker in [README.md](./README.md) and the updated layer count in [architecture.md](./architecture.md).*
