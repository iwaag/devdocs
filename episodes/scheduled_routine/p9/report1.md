# Step 1 — the project and its contract

`mediagen` exists as a four-surface project (Zulip channel, Plane project,
two Gitea repositories, workspace clones), created by one autolab serving on
a Developer request. The pattern document now carries a generation-test
contract that the bootstrap run read and applied without being told to.

## The pattern-doc change

`agautolab/agent/project_pattern.md`, commit `197df59` — 40 lines inserted,
one new section `## Repository-backed generation tests` between the local-test
section and `## "study" pattern`. Nothing else in the file was touched.

What it says, in six paragraphs:

1. **Same folder/repository path as a local test.** A media-generation study
   may add one bounded, resumable folder per subject on the ordinary
   `init-repo` naming path — `autolab project init-localtest <subject>`
   reuses it directly, `autolab project init-repo gentest-<subject>` plus a
   hand-written yaml is the alternative when the `paper-id`/`localtest`
   vocabulary does not fit. Either is fine; record the choice in
   `README_PROJECT.md`.
2. **What the resumable yaml records**: subject (checkpoint, LoRA set, or
   workflow family), the backend in generic terms only, the model or
   workflow under test, and the state. `report.md` stays the raw run log —
   matrix spec, per-image parameters and timings, what was judged against
   what.
3. **The distilled result is `main/<subject>/tips.md`.** A tip is "under
   these conditions this comes out", and a condition that did nothing is a
   tip too. Every tip carries its evidence on one line (matrix cell or seed
   plus the settings that mattered) and the date. **Append-only**: a later
   run that contradicts an earlier tip adds a newer one rather than
   rewriting, so the file reads as a history and a second run always knows
   where to append. `summary.md` keeps the hearsay (what the model is, what
   its authors and the public recommend); `tips.md` keeps what we found.
4. **No level scale.** Stated explicitly against the `L1`–`L4` table
   directly above it: that axis is a claim about how far a paper was
   reproduced and means nothing for a generation sweep. The `tips` column of
   `INDEX.md` therefore says only `no` or the date of the most recent tip.
5. **Stands alone when copied out of `main/`**, same rule as `test.md`: no
   link into the test folder, no repository name, no host facts.
6. **Where the bytes go**: raw outputs in the test repository's ignored
   `.local/`; a few small contact sheets may be committed to the test
   repository, one or two in `main/` when they carry the point.

Committed and pushed: agautolab `197df59` → GitHub; pj-agdev submodule pin
`745d593` → GitHub. No restart — the pattern doc is read from disk per run,
and the listener that served the bootstrap forty minutes later picked up the
new section without one.

## The bootstrap, as the Developer

**Channel.** `#pj-mediagen`, stream **93**, created with
`ZulipClient.create_channel` from `.local/zulip/developer.env`, description
`autolab project: mediagen (study pattern)`, principals 8 (Developer) and 11
(autolab bot), **folder 1** — the folder named `pj-runsmoke2` but described
"Project channels and their work channels", which is where `pj-studyarxiv`,
`pj-ghtrends`, `pj-papers`, `pj-rtnotes`, `pj-studynourl` and `pj-runsmoke2`
all live. (`pj-foodchain`, `pj-runsmoke1` are unfiled; `pj-simpleshooter` has
its own folder 2. The folder naming is pre-existing untidiness, not
something this step changed.)

**Marker.** `agautolab/.local/projects/mediagen/README_PROJECT.md`, the same
80-byte line `ghtrends` was declared with:

```
pattern-managed; folders are created by the workplan on the developer's request
```

**State before the request** — `Mediagen` absent from Plane (25 projects
listed), no `mediagen*` in Gitea `autodev` (repo search returned nothing).

**The request** — `#pj-mediagen › workplan-create`, message **3057**, posted
as the Developer at 18:50:28Z:

> studyパターンのプロジェクトを作って。リポジトリは標準の名前でいい。mainには生成モデル（checkpointやLoRA）とComfyUI/SwarmUIのワークフローについてのまとめ、および生成テストの結果を蓄える。publishにはレビュー済みのコピーだけを置く。

**The reply** — message **3059**, 43 s later:

> @**Developer**
>
> studyパターンでプロジェクトを準備しました。
>
> - `main/`(`autodev/mediagen`) — checkpoint/LoRA と ComfyUI/SwarmUI ワークフローのまとめ、生成テスト結果(`tips.md`)を蓄積
> - `publish/`(`autodev/mediagen-publish`) — レビュー済みコピーのみ。`publish`ルーチンでのみ更新
>
> `README_PROJECT.md` に記録済みです。チェックポイントやLoRAなど具体的な生成テスト対象(subject)が決まったら、そのつど `gentest-<subject>/` リポジトリを追加します。
>
> これはプロジェクト準備であり、まだミッション(plan.md)ではありません。具体的な作業内容(どのモデル/ワークフローをまとめるか等)が決まったら、ミッションとして計画します。

Listener log, the two lines a pattern-managed serving produces:

```
2026-08-29T18:50:28Z serving 'pj-mediagen'/'workplan-create'
2026-08-29T18:50:28Z workplan topic 'pj-mediagen'/'workplan-create'
```

## Verification

| surface | before | after |
|---|---|---|
| Plane project | 25 projects, no `Mediagen` | 26, `Mediagen` (`M`, `e5ae3428…`), not archived |
| Gitea `autodev/mediagen` | absent | created `18:50:59Z`, `empty: true` |
| Gitea `autodev/mediagen-publish` | absent | created `18:51:00Z`, `empty: true` |
| workspace | marker only | `main/`, `publish/` cloned, both on the unborn branch (`origin/main [gone]`) |
| `README_PROJECT.md` | 80-byte marker | rewritten (below) |

Both clones point at `http://agstudio.local:3000/autodev/mediagen{,-publish}.git`
and hold no commits, so Step 2's first push needs `git push -u origin main`.

`README_PROJECT.md` as landed:

```
# mediagen — study pattern

- `main/` — repository `autodev/mediagen`. Working notes on generation
  models (checkpoints, LoRA sets) and ComfyUI/SwarmUI workflows, plus
  generation test results (`tips.md` per subject, per the "Repository-backed
  generation tests" convention in `autolab doc patterns`).
- `publish/` — repository `autodev/mediagen-publish`. Reviewed copies of
  `main/` reports only, produced by the `publish` routine. Never edited
  directly; never pushed by an agent.

Generation-test subjects (checkpoints, LoRA sets, workflow families) each
get their own `gentest-<subject>/` repository, created with
`autolab project init-localtest <subject>` (or `init-repo
gentest-<subject>` plus a hand-written yaml) when a specific subject is
started. None exist yet.
```

## What the bootstrap did differently from p5

1. **The declaration line is gone — the marker was replaced, not appended
   to.** `project_pattern`'s long-standing flaw (2 of 3 times in p5's count:
   a sentence written for `init_project` left standing as documentation) did
   not reproduce. Nothing in the request or the pattern doc asked for that,
   so this is one data point that the flaw is not deterministic, not that it
   is fixed.
2. **It read the new section unprompted.** The request says nothing about
   generation tests beyond "生成テストの結果を蓄える". The run went and found
   the `tips.md` convention, wrote it into `README_PROJECT.md`, named the
   section it came from, and told the Developer in its reply that a
   `gentest-<subject>/` repository will be added per subject. The contract
   travelled without a mission text having to restate it — which is exactly
   what p8's friction #4 (missions spending tool calls hunting for the
   pattern doc) wanted.
3. **It named the doc as `autolab doc patterns`**, a CLI route rather than a
   path — better than p8's relative path, and the route is real:
   `autolab doc patterns` prints this file (`autolab doc --help` lists it as
   the one known document). p8's friction #4 is answered by that command.
4. **One inconsistency it introduced**: `README_PROJECT.md` says the folder
   is `gentest-<subject>/` *and* that it is created with `init-localtest
   <subject>` — but `init-localtest` makes `localtest-<subject>/`. The two
   halves of the pattern doc's "either is fine" got spliced. Step 3 picks
   one for real and must correct that line.
5. **Slower and dearer than p5**: 43 s / 12 turns / **$0.187** against
   `ghtrends`' 21 s / 7 turns / $0.122 (`superdirector/run-0111` vs
   `run-0086`, both `claude_code` / `anthropic/claude-sonnet-5`, both
   `"outcome": "done"`). The extra five turns are the pattern-doc reading.

## Run record

`agautolab/.local/agent/superdirector/run-0111.json`: `superdirector` /
`claude_code` / `anthropic/claude-sonnet-5`, **12 turns**, **42 516 ms**,
**$0.187209**, `"outcome": "done"`. No mission, no `plan.md`, no Plane
Sub-Work, no `workrun-` topic — project setup only, as the reply says.

## Live-stack note recorded while checking (for Steps 2–3)

Both generation backends answer today, which the plan did not expect:

- SwarmUI is **0.9.8.2** (`GIT-2d50413a`), not the 0.9.7.4 the plan recorded.
- The standalone ComfyUI on `:8188` **answers** — `/system_stats` returns
  ComfyUI **0.33.1**, 132 GB RAM, PyTorch 2.13.0+cu130. The plan recorded it
  as connection-refused on 2026-08-30; it is up now, so Step 3 has three
  backend options rather than two.
- `agpc.local` was avoided throughout in favour of the IPv4 literal, per the
  known AAAA stall.

## Deus Ex Machina interventions

- None beyond the plan's own division of labour. The pattern doc, the
  channel, the marker file and the request are harness-side Developer work
  by design; everything inside `main/`, `publish/` and the workspace was
  written by the autolab run.
- No permission-classifier stops this step.
