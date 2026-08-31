# Step 1 — command intake in the daemon

## What was built

`comfynotify`'s daemon grew a second sweep, in the same process and on the
same 5 s cadence. Each tick it reads Zulip's `is:mentioned` narrow, turns
every new mention into a ticket, and acknowledges with a reaction:

```
@**Comfy Notifier** watch <prompt_id> [free text kept as the note]
```

Files: `comfynotify/src/comfynotify/commands.py` (new — parser, high-water
mark, `CommandIntake`), `cli.py` (`build_intake`, the daemon loop, the
`--command-state` / `--no-commands` switches), `notifier.py` (`Notifier.send`
extracted so the callback and the command's error line leave by one door),
`pyproject.toml` (pyagag from git), `tests/test_commands.py` (nine tests).

## The implementer's calls the plan left open

- **pyagag in-process, not a direct HTTP call.** `ZulipClient.from_env` reads
  the credentials file the daemon's launchd env already names, and
  `mentions()` is the intake the plan asked for; writing a second Zulip
  client beside it to save one dependency would have been the worse trade.
  `ZulipClient.add_reaction(message_id, emoji_name="eyes")` was added to
  pyagag and pushed (`1eb20a2`) — it is a chat mechanic, and that is where
  the chat mechanics live. No dependant needed bumping: nothing else calls it
  yet, and comfynotify picks up `main` as a new dependency.
- **Emoji: `eyes`.** "I have seen this and I am watching it" is exactly what
  the ack means.
- **State file: `comfynotify/.local/command-mark.json`**, one object,
  `{"last_message_id", "updated_at"}`, written through the same atomic
  `replace_ticket` the tickets use. It sits *beside* the ticket directory
  rather than inside it, because `load_tickets` globs `*.json` and would
  otherwise try to serve the mark as a ticket.
- **The mark advances before the work, not after.** A crash in that window
  loses one command; the other order posts one callback twice, and a second
  callback serves an agent a second time — the prohibition that actually
  costs money. Nothing in `_handle` is expensive enough to be worth the
  inversion.
- **First run on a host seeds the mark to the newest visible mention and
  processes nothing.** The narrow remembers months; adopting the present as
  the past is the only sane start. Visible in the log as
  `command mark seeded at 4420`.
- **Grammar:** `watch` and the braindump's `watch_comfy`, case-insensitive,
  the `prompt_id` unquoted (an agent that writes `` `id` `` means the same
  job), the rest of the line kept verbatim as the note. Zulip's silent
  `@_**Name**` mention form is stripped like the loud one.
- **Zulip credentials or `AGFORGE_COMFYUI_URL` missing → intake off, logged,
  daemon still serves tickets.** Command intake is an addition, not a
  precondition; the CLI path the project started from is untouched.
- **A failed reaction never costs the ticket** — it is logged and the job is
  still watched.

`AGFORGE_COMFYUI_URL` was added to
`devenv/launchd/com.agdev.comfy-notifier.plist.in` as `__COMFYUI_URL__`,
alongside the existing `__PROJECTS_ROOT__`; the host itself exists only in
the installed copy under `~/Library/LaunchAgents/`.

## Tests

`uv run pytest` — 19 passed (10 pre-existing, 9 new). The new ones assert:
a `watch` line becomes a ticket aimed at the posting topic with the note and
the daemon's ComfyUI URL, acked by a reaction and **no post**; `watch_comfy`
and a backticked id accepted; junk refused rather than guessed; a malformed
command posts exactly one line and writes no ticket and no reaction; the same
feed replayed does nothing the second time; a *new process* on the same feed
re-tickets nothing but still catches the next command; our own posts,
selfnotes and DMs are not commands; a command in a `✔ ` topic is still
honoured; a failing reaction never costs the ticket.

The code-fence case is a documentation assertion, not a test: Zulip does not
render a mention inside a fence and does not set the `mentioned` flag, so
such a post never enters the narrow and never reaches this code.

## Live proof

`launchctl bootout` + `bootstrap` with the new plist. Log:

```
command intake on as Comfy Notifier (21)
command mark seeded at 4420
```

A 512×512, 8-step SDXL job was submitted directly to ComfyUI
(`ca370ad6-…`, finished `success`), then the command was hand-posted from the
Omni Agent account into the public scratch topic `#general > zulip-command-step1`:

```
[17:02:25] Omni Agent (4464): @**Comfy Notifier** watch ca370ad6-… step1 hand-posted proof
[17:02:27] Comfy Notifier (4465): comfy success ca370ad6 in 0s — 1 outputs · step1 hand-posted proof
                                  prompt_id `ca370ad6-…` — read `GET /history/<prompt_id>` for outputs
```

Two seconds, one sweep, as the plan predicted for an already-finished job.
Message 4464 carries `{'emoji_name': 'eyes', 'user_id': 21}` — the ack is a
reaction by the notifier bot, and nothing but the terminal callback was
posted. `command-mark.json` advanced to 4464; the ticket was archived under
`.local/tickets/done/`.

Done: tests pass, the daemon runs the new plist, and one Zulip line produced
a reaction, a ticket and the two-line callback with no CLI anywhere.
