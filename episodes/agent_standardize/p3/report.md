# agent_standardize p3 — plan/run topic vocabulary

**Done.** All three success criteria met live on 2026-08-21. Step reports:
[report1](report1.md) (agforge), [report2](report2.md) (agautolab + agfront),
[report3](report3.md) (cutover), [report4](report4.md) (proof).

## The rename

| old | new | owner | meaning |
|---|---|---|---|
| `create-` | `assetplan-` | agforge | plan an asset Work, do not execute |
| `runcreate-` | `assetrun-` | agforge | execute one asset Work |
| `mission-` | `workplan-` | agautolab | plan a mission, do not execute |
| `run-` | `workrun-` | agautolab | execute mission tasks |

`bmining-`, `front-`, `intro-`, `pj-`, `work-` and the `✔ ` resolved marker
are untouched. `create-asset_<issue>` became `assetplan-asset_<issue>`,
keeping the substructure. Derived names follow:
`workrun-task<N>-<label>`.

The braindump's complaint was that a name should say whether it plans or
executes. It now does, in both directions: `*plan-` never executes, `*run-`
only executes, and the `asset`/`work` stem says whose entrance it is.

**Nothing kept the old words.** No compatibility shim, no dual-sweep: an
old-prefix topic matches no sweep at all. The collision comment that used to
explain why `runcreate-` had to be matched before `create-` is gone with the
stem it described.

## What was wiped

One line, as the plan asked: **31 Zulip topics deleted across 10 channels and
11 pending Plane Works cancelled** — everything whose name or order key
embedded a retired prefix. All of it was test-only.

## Commits

`agforge` `2469d69` · `2b69bea` · `5752907` — `agautolab` `5675943` ·
`0892a68` · `9834691` — `agfront` `cc36fd0` — `pyagag` `1442783` —
`pj-agdev` `aced029` · `78c2382` · `e294697` — `devdocs` `c74026b` ·
`6e13ded` · `a87ec34` · `a3a88be`. All pushed to GitHub;
`agautolab1` deployed and verified at `5675943`.

## Evidence

1. **Criterion 1** — `assetplan-p3-smoke-icon` planned Work `F2-15`;
   `assetrun-p3-smoke` executed it, delivered the zip back into the origin
   `assetplan-` topic and marked the Work Done. [report4](report4.md).
2. **Criterion 2** — asked in `front-p3-vocab-check`, Front proposed
   "I'd need to open an `assetplan-…` topic", and on permission opened
   `agforge-agstudio1/assetplan-treasure-chest-icon`. **Zero agfront code or
   guide change**; the prefix reached the run only through the re-posted
   intro and the pre-run harvest, quoted in [report4](report4.md).
3. **Criterion 3** — the grep is clean over all four repos; three documented
   false-positive classes remain (`run-NNNN.json` record filenames, a fake
   client's `("create-channel", …)` call label, and `mission-level` /
   `mission-planning` as English compounds on the surviving domain noun).

Startup log lines are the cutover's own witness:

```
agforge zulip listener starting (pull sweep: all topics in
  'agforge-agstudio1', prefixes ('assetrun-', 'assetplan-') elsewhere)
agautolab zulip listener starting (pull sweep, prefixes
  ('workplan-', 'workrun-', 'assetplan-', 'bmining-'))
```

## What the episode taught

- **The intro is the contract, and it holds under a breaking change.** A
  prefix that agfront had never been told about arrived through the harvested
  `#agents` post and was used correctly on the first live run. That is
  criterion 2's whole point, and it is now demonstrated rather than asserted.
- **Guides carry no routing vocabulary.** All four prefixes moved without one
  byte of guide *content* changing — only folder names, which is code. So
  routing is a code change, never a prompt change. Saved to memory as
  `guides-carry-no-topic-vocabulary`.
- **A rename shaped around identifiers misses prose.** The first live run's
  first log line still said `create topic`. Reading the diff had not found
  it; running it did. Fixed in `agforge 2b69bea` / `agautolab 0892a68` —
  failure farming, working as intended.

## Living docs updated

`agforge/README_DEV.md`, `agautolab/README.md`,
`pj-agdev/.local/devenv.md`, and one new memory. Details in
[report4](report4.md).

## Deferred

- Collapsing `assetplan-asset_<issue>` further — out of scope by the plan.
- The `--limit agstudio` playbook, which fails in `claude_code_agent` on a
  user-scoped npm path this Mac does not have. Pre-existing, unrelated to the
  rename, harmless here (that placement's checkout is the working tree), and
  worth its own ENT episode.
- **A live `workplan-` → `workrun-` proof.** The forge half and the front
  half were both proven end to end; autolab's half is renamed, unit-tested
  and deployed but has not planned or executed a live mission in the new
  vocabulary. The obvious next thing to do.
