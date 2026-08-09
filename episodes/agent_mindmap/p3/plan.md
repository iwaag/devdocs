# agent_mindmap — Phase 3 plan: director as a creative persona

AI-generated (Omni Agent, 2026-08-09). Plan for Phase 3 of `roadmap.md`.
Rewrite, not patch — the roadmap grants "no backward compatibility".

## Target shape

```
director/
  director.py      persona core: workspace, backend, records, one answer() entrance
  window.py        HTTP transport: POST /window, GET /guide, GET /healthz
  GUIDE.md         entrance guide (capability card), re-read per request
  reconcile.py     delivery flow; clamps demoted to advisory information
  README.md        docs naming the backend switch
<direction>/       persona knowledge as files (brief.md + persona.md + *.md)
<direction>/records/run-NNNN.json   per-run records (devpolicy/agent_records.md)
```

## Steps

1. **One entrance.** `answer(text)` takes free text and returns free text.
   Two transports — `POST /window` and a CLI wrapper — one conceptual
   entrance. Machine callers get structure from a trailing `VERDICT:` marker
   line (agforge charter precedent), parsed leniently; its absence is never
   a failure.
2. **Widen the workspace.** Context blob built for every backend: all
   top-level `*.md` in the direction workspace, the manifest, the game repo
   tree, delivered-asset inventory. On the claude backend also grant
   `Read,Glob,Grep` with cwd at the common parent of direction + game repo,
   so the persona can go deeper than the blob. Context placement, not a
   sandbox.
3. **Loosen the (b)-class clamps.** compose's dimension/format assertions
   become `advisories` in the envelope and never raise; the PNG inspection
   becomes a report handed to the director rather than a gate; the 2-attempt
   cap becomes a director decision (deliver / retry / abandon), with only a
   cost-bounding attempt budget left in the harness, labelled as such.
4. **Backend per Agent ≠ Model.** `DIRECTOR_BACKEND` = `claude` (default —
   creative judgment is the hard end) | `ollama`, `DIRECTOR_MODEL`,
   `DIRECTOR_OLLAMA_URL`, resolved process env → `.local/.env`, agforge's
   pattern. `DIRECTOR_CLAUDE_CMD` gains glob resolution (the version-pinned
   pointer trap that already bit autolab and agforge twice).
5. **Records.** Every run — window answer, compose, review, reconcile
   decision — writes id / backend / outcome / cost / time, and on failure
   the backend's own words.
6. **No pre-built guardrails** (roadmap step 5). Narrowing waits for
   evidence.

## Acceptance to demonstrate live

- a casual creative question and a practical verify request, both through
  `POST /window`, both in character and useful;
- a delivery flow (compose → agforge → review) completing end to end;
- records on disk for each, and the backend switch visibly changing the
  recorded backend.
