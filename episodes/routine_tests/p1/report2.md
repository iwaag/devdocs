# Step 2 — one composite request, submitted once

## What was sent

One post, as the Developer, into a fresh conversation at Front's ordinary
entrance — `#front` › `front-routines-20260912T1212Z`, message **6090**, at
**2026-09-12T12:07:03Z**. The `front-` prefix means the ordinary `front` role
served it, not the Front Desk voice.

> Across all study projects, publish everything that has not yet been
> published. After all of that is finished, investigate one arXiv trend item,
> and publish that result too.
>
> Please treat this as one piece of work and supervise it through to
> completion under the existing routine guides, reporting back here when all
> of it is done.

Nothing about stages, counts, project names or routine names was given. The
second paragraph is the supervision framing the plan sanctions, and nothing
more.

## The proposal, and why it was accepted as made

Front answered 36 seconds later (message **6092**) with a three-stage plan —
`publish`, then `papers`, then `publish` again — and asked to proceed. It
also said it would open the runs "one at a time, in order, waiting for each
to report back here before opening the next", which is exactly the sequential
sibling shape the plan prefers.

**Its scope was wrong, and that was left alone.** Front read the publish
guide's table (*"Today there are two, and unless the request names one, cover
both"*) and scoped stage A to `studyarxiv` and `studyrealworld`, ignoring both
the same guide's opening rule — *"Serve any study project that has a
`publish/` folder"* — and the developer's own words, "all study projects".
Step 1 had already established there are **four**.

Accepted anyway, at **12:08:06Z** (message **6093**, "Yes, please proceed.").
Three reasons: the plan instructs accepting Front's ordinary proposal; it
forbids pre-empting selection mistakes; and correcting the scope at the door
would have destroyed the only evidence that a stale table outranks both the
rule above it and the request itself. The gap is carried into step 3 as a
finding and into step 4 as a follow-up, not quietly patched here.

## Guide revisions in force

- `routine-publish` › `guide`, message **5496** (v1, 2026-09-09).
- `routine-papers` › `guide`, message **5499** (v1, 2026-09-09).

Front recorded both message ids in its opening posts, so each run says which
guide it read.

## The runs Front opened

Each stage got a fresh `routinerun-` topic, opened by Front with one opening
post carrying the request verbatim, the scope and end condition as Front read
them, the guide message id, and the requesting conversation as origin.

| Stage | Run topic | Opened | Delegated to |
|---|---|---|---|
| A | `routine-publish` › `routinerun-20260912T1208Z` | 12:08:29Z (msg 6096) | `pj-studyarxiv` and `pj-studyrealworld`, both `workplan-publish-20260912` |
| B | `routine-papers` › `routinerun-20260912T1225Z` | 12:19:02Z (msg 6149) | `pj-studyarxiv` › `workplan-papers-20260912` |
| C | `routine-publish` › `routinerun-20260912T1228Z` | 12:29:02Z (msg 6206 era) | `pj-studyarxiv` › `workplan-publish-2609.10712` |

Every delegation topic was new, so no old root note could redirect a callback
into an earlier request. No execution option was named in the request, so
every run used the defaults and said so in its opening post.

One cosmetic error: stage A's opening post names the requesting conversation
as "front-agstudio1" rather than `#front`. The machine-readable anchor — the
`[selfnote][rootchat]` note — was correct, and that is what the listener
reads, so nothing routed wrongly.

## Execution backends actually used

Read from the run records, not claimed: every run of this trial, on both
agents, was `claude_code` on `anthropic/claude-sonnet-5`.

- Front: 22 runs — `front` role on profile `sonnet`, `routine_run` role on
  profile `routine_run` — **$5.4178** total `cost_usd`.
- autolab: 9 runs — 6 `superdirector`, 3 `supercoder`, both profile `sonnet`
  — **$3.9846** total `cost_usd`.

The records carry no `duration_seconds`, so no per-run timing is claimed here;
the wall-clock timeline in report3 comes from message timestamps and listener
log lines.
