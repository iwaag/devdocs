# front_desk p2 — settings and character conversations

The braindump asked for three things: character settings and backgrounds
cloned from a configurable repository and read at runtime; Front's
`character_talk` turning the real exchanges with autolab and the others
into a short conversation between characters, replayed in the Front Desk
as a graphic novel with the other speaker in the upper left; and a history
with faces. All three exist and one delegated task went through them end
to end: `ayghri/i-have-adhd`, commit `fdf6d28` in the ghtrends repo,
reported back as a scene between 姐さん and 親方 citing the post it came from.

Steps: `report1.md` (settings repository, sync, `/settings`),
`report2.md` (lore and evidence for the run), `report3.md` (the dialogue
block), `report4.md` (scene playback and portrait history), `report5.md`
(verification, deployment, the live conversations).

## What exists now

- **`iwaag/agdevworld-settings`** has a `manifest.toml`: characters
  (name, nickname, lore, face, which agents speak as them) and rooms
  (background). Adding a character is a table and two files there.
- **`agentroom-settings sync`** (agdevworld, config in
  `.local/settings.toml` from the tracked example) fetches, snapshots the
  commit, checks the manifest and every file it names, and only then
  switches the active revision; a failure keeps the previous one and says
  why. Every revision synced stays under `revisions/<sha>/`; `current/` is
  the active one as files. The relay serves `/settings`,
  `/settings/<sha>` and `/settings/<sha>/<path>` with the revision in every
  asset URL. No restart, no rebuild for a content update — shown live.
- **`character_talk`** pins the active revision at the start of each run,
  gets `characters.md` (complete lore, Front marked) and evidence files
  that keep message ids, sender ids and topic names and say when a thread
  is `✔`, bounded or unreadable; the guide no longer describes the
  character. The run's reply may end with an `ag-dialogue` JSON block —
  validated against the pinned revision, re-serialized, revision stamped
  — posted with the reply in one Zulip message; an unusable block keeps
  the reply and records why. Zulip remains the only record.
- **The Front Desk** draws faces, names and the background from the
  settings revision; plays a reply as turns (Front lower-left, the other
  character upper-left, dimmed when not speaking) with paging, queued
  arrivals and a way forward; shows history as portrait, name and text
  per turn with a user icon for the developer and a common icon for an
  unknown speaker; draws a scene with the revision it was written for and
  reports a missing one.

## What the live run taught

- **A callback lands in the conversation that first anchored the topic.**
  The p2live screen asked for the mission; the answers went to the
  morning's `front-desk-20260908-164810`, which had opened
  `workplan-trend8`. One root note per topic, earliest wins (p8).
- **Plain `agentchat read` did not follow the ✔ rename** (only `--since`
  did), so Front twice called a real completion fabricated — while the
  thread file it had been given said, in its header, that the topic was
  resolved. Fixed in pyagag `41cc5d5`; the other agents' locks still hold
  the old pin.
- **The guide could not stop the English preface** on sonnet-5 (three of
  three casual runs after the stronger wording); a one-rule post-process
  in agfront now drops it and logs it.
- Front's own judgement was otherwise good: it refused to call a resolved
  but empty topic done, asked autolab in its own channel, cited posts by
  id, and kept results, ids and names exact inside the character voice.

## References

- Conversations: `#front` › `front-desk-20260908-p2live` (5281–5317),
  `front-desk-20260908-164810` (5300–5314); autolab:
  `autolab-agstudio1 › status-ghtrends-g17` (5287–5290), `pj-ghtrends ›
  workplan-trend8` (5292–5299), `work-g-17 › ✔ workrun-rerun-task1-g-17`
  (5298–5308).
- Result: ghtrends `main` commit `fdf6d28`, pushed.
- Runs: `agfront/.local/agent/character_talk/run-0019…0025.json`, $1.32.
- Code: agdevworld `b0427fb`, `62a8084`, `1f8b785`; agfront `9c002ba`,
  `ef26752`, `143908c`, `a446819`, `82c73ad`; pyagag `41cc5d5`;
  agdevworld-settings `4f3b55f`; pj-agdev pointers through step 5.
- Screenshots: `agdevworld/.local/shots/p2/` (ignored).

## Remaining issues

See `report5.md`: callback routing to the first anchor; the preface rule
as a small shackle; other agents' pyagag pin; the `agautolab1` VM; two
cosmetic items in the scene.
