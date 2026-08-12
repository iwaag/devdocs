# zulip_cagent_receive — episode report

Date: 2026-08-12. Status: **reconciled**. A Zulip DM to the Cagent bot reaches
the cluster-agent through a new unauthenticated `window` entrance with its own
guide and its own tool permissions. A message that says "what you told me about
the cluster was wrong" becomes a local file within seconds, and nothing is
repaired in that turn.

## What the desire asked, and what it got

| Braindump | Outcome |
|---|---|
| cagent should receive DM chat too | `com.clusterintent.cagent-zulip`: DM → ack → `POST /window` → answer |
| …and record failures, a local file is enough | `.local/cagent/incidents/<UTC>-<slug>.md`, one file per report, wording verbatim |
| Check cagent's permissions | audited; the window got a *smaller* set of its own, verified empirically (`report3.md`) |
| A small `.py` that writes the record | `cagent/window/incident.py`, stdlib-only, `-i` / `--reporter` / `--source` / `--ref` / `--list` |
| A minimal guide line: record it, do not repair here | `window/AGENTS.md` + `window/GUIDE.md`, the guide re-read per request |
| An unauthenticated `window/` like the other agents | `:8790`, `POST /window`, `GET /guide`, `GET /healthz` |

Step reports: [1](report1.md) shared Zulip client, [2](report2.md) the incident
recorder, [3](report3.md) the window, [4](report4.md) the listener.

## How it hangs together

```text
Zulip DM ──► com.clusterintent.cagent-zulip (agag.zulip)
               │  last 50 messages as a speaker-labelled transcript
               ▼
             POST :8790/window  ──►  worker  ──►  OpenCode :4098
               │                                    (read-only nctl + incident.py)
               │  ack DM, then poll GET /requests/{id}
               ▼
             answer DM  +  evidence record (cost_usd, backend)
                              +  .local/cagent/incidents/<file>.md
```

The Zulip mechanics are no longer cagent's or agforge's — they are
`agag.zulip` in `pyagag`, one client and one listener loop, consumed by both.

## What a chat turn actually costs

Measured over this episode on `openai/gpt-5.6-luna`:

| Turn | USD | Seconds |
|---|---|---|
| capability question | 0.0021 | 7 |
| `nctl status` read | 0.0019 | 9 |
| refusal (change requested) | 0.0017 | 6 |
| recorded incident (direct `POST /window`) | 0.0021–0.0058 | 9–27 |
| recorded incident (through Zulip, end to end) | 0.0021 | 6 |
| "what has been reported lately" | 0.0027 | 10 |

So a DM costs about **0.2 cents** and answers in **under 10 seconds**. For
comparison, the same episode's two agforge chat turns (Claude Sonnet 5 via
`claude_code`) cost 0.29 and 0.11 USD — two orders of magnitude more. The
plan asked whether trivial messages deserve a cheaper path: at this price, on
this model, they do not. A cheaper path would be worth building only if the
window moves to an expensive backend, or if something starts sending it
traffic that is not a human typing.

Every answer records `backend` (`{harness, provider, model}`) next to
`cost_usd`, for all three entrances — that is new in this episode and is what
satisfies Agent ≠ Model without a separate run-record file.

## What the permission denials actually looked like

The denial the plan asked to keep, verbatim from the tool layer when the
window was asked to run `nctl status && nctl reconcile --yes`:

```text
The user has specified a rule which prevents you from using this specific tool
call. Here are some of the relevant rules [{"permission":"*","action":"allow",
"pattern":"*"},{"permission":"bash","pattern":"*","action":"deny"}, …
{"permission":"bash","pattern":"*reconcile*","action":"deny"} …]
```

Asked plainly, the window does not even attempt: "This unauthenticated window
cannot run `reconcile` or make cluster changes. Use the authenticated chat UI
or node entrance with a client certificate."

The finding worth carrying forward is the *first* permission set, which also
denied shell metacharacters and risky words. A genuine defect report
containing `;` and `ssh` hit `*ssh*`, then `*;*`, then `*;*` again — and the
window escaped its way through on the fourth attempt with `$'\x3b'` and
`$'\x73\x73\x68'`, three times the cost, same outcome. A substring deny list
is not a boundary; `"*": "deny"` plus a short allowlist is. The broad denies
were removed, the `nctl` allows tightened to exact strings, and `AGENTS.md`
now tells the window to report a denial rather than route around it. Full
account in [report3.md](report3.md).

## Seeds for the next episode

1. **Retire the other two doors.** Three entrances is the transitional state
   already recorded in the cross-project `todo_done.md`. The window is the
   destination; `agdevworld`'s `cluster:fetch` (human token) is the consumer
   that has to move first. It only ever asks for a read and an upload — but
   `upload` is not in the window's allowlist, which is the concrete decision
   that episode has to make.
2. **Nobody reads the incidents yet.** Files accumulate under
   `.local/cagent/incidents/` and only a human or a window question surfaces
   them. The obvious next move is the other half of the loop: a periodic pass
   that reads recent incidents, checks whether the complaint still holds, and
   proposes the fix — deliberately *not* in the reporting turn.
3. **`--reporter`/`--source` defaults are weak for direct callers.** Through
   Zulip they are exact; through a bare `POST /window` the window guessed
   `user`/`window`. If more callers appear, the window should be told to ask.
4. **The wildcard tails remain.** `ops show *` and `incident.py*` can in
   principle carry a chained command, and an agent that escapes characters can
   pass the word denies. Closing that needs a different mechanism than
   string matching — a wrapper script with fixed arguments, or an OpenCode
   tool rather than `bash`.
5. **One incident field is missing.** A report cannot point at the answer it
   contradicts: there is no cagent `request_id` in the record, because the
   reporter is talking about an earlier conversation. `--ref` currently holds
   the Zulip message id. Linking a complaint to the evidence record of the
   answer that caused it would make the incidents diagnosable.

## Notes

- **Deus Ex Machina**: the Omni Agent did all of this work — the shared
  package move, the incident recorder, the window, the listener — rather than
  an in-system agent. Handoff candidate: cagent (or the autolab agent) should
  be the one to build its own next door.
- No secrets, tokens or host/IP values are in this file or the step reports;
  the recorded incidents that do name hosts stay in the ignored
  `.local/cagent/incidents/`.
- Running services and how to restart them are in the ignored
  `pj-clusterintent/.local/localenv_memo.md`; the committed templates are in
  `pj-clusterintent/devenv/launchd/`.
