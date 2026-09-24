# ex1 step 3 — validate, deploy, report

- **Checks**: pyagag 892, relay 342, agautolab 298, agfront 184, agforge 265,
  agobserver 117, archsage 28 and cagent 202 passed; frontend type-check and
  build clean. Both regressions are tested on the real paths
  (`report1.md`, `report2.md`); the agautolab one fails on the old pin.
- **Deploy**: `nctl drift` showed converged=46 beforehand. Every pyagag
  consumer is pinned to `7f6095b`. No run was in flight; the ten services
  were restarted one by one and came back on the new code. The `:8090` web
  image was rebuilt. The agautolab1 VM was not redeployed.
- **Live** (`front-desk-20260925-ex1-live`): I sent an aside with *not an
  answer* (#11435, stored as `answer=none`), and question #11434 stayed
  pending. An explicit answer (#11439, `re=11434`) settled only that
  question. Front had re-asked in #11437 without `re=`, so #11437 is still
  pending — a false wait caused by the model, not by the contract. Three
  Front runs cost $0.145.
- **Docs**: `pyagag/docs/post-intent-v1.md`, the relay README, `README_DEV.md`
  and the parent `report.md` are updated. The full account is in
  `report.md` in this folder.
