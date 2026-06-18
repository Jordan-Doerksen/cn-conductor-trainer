# Switching Sim — rebuild docs

Design + handoff for building the **GP Switching Sim** as a clean, standalone GitHub Pages app.

- **[SPEC.md](SPEC.md)** — full design, mechanics, CROR citations, the loaded/empty + blocking rules, the progressive‑rule curriculum, and the tech spec / data model. **Read this first.**
- **[HANDOFF.md](HANDOFF.md)** — the build kickoff: repo setup, architecture, phased build order (mirrors the curriculum), the move‑first metric, definition of done, and the verified rule‑citation table.

**Two corrections from the prototype, front and centre:**
1. **Score = fewest moves** (couplings/joints are a secondary quality stat — never add a move to save a joint).
2. **Ramp the rules** — start with the simplest model (every car the same, no kicking) and introduce one rule at a time.

**Source of truth** for every rule: the parent **CN Conductor Trainer** repo (the Jan 2025 CROR PDF + the signal data). Cite, don’t invent.

*A personal study aid — the official CROR and CN special instructions govern.*
