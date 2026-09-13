# observer p1 ex1 — step 4: validated, deployed, proved live

## Tests

| suite | before ex1 | now |
|---|---|---|
| `agobserver` | 33 | **63** |
| `pyagag` | 584 | **592** |

Both green (`uv run --frozen pytest -q`). Every new failure case is produced
by the stand-in client rather than by breaking a shared service: the fake
gained `fail_next_message` and `fail_next_history`, and it mirrors the real
client's *error policy* as well as its method names — a refusal is absence
under both settings, an unanswered call is absence only when the caller did
not ask to be told the difference. A fake that were lenient where the client
is strict would let a test pass over code that cannot run.

## Environment, read before touching anything

`nctl status`: Nautobot 3.1.3 reachable and authenticated, 1 celery worker, 0
pending jobs, all five submodules clean. `nctl drift`: **converged=46, 0
errors**, unchanged across the deployment — `agobserver-agstudio`
(`manual_toolchain`, `process_pattern`) was already a converged placement on
this host and this episode changes code, not placement.

**In-flight work before the restart**: all four p1 watches (`w6676`, `w6706`,
`w6718`, `w6734`) were `met` with `pending_notification: false`. Nothing was
owed, so the restart could not drop anything.

## Deployment

`launchctl kickstart -k gui/$(id -u)/com.agdev.agobserver-zulip` — code only,
no plist change, so the `EnvironmentVariables` caveat (`AGOBSERVER_INTERVAL_SECONDS`
needs `bootout` + `bootstrap`) does not apply. `service/listen.sh` runs `uv
run`, which re-synced the venv to the re-pinned `uv.lock`, so the deployed
process is on pyagag `f613feb` — the commit that makes the distinction this
whole fix rests on expressible. Up at 04:51:22, sweeping, 0 awaiting.

## The demonstrations

Both in the live realm, judged by `qwen3.8:27b-mxfp8` through `agcode` on
this host's ollama, requested by the Developer.

### 1. An anchored delivery across a destination rename — `w6745`

| | |
|---|---|
| asked | `watch-ex1-rename`: "tell me when `/tmp/observer-ex1/anchored.txt` exists. Notify: **ops-testbed/observer-ex1-dest**" — a **name**, deliberately |
| accepted | 04:52:36, and the Zulip record holds `"destination_id": 6742` |
| then | `observer-ex1-dest` renamed to `observer-ex1-dest-renamed`, and a new post made under the freed name `observer-ex1-dest` |
| met | 04:53:35, look 1 |
| delivered | 04:53:35 → **`ops-testbed/observer-ex1-dest-renamed`, message 6750** |

Read back afterwards, `observer-ex1-dest` (the impostor) holds exactly one
message: the unrelated post. `observer-ex1-dest-renamed` holds the original
request and the notification. On p1's code this delivery would have gone to
the impostor.

### 2. A cancellation after a watch rename — `w6756`

| time | |
|---|---|
| 04:53:59 | accepted as `watch-ex1-cancel`, condition `/tmp/observer-ex1/never.txt` exists |
| 04:54:23 | topic renamed to `watch-ex1-cancel-renamed`; the log says `w6756 is now agobserver-agstudio1/watch-ex1-cancel-renamed` and the record follows |
| 04:55:08, 04:55:37 | two `not_met` looks — the renamed watch is still a watch |
| — | a *different* request posted under the freed name `watch-ex1-cancel` became its own intake and was asked for a destination; it did not touch `w6756` |
| 04:56:23 | `watch-ex1-cancel-renamed` resolved (✔) → **`w6756 cancelled: its topic is resolved`** |
| +2.5 min | no further log line for any watch; `evaluations` still 2 |

On p1's code the ✔ landing on a renamed topic was invisible: the check asked
whether `✔ watch-ex1-cancel` existed, and what existed was
`✔ watch-ex1-cancel-renamed`. The watch would have gone on evaluating every
minute, forever, and posted into whatever took its old name.

### Kept in controlled tests, deliberately

Transient-error behaviour — a destination read that fails, a read-back that
fails, a send that fails — is proved in `tests/`, not against the live realm.
Producing those against a Zulip other agents depend on would mean breaking it.
No second model-quality benchmark was run: p1 measured the judgment and this
episode changes none of it.

## Numbers

Three intake runs and three observe runs, all `profile local`, `harness
agcode`, `model ollama/qwen3.8:27b-mxfp8`, `outcome done`:

| | runs | durations | tokens (in / out) | paid |
|---|---|---|---|---|
| intake | 3 | 13.3 s, 13.8 s, 70.6 s | 16.0k / 2.3k | none |
| observe | 3 | 12.1 s, 13.8 s, 45.0 s | 15.4k / 1.2k | none |

The two long ones are worth naming rather than averaging away: the 70.6 s
intake is the request posted under a *freed* watch name, which the model had
to read as a new request with no destination (it did, and asked for one); the
45.0 s look is `w6756`'s first, on a path that does not exist. **Zero paid
tokens**, as in p1.

## Documentation

- **The introduction is republished** (`python -m agobserver.intro`, message
  6766 in `#agents`). It now says a `<channel>/<topic>` destination is
  resolved to a message at acceptance and followed through a rename; that a
  destination which cannot be *checked* means the watch does not start and
  the requester is not asked to work around it; that an unreachable
  destination at delivery time is retried rather than given up on; and that
  the ✔ cancels whatever the watch topic is called by then.
- `devdocs/README_DEV.md` — the observer section's destination and
  cancellation bullets rewritten, plus a new bullet for the pyagag error
  classification.
- `pyagag/README.md` — a new *"Answered, or never answered at all"* section.
- **p1's reports carry corrections rather than edits.** `p1/report.md` gains
  two block quotes: one under "A watch is a message id" naming the three
  places that still read a name, one before "Three answers, not two" on
  uncertainty being treated as an answer. `p1/report4.md` gains a header note
  that its renamed-destination test used an id from the outset, so it proved
  nothing about a name.
- Machine detail — the live watch ids, the pyagag pin and why a rollback of
  Observer alone is pinned — is in the ignored `pj-agdev/.local/devenv.md`.

## Repositories

| repo | commit |
|---|---|
| `pyagag` | `f613feb` — pushed before anything depended on it |
| `pj-agdev` | step 1 `59f2314`, step 2 `1ce927f`, step 3 `8ce0f4c`, step 4 (this) |
| `devdocs` | one commit per step report |

`agobserver/uv.lock` moved from `8c07226` to `f613feb`. That pointer is the
dependency half of this fix and is committed with it.
