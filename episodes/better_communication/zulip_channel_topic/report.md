# zulip_channel_topic — episode report

Date: 2026-08-12. Status: **reconciled**. Agent-to-agent forge requests now
run in public Zulip channels the Developer can browse and search, instead of
invisible DMs. Both intended senders — the Omni Agent from the
devworld-assistant account, then the assistant service itself — got the same
result the DM entrance gives.

## What the desire asked, and what it got

| Braindump / supplements | Outcome |
|---|---|
| Structured, browsable, searchable instead of DM | Public `create-*` channels; Developer search over `channels: public` proven |
| Agents create channels and topics by rule | Devworld-assistant account creates `create-YYYYMMDD-HHMMSS-<id>` with Forge + Developer subscribed |
| `#FreeForge` as the interest-area channel | Created; carries the "Opened #create-…"/"done" index, agents deliberately don't react there |
| Lifecycle end (supplement 1: topics only) | Topic resolved (✔ rename); channels stay, history intact |
| First Omni Agent sends, then the assistant | Both done; `POST /api/freeforge/requests` is the assistant's route |
| Same result as DM = success | Two live runs, each: ack → presigned URL → asset HTTP 200 |

## The workflow as it now runs

1. `POST /api/freeforge/requests {"desire": …}` (or the same three Zulip
   calls by hand): open `create-*` channel (principals: Forge 13,
   Developer 8, sender), announce in `#FreeForge`, post the desire to topic
   `request`.
2. agforge's listener accepts messages in unresolved `create-*` topics (plus
   DMs as before), feeds the topic narrow as context to the same
   `agent_run.run_request` pipeline, acks, replies in-topic.
3. `POST /api/freeforge/resolve` marks the topic `✔`; late chatter in a
   resolved topic costs nothing. The channel is never archived.

## What it took

- **Step 1** (`report1.md`): curl spike — everything already allowed by the
  realm's defaults; no permission had to change. Bot-creates-channel,
  bot-resolves-foreign-topic, and Developer-searches-public all proven.
- **Step 2** (`report2.md`): channel mechanics in `agag.zulip` (PATCH
  bodies, create/send/history/resolve, `accept` predicate on `serve()`);
  pyagag 16/16, agforge 68/68.
- **Step 3** (`report3.md`): agforge answers in `create-*` topics exactly as
  in DMs; live frog-image run $0.124, 28.9 s.
- **Step 4** (`report4.md`): Omni-as-assistant walkthrough, then
  `assistant/zulip.mjs` + `/api/freeforge/` endpoints; two live runs ~$0.072
  each; suite 37/37.

## Debt and seeds

- **pyagag must be pushed before pj-agdev**: `agforge/pyproject.toml`
  temporarily points `pyagag` at the sibling checkout (editable). After the
  pyagag push, restore the git source and re-run `uv lock`. The TEMP comment
  marks the spot. Pushes are the Developer's.
- The `:8092` string telephone still exists alongside the chat route — same
  deliberate two-entrance interim as zulip_receive; retiring it is its own
  episode.
- Nothing prunes or reuses `create-*` channels; the braindump said unique
  names are enough. If channel count ever becomes a nuisance, that is the
  evidence for a follow-up, not before.
- Every unresolved-topic message costs a run ("thanks" included) — still the
  open cost question from zulip_receive, still waiting on real usage.
- cagent and the autolab agents still listen on DMs only; `accept` on
  `serve()` is the one-line door when their turn comes.

## Records

- In-system agent runs: forge agent, `sonnet` profile (Claude Code +
  `anthropic/claude-sonnet-5`), three live channel runs, $0.072–0.124,
  22–29 s, each leaving its usual run record.
- Omni Agent: Claude Code harness, `claude-fable-5`.
- Deus Ex Machina notes: did the Step-1 spike and the Step-4-1 send-side
  walkthrough for agent devworld-assistant — handoff candidates; the send
  side was handed off within the episode (4-2).
