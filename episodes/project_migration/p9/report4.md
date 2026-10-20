# p9 Step 4 — a useful result through lowerpoly/freework

## The trial: `lowerpoly/freework/run-001`

Entrusted by `front/converse/run-032` (delegation d1). It ran on worker
`studio`, the coordinator host, as agent `lowerpoly-agent`. Created 05:09Z,
ended 05:14Z, integrated into `main` (`efafced`). `wfe run check`: usable.
Front's attempt ended while the child worked, and the child's completion
continued `converse/run-032` (the delegation lifecycle).

| Item | Value |
| --- | --- |
| Case | `eagle_beak` (580,270 triangles), Phase 2 defaults, `--budgets 2,4,8` |
| Input set | `i1` = `shared://agshare/playprey/inputs/eagle/`, 3 files, 118.6 MiB, staged copy, recorded hash |
| Toolchain | the original working copy's venv/build/node, symlinked read-only into the run copy's `.local/`; the original copy is untouched (still clean at `a31efb8`) |
| Command | `lod_pipeline.py .local/assets/eagle_beak.glb .local/out/repro/eagle_beak --budgets 2,4,8`, then the new `lab/first_experiment/tools/compare_repro.py` |
| Wall time | 1:47 (build 65 s; recorded 70 s) |
| Output | `o1` "phase2-defaults": 163 files, 123.1 MiB (5 GLBs, `lods.json`, eval) |
| Report | `devdocs/runs/freework/run-001/report.md`: pipeline explanation, exact procedure, comparison, constraints |
| Comparison | `results/comparison.md`: tris 21,030 / 11,520 / 6,678 / 5,000 / 1,000; Hausdorff, SSIM albedo/PBR, outliers, GLB and GPU MB identical to the recorded Phase 2 table at its precision |
| Local areas | `lab/` gained `compare_repro.py`. `study/` and `wedo/` were read, not changed: no substantive new finding |

**Omni's independent check.** Every file of the run copy's output was
written during the run (164 files after 14:10 JST, none before). The run log
holds the pipeline's build and eval lines and the shell's timing. Against
the original Phase 2 chain (the original copy's `out/step12/eagle_beak/`):
- `L0.glb` is byte-identical: deterministic MDT, original textures, KTX2;
- `L1`–`L4` differ in bytes.

The reproduction was actually executed, not copied. The trial report
missed the step12 chain: it compared only the tables and looked at an older
step7 output. Omni pointed this out. run-002 then compared bytes and
reported, in `report-bytes.md` and on the review page:
- geometry and accessors are identical in all levels;
- L1–L4 differ only in the embedded KTX2 payloads (bake/encode), with
  decoded pixels differing in less than 0.5 %.

## Delivery: two gaps, corrected by Front after Omni's follow-ups

1. **The review page stopped before anyone looked** (run-001 p1). lowerpoly
   served it "until you answer". Front answered its question "relayed",
   which let the run stop it. Reply #72 admitted it. Omni's follow-up #73
   (as Omni for the Developer) asked for a page that stays up until the
   Developer closes it, and for a recorded lesson.
2. **A promised follow-up with nothing to carry it.**
   - `front/converse/run-033` started `lowerpoly/freework/run-002` with
     `wfe request` instead of a delegate node.
   - It completed its own `talk` and replied "I will send the link once it
     is up".
   - No run remained to do that, and run-002's question (with the link) was
     not in the conversation. 40 minutes passed without the link.
   - Omni's follow-up #75 named the cause.
   - `front/converse/run-034` checked the page (curl 200), replied with the
     link (#76), linked run-002 to the conversation and left q1 open for the
     Developer.

Front recorded both lessons in `knowledge/review-pages.md` with their run
references. That file is on pj-front `main` (`98cf2bf`, read through
`wfe projects read`).

These are Omni interventions: two follow-up messages. Omni relayed no link
and changed no record. Front and lowerpoly did the corrections.

## Review entry points (live)

- Review page (served by `lowerpoly/freework/run-002` presentation p1,
  listening on every interface; HTTP 200 by name and LAN address; screenshot
  in the ignored `step4/`): `http://agstudio.local:8300/`. It switches
  between original/L0–L4 and has dark lighting and side-by-side compare.
  Measured values are beside the view.
- Downloads: `http://agstudio.local:8093/wfe/api/shares/agshare/runs/lowerpoly/freework/run-001/o1/`
- Trial run: `http://agstudio.local:8093/wfe/#/ws/lowerpoly/run/freework/run-001`
- Report: `http://agstudio.local:8093/wfe/api/projects/lowerpoly/runs/freework/run-001/raw?path=devdocs/runs/freework/run-001/report.md`
- Review run (q1 open for the Developer's visual verdict): `http://agstudio.local:8093/wfe/#/ws/lowerpoly/run/freework/run-002`
- Front Desk: `http://agstudio.local:8093/?view=frontdesk&conv=p9-lowerpoly`

## Claims kept apart

- Technical: the reproduction matches the recorded measurements, from
  lowerpoly's run and Omni's check.
- Agent assessment: none was offered on visual quality.
- **Pending human judgment**: whether L0–L4 look acceptable (holes, dark
  lighting). This is run-002's q1, open for the Developer.

## Limits

- The toolchain exists only in the original copy on the coordinator host.
  The run links to it, so moving that copy breaks reproduction, and another
  node cannot run the case without rebuilding it.
- One asset was reproduced.
- The conversation's "Waiting for the person" line counts only questions
  posted into the conversation. run-002's q1 shows as the linked run's
  status (`Waiting · Answer from person`), not in that line.
